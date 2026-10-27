# k8srca-scenario-data

Recorded data for [k8srca](https://github.com/BenedatLLC/k8srca)'s evaluation
scenarios, kept out of that repository's history: a re-recorded capture would
stay there for good.

## Layout

```
captures/<digest>.json.gz   a k8stools cluster capture (`k8s-capture-state`)
skills/<digest>.tar.gz      a scenario's pinned cluster-architecture skill
metrics/<digest>.tar.gz     a Prometheus metrics capture, as a TSDB block
```

Each file is named by a digest: the first 16 hex digits of a SHA-256.
- **A capture** is named by the digest of its JSON, before gzip. The
  scenario's `scenario.yaml` pins it (`sources.k8stools.digest`).
- **A skill** is named by its bundle digest, the one k8srca computes over the
  skill's files, and the scenario's `truth.yaml` pins it
  (`capture.skill_digests` or `capture.skill_digest`).
- **A metrics block** is named by the digest of its tarball, and `truth.yaml`
  pins it (`capture.metrics.digest`).

The scenario runner fetches a file by name from this repository's raw URL, or
from a local checkout, then checks it. Nothing needs to clone this repository
to run a scenario. `uv run k8srca scenario fetch` fetches every scenario's
files at once.

A capture is the allowlist of series `promtools capture` reads, for the
scenario's namespaces, up to the moment of the scenario's cluster capture
(k8srca design 009 §5). It is packed reproducibly: sorted, with no times or
owners. The scenario's own directory holds its `manifest.json`: window,
retention and scrape intervals.

## Adding one

From a k8srca checkout, after `k8srca scenario record` (with `--metrics` if
the scenario has them) and writing the truth:

```bash
uv run k8srca scenario publish <scenario-id> \
    --data-repo ../k8srca-scenario-data --push
```

Files here are never edited or deleted while a pinned truth names them. A
re-recording adds a new digest.

## What is in a cluster capture

The k8stools capture of the scenario's namespaces: object summaries, events and
container logs. `k8s-capture-state` redacts secrets in it before it is written,
as it does for every capture.

## What is in a metrics capture

Numbers and labels only: no logs, environment or configuration values. The
labels are Kubernetes names, ids, addresses and enumerations. `promtools
capture` refuses a label name it does not know, because nothing redacts a
capture before it is published here. The data comes from k8srca's demo
cluster (the OpenTelemetry demo on minikube), and the addresses in it are
that cluster's private ones.

No Git LFS: the runner fetches single files over HTTPS, and LFS would serve
it pointer files there instead.
