# curl — signed S3 requests against Linode Object Storage

Reference material, non-normative. Policy lives in [`skills/tooling.md`](../../skills/tooling.md).

Recorded 2026-09-09 from remote-ops WI 1380, where `curl` replaced a boto3-in-a-
virtualenv call in `ro-home-backup`'s failure path. That skill names raw `curl`
against an API with a supported CLI as a deviation; the case for the deviation is
recorded in that work item's plan, not here. What follows is the mechanics.

## Why it comes up at all

`curl` is a distro package outside `$HOME`. A `linode-cli`/`boto3` virtualenv under
`~/.local/venvs` is not, so on a host restored from a home-directory backup that does
not carry the venv, the CLI is gone and `curl` is not. That is the only situation in
which reaching for `curl` here is the *more* reliable choice.

## Signing

`--aws-sigv4 "aws:amz:<region>:s3"` produces an AWS SigV4 signature. Against
`us-iad-10.linodeobjects.com` this works with the ordinary object-storage
Access Key / Secret Key pair.

**The region string is not validated.** `us-iad-10`, `us-iad` and `us-east-1` each
returned HTTP 200 for the same request. Pin the actual cluster id anyway — the
laxity is Linode's, not something to depend on.

Requires curl ≥ 7.75. Ubuntu 24.04 ships 8.5.0, which has it. Probe rather than
assume:

```sh
curl --help all 2>/dev/null | grep -q -- '--aws-sigv4'
```

## Keeping the secret out of the process table

`curl -u "$AK:$SK"` puts the secret in `argv`, where any user on the box can read it
from `ps`. Pass a config stanza on standard input instead:

```sh
printf 'user = "%s:%s"\n' "$AK" "$SK" | curl -K - --aws-sigv4 "aws:amz:us-iad-10:s3" ...
```

`-K -` reads curl's own config-file syntax from stdin. Verified working against the
live endpoint.

## PUT with a body, when stdin is already taken

`-K -` consumes stdin, so `--data-binary @-` cannot also have it. Use
`--upload-file FILE`, which issues a PUT and reads the body from a file:

```sh
body=$(mktemp); printf '%s\n' "$message" > "$body"
code=$(printf 'user = "%s:%s"\n' "$AK" "$SK" \
  | curl -sS -o /dev/null -w '%{http_code}' -K - \
      --aws-sigv4 "aws:amz:us-iad-10:s3" \
      --upload-file "$body" "https://us-iad-10.linodeobjects.com/$bucket/$key")
rm -f "$body"
```

Routing the body through a file has a second benefit worth the temp file: the message
is never parsed as source. The code this replaced interpolated a shell variable into
an unquoted Python heredoc, so a failure message containing an apostrophe would have
raised `SyntaxError` instead of writing the marker.

`-w '%{http_code}'` is the outcome check. `curl` exits 0 on a 403, so testing `$?`
alone reports success on a rejected request.

## Listing (read-only probe)

```sh
curl -s --aws-sigv4 "aws:amz:us-iad-10:s3" -K - \
  "https://us-iad-10.linodeobjects.com/$bucket/?list-type=2&prefix=$prefix/"
```

Returns S3 ListObjectsV2 XML. Useful as the "one read-only call" that
`skills/tooling.md` asks for before depending on a tool.
