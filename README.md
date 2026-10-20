# k8srca-scenario-data

Recorded data for [k8srca](https://github.com/BenedatLLC/k8srca)'s evaluation
scenarios, kept out of that repository's history: a re-recorded capture would
stay there for good.

## Layout

```
metrics/<digest>.tar.gz   a Prometheus metrics capture, as a TSDB block
```

Each file is named by its digest: the first 16 hex digits of its SHA-256. A
scenario's `truth.yaml` pins that digest (`capture.metrics.digest`), and the
scenario runner fetches the file by name from this repository's raw URL, then
checks it. Nothing needs to clone this repository to run a scenario.

A capture is the allowlist of series `promtools capture` reads, for the
scenario's namespaces, up to the moment of the scenario's cluster capture
(k8srca design 009 §5). It is packed reproducibly: sorted, with no times or
owners. The scenario's own directory holds its `manifest.json`: window,
retention and scrape intervals.

## Adding one

From a k8srca checkout, after `k8srca scenario record ... --metrics` and
writing the truth:

```bash
uv run k8srca scenario publish <scenario-id> \
    --data-repo ../k8srca-scenario-data --push
```

Files here are never edited or deleted while a pinned truth names them. A
re-recording adds a new digest.

## What is in a capture

Numbers and labels only: no logs, environment or configuration values. The
labels are Kubernetes names, ids, addresses and enumerations. `promtools
capture` refuses a label name it does not know, because nothing redacts a
capture before it is published here. The data comes from k8srca's demo
cluster (the OpenTelemetry demo on minikube), and the addresses in it are
that cluster's private ones.

No Git LFS: the runner fetches single files over HTTPS, and LFS would serve
it pointer files there instead.
