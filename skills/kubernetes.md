---
name: kubernetes
description: Use when authoring or verifying a Helm chart or Kubernetes deployment — chart gates, per-pod identity in a StatefulSet, and a throwaway kind cluster with podman
---

# Kubernetes and Helm

## Verify a chart by running it

A chart that only ever gets grepped is unverified. Text assertions catch drift between templates and
`values.yaml`; they cannot catch a chart that renders valid YAML the cluster then refuses to run.
Climb the ladder:

```sh
helm lint deploy/helm/<chart>                                    # syntax, metadata
helm template <rel> deploy/helm/<chart> --set <mode>=<alt>        # renders — once per branch
kind create cluster --name <task> --wait 120s                     # from here on, a live API server
helm template <rel> deploy/helm/<chart> | kubectl apply --dry-run=server -f -   # schema, admission
helm install <rel> deploy/helm/<chart> -n <ns> --create-namespace [--set …]
kubectl -n <ns> rollout status statefulset/<rel> --timeout=300s
```

Every `{{- if }}` branch (TLS modes, optional listeners) needs its own render: the branch nobody
renders is the branch that is broken. The ladder ends at pods reporting Ready, which is not proof —
the chart is proven when a client has used what it deployed (see "Drive it from inside the
cluster").

### Verifying a rendered manifest against a cluster you do not own

A committed generated manifest (ArgoCD repos, `DO NOT EDIT MANUALLY` headers) verifies in a different
order from a chart you are authoring. First prove the file is what the chart produces — the only check
that catches a hand-edit or a stale regeneration:

```sh
helm template <rel> <chart>.tgz -n <ns> --values=… | diff - <(tail -n +4 <app>/<env>/manifest.yaml)
```

Then dry-run against a namespace that exists, **retargeting per namespace — never one blanket `sed`**.
A manifest whose app sits in one namespace and whose `KafkaUser` sits in the operator's collapses into
one, validating objects where they will never live and erasing the cross-namespace relationship most
likely to be wrong. (`^  namespace:` also misses nested ones: webhook `clientConfig`, RoleBinding
subjects.) Run both paths — they answer different questions:

```sh
kubectl -n <ns> apply  --dry-run=server -f retargeted.yaml        # validates the MERGE with the live object
kubectl -n <ns> create --dry-run=server -o yaml -f one-object.yaml # your object + what the API defaults
```

`apply` inherits omitted fields from the live object, so an omission cannot be caught there. `create`
needs a scratch `metadata.name` **and every self-reference scratched** (`scaleTargetRef.name`,
`advanced.horizontalPodAutoscalerConfig.name`) or an operator's webhook refuses the duplicate claim —
and you are then validating a differently-shaped object than you ship.

Three buckets for a rejection; none is a pass. `field is immutable` (`spec.selector`) is
environmental — the release is installed here under another release name. A type or schema error is
the manifest. `admission webhook "…" denied the request` is usually environmental; read it.

**A green dry run is namespace-scoped.** Pod Security Admission, ResourceQuota, LimitRange and policy
webhooks in the *target* namespace were never consulted — a manifest with no `securityContext` passes
in a permissive namespace and is refused by a `restricted` one.

**Admission success is not reconciliation.** An operator-owned object is inert until claimed, by
bindings the API server does not check: a `KafkaUser` labelled for a Strimzi cluster absent from its
namespace is accepted and yields no SCRAM user; a `PodMonitor` is scraped only if the Prometheus CR's
`podMonitorNamespaceSelector` matches a label on the namespace; a `ScaledObject` can sit with empty
`.status` and no generated HPA (`kubectl get hpa -o jsonpath` on
`.metadata.labels.scaledobject\.keda\.sh/name` shows which HPAs KEDA actually made). Name the binding
each CR is claimed by, and say whether you checked it. Report what you could not reach as **not
verifiable here** — a report with no such category turns unreachable risk into a pass.

## Three ways a chart stops pods dead

**Service links.** The kubelet injects `<SERVICE>_PORT`, `<SERVICE>_SERVICE_HOST` and six more for
every Service in the namespace, uppercased with `-` → `_`. An application that namespaces its own
environment variables by a prefix and rejects unknown names in it — as a strict configuration loader
should — crash-loops as soon as a Service name matches that prefix. Set `enableServiceLinks: false`
on the pod spec; fix it in the chart, the strict parser is worth keeping.

**Probes need the component that serves them.** `/healthz` and `/readyz` are endpoints only if
something binds them. Configuring an address does not enable the module that listens on it — the
generated configuration must turn it on, or the readiness (or startup) probe kills every pod on a
listener nobody opened:

```yaml
"modules" (dict "metrics.prometheus" (dict))    # merged under any user-supplied modules
```

**A stale image.** `kind load image-archive` over an unchanged `:dev` tag plus
`imagePullPolicy: IfNotPresent` leaves the old layer running. Reload *and* replace the pods
(`kubectl -n <ns> rollout restart statefulset/<rel>`), or you debug a fix that never shipped.
Likewise, a ConfigMap edit alone restarts nothing: put a `checksum/config` annotation on the pod
template.

## Per-pod identity in a StatefulSet

One pod spec serves every ordinal, so anything per-pod is built from the pod's own name:

```yaml
env:
  - name: POD_NAME
    valueFrom: { fieldRef: { fieldPath: metadata.name } }
  - name: APP_TLS_CERT_FILE
    value: /etc/app/tls/$(POD_NAME).crt     # $(VAR) expands from an env var defined ABOVE it
```

Mutual TLS between peers usually checks that the certificate's common name is the peer's node name,
so issue **one certificate per ordinal**, not one per release — SANs alone do not satisfy a
common-name check. Generate them at template time and reuse them:

```
{{- $existing := lookup "v1" "Secret" .Release.Namespace $name }}   # reuse on upgrade, else:
{{- $ca := genCA (printf "%s CA" $name) $days }}
{{- range $i := until (int .Values.replicaCount) }}
{{- $pod := printf "%s-%d" $name $i }}
{{- $cert := genSignedCert $pod nil $sans $days $ca }}              # per ordinal, CN = pod name
{{- end }}
```

Keep them in an `Opaque` Secret (`kubernetes.io/tls` holds one pair only) as `ca.crt` plus
`<pod>.crt`/`<pod>.key`. Three traps. A Secret's `type` is immutable, so changing it means deleting
the object, not upgrading over it. `lookup` returns nothing outside a live install, so `helm
template` and the server dry-run render fresh certificates every time and only the install path
exercises reuse. And `helm.sh/resource-policy: keep` — worth keeping, it protects the CA from an
accidental uninstall — means the Secret outlives `helm uninstall` and returns through `lookup`
carrying certificates only for the ordinals that existed when it was written. Reuse is what you
want in production and what masks your fix while iterating: delete the namespace between chart
iterations.

## Throwaway cluster: kind on podman

Creating one is Tier A under the site's ops persona — act freely, delete in the same session.

```sh
export KIND_EXPERIMENTAL_PROVIDER=podman
kind create cluster --name <task> --wait 120s
podman build -t docker.io/library/<img>:dev .
podman save docker.io/library/<img>:dev -o /tmp/<img>.tar
kind load image-archive /tmp/<img>.tar --name <task>            # not `kind load docker-image`
kind delete cluster --name <task>                               # same session, always
```

Tag under `docker.io/library` so the kubelet's short-name resolution finds the image from
`image: <img>:dev`. Fully qualify base images in the `Dockerfile` too — podman resolves a short
name only against configured unqualified-search registries, and many hosts configure none.

Loading a locally built single-architecture image is fine. Loading a multi-arch OCI **index** is
**not** — the loaded record is still an index, and containerd on an arm64 node refuses one that
lists only `amd64`. That case, the node `RLIMIT_NOFILE` inheritance, the disk budget, `max-pods`,
and serial image pulls are all in [`knowledge/tools/kind.md`](../knowledge/tools/kind.md); read it
before standing up a cluster that has to hold a full namespace rather than one chart.

## Drive it from inside the cluster

`kubectl port-forward` binds one pod and dies with it, so it cannot survive the rolling restart it
is meant to measure, and any address the server hands back (`<pod>.<headless>.<ns>.svc`) resolves
only in-cluster. Run acceptance clients as pods: a small client image, the scripts in a ConfigMap,
one pod per role.

Write them to measure the server, not the client library. Two distinct traps:

- Libraries queue a request issued while disconnected and retry it later, so count one only when the
  library reports the server's acknowledgement — not the return of the send call.
- **Read what the acknowledgement said, not just that one arrived.** A blocking confirm typically
  reports the library's own local result, so a negative acknowledgement from the server satisfies it
  and scores as success. Take the server's status code from the per-message callback and count only
  the codes that mean accepted. A client blind to refusals turns "the server refused two thirds of
  these" into "all delivered", and the run then looks like a rare loss instead of a systematic
  refusal.

Wait for a count rather than a fixed sleep, and print disconnect reason codes so a redirect is
observed rather than assumed.

## Site setup

Which host carries kind, kubectl and helm, what is or is not on its non-interactive PATH, and which
ledger to read first are site facts: see the site overlay's knowledge store (`kubernetes-setup.md`)
and its ops persona.

Source: a home-realm Helm work item, 2026-09; the chart and StatefulSet material above is general.
