# TensorFusion Remote/Local vGPU Benchmark Helm Chart

This Helm chart deploys the TensorFusion Remote/Local vGPU Benchmark application, which includes a deployment for running the benchmark tests and a cronjob for automated testing.

## Benchmark Results

### TorchBenchmark Results (2025-07-04)

To run the TorchBenchmark tests:
```bash
cd benchmark
python3 test.py -k "test_${model_name}_eval_cuda" -t ${eval_times}
```

| Model | Native | NGPU Mode | Loss(NGPU) | Local | Loss(Local) | Same AZ | Loss(Same AZ) | Cross AZ | Loss(Cross AZ) |
|-------|---------|------------|------------|--------|-------------|---------|--------------|----------|----------------|
| basic_gnn_edgecnn | 76.09 s | 76.01 s | -0.11% | 77.88 s | 2.35% | 81.05 s | 6.52% | - | - |
| BERT_pytorch | 481.82 s | 479.31 s | -0.52% | 476.19 s | -1.17% | 477.08 s | -0.98% | - | - |
| basic_gnn_gcn | 16.23 s | 16.32 s | 0.55% | 20.31 s | 25.14% | 31.58 s | 94.58% | - | - |
| basic_gnn_gin | 13.06 s | 12.81 s | -1.91% | 13.69 s | 4.82% | 17.01 s | 30.25% | - | - |
| hf_Albert | 38.27 s | 36.82 s | -3.79% | 38.66 s | 1.02% | 45.24 s | 18.21% | - | - |
| hf_Bart | 54.29 s | 53.81 s | -0.88% | 65.62 s | 20.87% | 102.89 s | 89.52% | - | - |
| hf_Bert | 33.95 s | 33.33 s | -1.83% | 37.57 s | 10.66% | 48.47 s | 42.77% | - | - |
| llama | 62.04 s | 60.60 s | -2.32% | 61.45 s | -0.95% | 62.55 s | 0.82% | - | - |
| hf_distil_whisper | 175.42 s | 175.95 s | 0.30% | 237.67 s | 35.49% | 375.54 s | 114.08% | 375.54 s | - |
| hf_clip | 358.16 s | 356.68 s | -0.41% | 354.35 s | -1.06% | 355.09 s | -0.86% | 371.04 s | 3.60% |
| hf_Whisper | 59.91 s | 59.99 s | 0.13% | 73.62 s | 22.88% | 100.14 s | 67.15% | 177.26 s | 195.88% |
| **Average Loss** | - | - | **-0.98%** | - | **10.91%** | - | **42.00%** | - | - |

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
