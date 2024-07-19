# NATS and Kubernetes

The recommended way to deploy NATS on Kubernetes is using Helm with the official NATS Helm Chart.

## Register the NATS Helm Chart

To register the NATS Helm chart, run:

```bash
helm repo add nats https://nats-io.github.io/k8s/helm/charts/
```

## Install the NATS Helm Chart

The default configuration values of the chart will deploy a single NATS server as a StatefulSet and a single replica `nats-box` Deployment.

To install the chart with the desired configuration, run:

```bash
helm install nats nats/nats
```

## Validate Connectivity

Once the pods are up, validate by accessing the `nats-box` container and running a CLI command:

```bash
kubectl exec -it deployment/nats-box -- nats pub test hi
```

The output should indicate a successful publish to NATS:

```bash
16:17:00 Published 2 bytes to "test"
```

For a comprehensive list of Helm configuration values and examples for the current release, refer to the [ArtifactHub page](https://artifacthub.io/packages/helm/nats/nats).

To track the development version, refer to the [source repo](https://github.com/nats-io/k8s).

## Summary

Deploying NATS on Kubernetes using Helm simplifies the process, providing a scalable and manageable solution. By registering the Helm chart, installing it with the desired configuration, and validating connectivity through `nats-box`, you can efficiently deploy and manage NATS clusters on Kubernetes.