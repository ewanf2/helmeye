# helmeye

A small command-line tool for checking on a Helm release without stitching together half a dozen `kubectl` commands.

Give it a release name and it finds every Deployment, StatefulSet and DaemonSet that Helm installed for that release, then prints a tree for each one showing:

- the container image
- ready vs. desired replicas
- pod phases (running / pending / failed)
- the status of each container, with the waiting or termination reason when something's wrong
- the three most recent pod events, with warnings highlighted in red

```
[my-app-api]
├─ Kind: Deployment
├─ Image: ghcr.io/example/api:1.4.2
├─ Replicas: 3/3
├─ Pod phases: 2 Running | 1 Pending | 0 Failed
├─ Events
│  ├─ [Normal] Pulled (x1) 2026-10-26 13:02:11 +0000 UTC
│  └─ [Warning] BackOff Back-off restarting failed container (x4) 2026-10-26 13:05:40 +0000 UTC
└─ Container Status
   ├─ [my-app-api-7c9d8-abcde] true
   └─ [my-app-api-7c9d8-fghij] false [CrashLoopBackOff] back-off 40s restarting failed container
```

*(Example output — your colours and values will differ.)*

## Installation

### macOS / Linux

```sh
curl -sSL https://raw.githubusercontent.com/ewanf2/cli_tool/main/install.sh | bash
```

The script picks the right build for your OS and architecture (amd64 or arm64) from the latest GitHub release and installs it to `/usr/local/bin/ht`. It will ask for `sudo` if that directory isn't writable.

### From source

Requires Go 1.25 or newer.

```sh
git clone https://github.com/ewanf2/cli_tool.git
cd cli_tool
go build -o ht .
```

## Usage

```sh
ht watch <release-name>
```

For example:

```sh
ht watch my-app
```

### Kubeconfig and namespace

`ht` uses your normal kubeconfig, the same way `kubectl` does (`$KUBECONFIG`, or `~/.kube/config` by default), and talks to whichever cluster your current context points at.

It looks in the namespace set on your current context. To check a release in a different namespace, switch the context's namespace first:

```sh
kubectl config set-context --current --namespace=<namespace>
```

### How releases are matched

A workload counts as part of a release when it has the label `app.kubernetes.io/managed-by=Helm` and the annotation `meta.helm.sh/release-name` set to the release name. Helm 3 adds both of these automatically, so anything installed with `helm install` should be picked up.

## Releasing

Pushing a tag that starts with `v` (for example `v0.2.0`) runs the GitHub Actions workflow in `.github/workflows/release.yml`, which builds `ht` for Linux and macOS on amd64 and arm64 and attaches the tarballs to a GitHub release. The install script always downloads from the latest release.

```sh
git tag v0.2.0
git push origin v0.2.0
```

## Built with

- [Cobra](https://github.com/spf13/cobra) for the CLI
- [client-go](https://github.com/kubernetes/client-go) for talking to the cluster
- [pterm](https://github.com/pterm/pterm) for the coloured tree output
