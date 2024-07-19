# NATS and Kubernetes

```
The recommended way to deploy NATS on Kubernetes is using Helm with the ocial
NATS Helm Chart.
```
```
To register the NATS Helm chart run:
```
```
The default conguration values of the chart will deploy a single NATS server as a
StatefulSet and a single replica nats-box Deployment.
The ArtifactHub page provides the list of Helm conguration values and examples for the
current release.
For tracking the development version, refer to the source repo.
Once the desired conguration is created, install the chart:
```
```
Once the pods are up, validate by accessing the nats-box container and running a CLI
command.
```
```
helm repo add nats https://nats-io.github.io/k8s/helm/charts/
```
```
helm install nats nats/nats
```
```
kubectl exec -it deployment/nats-box -- nats pub test hi
```
## Helm repo

## Config values

## Validate connectivity

7/19/24, 11:28 AM NATS Docs

https://docs.nats.io/~gitbook/pdf?page=EpQjI0ZiK0HNPCwAjAgb&only=yes 1 / 2


```
The output should indicate a successful publish to NATS.
16:17:00 Published 2 bytes to "test"
```
7/19/24, 11:28 AM NATS Docs

https://docs.nats.io/~gitbook/pdf?page=EpQjI0ZiK0HNPCwAjAgb&only=yes 2 / 2


