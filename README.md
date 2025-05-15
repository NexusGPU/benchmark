# TensorFusion Remote/Local vGPU Benchmark Helm Chart

This Helm chart deploys the TensorFusion Remote/Local vGPU Benchmark application, which includes a deployment for running the benchmark tests and a cronjob for automated testing.

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