# llm-inference-stack

A Helm chart (`vllm-stack`) for deploying a production LLM inference stack on Kubernetes. It combines a [vLLM](https://github.com/vllm-project/vllm) serving engine with an [llm-d](https://github.com/llm-d/llm-d) EndpointPicker (EPP) router for prefix-cache-aware request routing.

## What it deploys

- **vLLM serving engine** — the model server (Deployment, Service, PodDisruptionBudget), one model per Helm release.
- **EPP router** — llm-d EndpointPicker with precise prefix-cache routing, fronted by an Envoy (or agentgateway) proxy.
- **Tokenizer sidecar** — a CPU vLLM (`vllm-render`) sidecar on the EPP pod for prompt tokenization.
- **Observability** — optional ServiceMonitors for the engine and EPP, plus a Grafana dashboard for prefix-cache routing.
- **Networking** — optional Ingress and HTTPRoute (Gateway API / Inference Gateway).
- **S3 registry** — optional credential injection for loading model weights from S3 (e.g. vLLM `runai_streamer`).

## Prerequisites

- Kubernetes cluster (GPU nodes for GPU models)
- Helm 3
- Prometheus Operator (only if enabling ServiceMonitors)

## Usage

One Helm release serves one model. Copy the example values, fill in your model and credentials, then install:

```sh
cp helm/values-example.yaml my-values.yaml
# edit my-values.yaml
helm install my-model ./helm -f my-values.yaml
```

Install the published chart from GHCR:

```sh
helm install my-model oci://ghcr.io/kaasops/charts/vllm-stack --version <version> -f my-values.yaml
```

See `helm/values.yaml` for all configuration options and `helm/values-example.yaml` for a complete example (S3-loaded FP8 model with a custom chat template).

## Repository layout

```
helm/
  Chart.yaml            # chart metadata
  values.yaml           # default values (all options documented inline)
  values-example.yaml   # example per-release values
  templates/            # Kubernetes manifests
  dashboards/           # Grafana dashboard JSON
.github/workflows/      # lint + publish chart to GHCR on tag push
```

## Publishing

The chart is linted on PRs and packaged/pushed to GHCR as an OCI artifact when a `v*` (or `pre-v*`) tag is pushed. The chart version is derived from the tag.
