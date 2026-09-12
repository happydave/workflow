# kind (Kubernetes in Docker) — reference

Non-normative reference for the `kind` CLI. Policy lives in `skills/tooling.md`; the throwaway-cluster
recipe and the host rules live in `skills/kubernetes.md`.

## Facts that bit

- **Context switching.** `kind create cluster` sets `kubectl` current-context to `kind-<name>`. A tool that runs
  later commands without `--context` will act on the new cluster — or, after the user switches back, on whatever is
  current. Always pass `--context kind-<name>` (kubectl) and `--kube-context` (helm); restore the user's previous
  context after create if the tool created the cluster.
- **Node process limits.** The `kindest/node` image runs containerd under systemd with `LimitNOFILE=infinity`, so
  every pod inherits `RLIMIT_NOFILE` = 1,073,741,816 (plain Docker gives 1024). Software that sizes tables from
  that limit (uWSGI) allocates gigabytes at start. Fix: a drop-in on the node
  (`podman exec <node> sh -c 'mkdir -p /etc/systemd/system/containerd.service.d && printf "[Service]\nLimitNOFILE=1048576\n" > …/10-nofile.conf && systemctl daemon-reload && systemctl restart containerd'`);
  restarting containerd keeps running containers. Pods created before the change keep the old limit.
- **Platform matching.** containerd on an arm64 node refuses an OCI index that lists only `amd64` + `unknown`
  (`no match for platform in manifest`), regardless of the host's emulator. Workaround: pull the amd64 manifest by
  digest inside the node (`ctr -n k8s.io images pull --platform linux/amd64 <name>@<digest>`) and
  `ctr -n k8s.io images tag --force <name>@<digest> <name>:<tag>`. Loading an image index into kind does **not**
  help — the loaded record is still an index.
- **Disk.** Node image layers live in a volume under the container engine's data root, not on the host; a full
  namespace's chart set can unpack to tens of GiB there. Budget ≥ 30 GiB free before `create`.
- **max-pods.** kubelet default is 110; a platform namespace can need ~120 plus operators. Set via
  `kubeadmConfigPatches` → `kubeletExtraArgs: {max-pods: "250"}`.
- **Serial image pulls.** kubelet pulls one image at a time per node (`serializeImagePulls` default true). With ~100
  pods on one node, a pod's first pull attempt can come 5–10 minutes after its creation, so anything that fixes
  images on failure (the index retag above) must keep running long after the deploy command has exited — and must
  retry: a registry has answered `not found` once for a digest present in its own index.
- **Two `stop()`s on one channel-based companion hang forever.** A `defer stop()` plus an explicit `stop()` before
  the final phase is the natural shape for a helper goroutine pair; make `stop` idempotent (`sync.Once` over
  `cancel(); wg.Wait()`), or the process prints its final result and never exits.
- **Ingress ports.** Map `extraPortMappings` 30080/30443 and run ingress-nginx as NodePort on those.
