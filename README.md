# TensorFusion Remote/Local vGPU Benchmark Helm Chart

This Helm chart deploys the TensorFusion Remote/Local vGPU Benchmark application, which includes a deployment for running the benchmark tests and a cronjob for automated testing.

## Benchmark Results

### TorchBenchmark Results (2025-07-06)

To run the TorchBenchmark tests:
```bash
cd benchmark
python3 test.py -k "test_${model_name}_eval_cuda" -t ${eval_times}
```

| Model | Native | NGPU Mode | Loss(NGPU) | Local | Loss(Local) | Same AZ | Loss(Same AZ) | Cross AZ | Loss(Cross AZ) |
|-------|---------|------------|------------|--------|-------------|---------|--------------|----------|----------------|
| basic_gnn_edgecnn | 41.15 s | 40.95 s | -0.49% | 43.48 s | 5.66% | 46.07 s | 11.96% | 54.97 s | 33.58% |
| BERT_pytorch | 249.02 s | 248.84 s | -0.07% | 251.26 s | 0.90% | 253.71 s | 1.88% | 261.62 s | 5.06% |
| basic_gnn_gcn | 15.05 s | 15.24 s | 1.26% | 19.63 s | 30.43% | 29.70 s | 97.34% | 64.39 s | 327.84% |
| basic_gnn_gin | 9.47 s | 9.53 s | 0.63% | 9.78 s | 3.27% | 12.66 s | 33.69% | 21.83 s | 130.52% |
| hf_Albert | 24.73 s | 24.00 s | -2.95% | 29.19 s | 18.03% | 39.19 s | 58.47% | 73.00 s | 195.19% |
| hf_Bart | 39.88 s | 38.68 s | -3.01% | 54.96 s | 37.81% | 94.17 s | 136.13% | 211.68 s | 430.79% |
| hf_Bert | 24.15 s | 24.35 s | 0.83% | 29.55 s | 22.36% | 42.00 s | 73.91% | 75.86 s | 214.12% |
| llama | 39.91 s | 41.20 s | 3.23% | 42.90 s | 7.49% | 45.80 s | 14.76% | 52.55 s | 31.67% |
| hf_distil_whisper | 170.61 s | 170.87 s | 0.15% | 172.16 s | 0.91% | 178.75 s | 4.77% | 189.45 s | 11.04% |
| hf_clip | 191.60 s | 191.70 s | 0.05% | 194.52 s | 1.52% | 197.51 s | 3.08% | 208.90 s | 9.03% |
| hf_Whisper | 58.98 s | 59.18 s | 0.34% | 63.50 s | 7.66% | 66.66 s | 13.02% | 72.63 s | 23.14% |
| **Average Loss** | - | - | **0.00%** | - | **12.37%** | - | **40.82%** | - | **128.36%** |

### MLPerf Results (2025-07-04)

To run the MLPerf benchmark:
```bash
mlcr run-mlperf,inference,_full,_r5.0-dev \
    --model=bert-99 \
    --implementation=reference \
    --framework=pytorch \
    --category=edge \
    --scenario=SingleStream \
    --execution_mode=valid \
    --device=cuda \
    --quiet --rerun
```

| Mode | Time | Loss |
|------|------|------|
| Native | 27.008 s | - |
| Local | 29.930 s | 10.82% |
| Same AZ | 33.341 s | 23.45% |
| Cross AZ | 41.597 s | 54.02% |

### Simulating AZ Latencies

To simulate different AZ (Availability Zone) network conditions, you can use the Linux Traffic Control (tc) tool to inject artificial network latency:

1. Inject network latency:
```bash
# For Same AZ simulation (0.3ms latency)
tc qdisc add dev lo root netem delay 0.3ms

# For Cross AZ simulation (1ms latency)
tc qdisc add dev lo root netem delay 1ms
```

2. Verify the latency:
```bash
ping target_host
```

3. Remove the artificial latency when done:
```bash
tc qdisc del dev lo root
```

## Prerequisites

- Kubernetes 1.19+
- Helm 3.2.0+
- PV provisioner support in the underlying infrastructure
- A GPU node with NVIDIA drivers installed

## Installing the Chart

To install the chart with the release name `my-release`:

```bash
helm install my-release ./helm/torchbench
```

The command deploys the benchmark application on the Kubernetes cluster with default configuration.

## Configuration

The following table lists the configurable parameters of the chart and their default values.

| Parameter | Description | Default |
|-----------|-------------|---------|
| `replicaCount` | Number of replicas | `1` |
| `image.repository` | Image repository | `crpi-wpzfqfci37r0ad3n.cn-hangzhou.personal.cr.aliyuncs.com/tensorfusionrobin/tensorfusionrobin` |
| `image.tag` | Image tag | `latest` |
| `image.pullPolicy` | Image pull policy | `Always` |
| `serviceAccount.create` | Create service account | `true` |
| `serviceAccount.name` | Service account name | `cronjob-sa` |
| `podAnnotations` | Pod annotations | See values.yaml |
| `podLabels` | Pod labels | See values.yaml |
| `resources` | Pod resource requests and limits | See values.yaml |
| `nodeSelector` | Node selector | `kubernetes.io/hostname: gpu-2` |
| `cronjob.schedule` | Cronjob schedule | `0 0 * * *` |
| `cronjob.concurrencyPolicy` | Cronjob concurrency policy | `Allow` |
| `cronjob.successfulJobsHistoryLimit` | Number of successful jobs to keep | `3` |
| `cronjob.failedJobsHistoryLimit` | Number of failed jobs to keep | `1` |

## Usage

### Running the Benchmark

The benchmark will run automatically according to the cronjob schedule. You can also manually trigger a benchmark run by:

1. Finding the cronjob:
```bash
kubectl get cronjob
```

2. Creating a job from the cronjob:
```bash
kubectl create job --from=cronjob/my-release-torchbench-test-runner manual-run
```

### Viewing Results

To view the benchmark results:

```bash
kubectl logs -l app=my-release-torchbench
```

### Customizing the Configuration

To customize the configuration, create a custom values file:

```bash
helm install my-release ./helm/torchbench -f custom-values.yaml
```

## Uninstalling the Chart

To uninstall/delete the deployment:

```bash
helm uninstall my-release
```

## Troubleshooting

If you encounter any issues:

1. Check the pod status:
```bash
kubectl get pods -l app=my-release-torchbench
```

2. Check the pod logs:
```bash
kubectl logs -l app=my-release-torchbench
```

3. Check the cronjob status:
```bash
kubectl get cronjob
kubectl get jobs
```

4. Check the service account:
```bash
kubectl get serviceaccount
``` 
