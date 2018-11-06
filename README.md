# Tillerless Helm plugin

[Helm](https://helm.sh) plugin for using [Tiller](https://docs.helm.sh/using_helm/#installing-tiller) locally and in your CI/CD pipelines.

My blog [post](https://rimusz.net/tillerless-helm/) on why `Tillerless Helm` is needed and what it solves.

**Note:** For a better security Tiller plugin comes with preset storage as `Secret`.

## Installation

Install Helm client as per one of recomended [ways]()https://docs.helm.sh/using_helm/#installing-the-helm-client.

**Note:** Initialize helm with `helm init --client-only`, flag `--client-only` is a must as otherwise you will get `Tiller` installed in to Kubernetes cluster.

Then install the latest plugin version:

```console
helm plugin install https://github.com/tiancai-hq/helm-tiller
```

## Usage

Usage:

```console
helm tiller install
helm tiller start [tiller_namespace]
helm tiller start-ci [tiller_namespace]
helm tiller stop
helm tiller run [tiller_namespace] -- [command] [args]

Available Commands:
install   Manually install/upgrade Tiller binary
start     Start Tiller and open new pre-set shell
start-ci  Start Tiller without opening new shell
run       Start Tiller and run arbitrary command within the environment
stop      Stop Tiller
```

Available environment variables:

- To silence plugin specific messages by setting `HELM_TILLER_SILENT=true`, only `helm` cli output will be printed.
- To change default Tiller port by setting `HELM_TILLER_PORT=44140`, default is `44134`.
- To change Tiller storage to `configmap` by setting `HELM_TILLER_STORAGE=configmap`, default is `secret`.
- To store Tiller logs in `$HOME/.helm/plugins/helm-tiller/logs` by setting `HELM_TILLER_LOGS=true`.
- You can set a specific folder/file for Tiller logs by setting `HELM_TILLER_LOGS_DIR=/some_folder/tiller.logs`.
- To change default Tiller maximum number of releases kept in release history by setting e.g. to 20 `HELM_TILLER_HISTORY_MAX=20`.

### Tiller start examples

Start Tiller with pre-set `bash` shell `HELM_HOST=127.0.0.1:44134`, it is handy to use locally:

```console
helm tiller start
```

The default working Tiller `namespace` is `kube-system`, you can set another one:

```console
helm tiller start my_tiller_namespace
```

> **Tip**: You can have many Tiller namespaces, e.g. one per team, just pass the name as an argument when you starting Tiller.

In CI pipelines you do not really need pre-set bash to be opened, so you can use:

```console
helm tiller start-ci
export HELM_HOST=127.0.0.1:44134
```

Then your `helm` will know where to connect to Tiller and you do not need to make any changes in your CI pipelines.

And when you done stop the Tiller:

```console
helm tiller stop
```

### Tiller run examples

Another option for CI workflows.

Examples use of `tiller run`, that starts/stops `tiller` before/after the specified command:

```console
helm tiller run helm list
helm tiller run my-tiller-namespace -- helm list
helm tiller run my-tiller-namespace -- bash -c 'echo running helm; helm list'
```

Handy `bash` aliases for use `Tillerless` locally:

```
alias hh="helm tiller run helm"
alias hr="helm tiller run"
alias ht="helm tiller start"
alias hts="helm tiller stop"
```

Examples of alias use:

```console
# helm tiller run helm list
hh ls

# helm tiller run my-tiller-namespace -- helm list
hr my-tiller-namespace -- helm list

# helm tiller run my-tiller-namespace -- bash -c 'echo running helm; helm list'
hr my-tiller-namespace -- bash -c 'echo running helm; helm list'
```

## Tiller binaries

### Tiller binaries for v2.11 Helm and above versions

Beginning of Helm v2.11 release, `helm` archive file comes packed with `tiller` binary as well.
Plugin will check the version and download the right archive file. No more building/retrieving of
`tiller` binary is needed anymore.

### Build/retrieve Tiller binaries and publish them for v2.10 Helm

To build `MacOS` and to retrieve `Linux` binaries and then publish them to `GCS` bucket run on your Mac:

```console
TILLER_VERSION=2.10.0 GCS_BUCKET=my_bucket make build
```

### Build patched Tiller binaries and publish them for pre v2.10 Helm

**Note:** `Tiller`in pre `v2.10` does not support kubeconfig files which use user authentication via `auth-provider`, so you need to use this approach for all pre `v2.10` `tiller` releases.

To build patched `MacOS` and `Linux` `tiller` binaries and then publish them to `GCS` bucket run on your Mac:

```console
TILLER_VERSION=2.9.1 GCS_BUCKET=my_bucket make build-patch
```
