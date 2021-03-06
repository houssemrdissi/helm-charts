# StreamSets Transformer

[StreamSets Transformer](https://streamsets.com/products/transformer)

StreamSets Transformer is an execution engine within the StreamSets DataOps platform that allows any user to
create data processing pipelines that execute on Spark.

StreamSets Transformer enables users to solve their core business problems without a deep technical understanding
of Apache Spark.

Run StreamSets Transformer as Cloud Native application in Kubernetes Cluster (Using Helm Charts).

The StreamSets Transformer application running inside the Kubernetes cluster can launch transformer pipeline jobs
outside the k8s cluster like Databricks Cluster, EMR Cluster, or Azure HDInsights.

And also, we can run a StreamSets Transformer pipeline using the k8s cluster (Spark on k8s) without any additional spark
installation support.


## Incubating Status

The status of this chart is `incubating` and as such you may still encounter issues.

## Introduction

This chart supports both RBAC and non-RBAC enabled clusters. It has no dependencies.

## Installing the Chart

First, add the StreamSets incubating repository to helm.

```bash
helm repo add streamsets-incubating https://streamsets.github.io/helm-charts/incubating
```

To install the chart into the namespace `streamsets`:

```bash
helm install transformer streamsets-incubating/transformer --namespace streamsets
```

## Configuration

The following tables lists the configurable parameters of the chart and their default values.

| Parameter                       | Description                                                          | Default                                   |
| ------------------------------- | -------------------------------------------------------------------- | ----------------------------------------- |
| `image.repository`              | StreamSets Transformer image name                                    | `streamsets/transformer`                  |
| `image.tag`                     | The version of the official image to use                             | `3.13.0`                         |
| `image.pullPolicy`              | Pull policy for the image                                            | `Always`                            |
|                                 |                                                                      |                                           |
| `controlHub.enabled`            | Control Hub Enabled                                                  | `false`                                   |
| `controlHub.url`                | The URL for the StreamSets Control Hub instance to connect to        | `https://cloud.streamsets.com`            |
| `controlHub.token`              | Transformer auth token from the Control Hub REST API or UI           | None                                      |
| `controlHub.labels`             | Labels for this Transformer to report the Control Hub                | `all`                                     |
|                                 |                                                                      |                                           |
| `ingress.enabled`               | Ingress Enabled                                                      | `false`                                   |
| `ingress.path`                  | Ingress path                                                         | `\`                                       |
|                                 |                                                                      |                                           |
| `transformer.baseHttpUrl`       | Value for configuration "transformer.base.http.url"                  | None                                      |
| `transformer.truststoreFile`    | Transformer Java truststore file which stores certificates           | None                                      |
| `transformer.truststorePassword`| Transformer Java truststore password                                 | None                                      |
|                                 |                                                                      |                                           |
| `rbac.enabled`                  | Creates req'd ServiceAccount and Role on RBAC-enabled cluster        | `true`                                    |
|                                 |                                                                      |                                           |
| `resources`                     | Resource request for the pod                                         | None                                      |
| `nodeSelector`                  | Node Selector to apply to the deployment                             | None                                      |


Specify each parameter using the `--set key=value[,key=value]` argument to `helm install`. For example:

```bash
helm install transformer streamsets-incubating/transformer \
    --set ingress.enabled=true \
    --set transformer.baseHttpUrl=https://192.168.64.58
```

Alternatively, a YAML file that specifies the values for the parameters can be provided while
installing the chart. For example:

```bash
helm install transformer streamsets-incubating/transformer --values values.yaml
```

#### Deleting

The following command functions as expected:
```bash
helm uninstall transformer
```

Yet depending on the install state when this is executed, k8s jobs may be still queued. It is advised to:

```bash
kubectl get all -n streamsets
```

Manually delete all Transformer Spark Driver pods using `kubectl delete pod --field-selector=status.phase==Succeeded`

## Custom Configuration Tips

### Deploying StreamSets Transformer in minikube

```bash
helm install transformer streamsets-incubating/transformer \
    --set ingress.enabled=true \
    --set transformer.baseHttpUrl=https://$(minikube ip)
```

For custom path-based routes (https://192.168.64.69/transformer)

```bash
helm install transformer streamsets-incubating/transformer \
    --set ingress.enabled=true \
    --set ingress.path=/transformer \
    --set transformer.baseHttpUrl=https://$(minikube ip)/transformer/
```

### Deploying StreamSets Transformer in minikube with Control Hub enabled

Generate StreamSets Transformer auth token from URL - https://cloud.streamsets.com/sch/security/transformers

```bash
token = <Transformer auth token from the Control Hub REST API or UI>
helm install transformer streamsets-incubating/transformer \
    --set controlHub.enabled=true \
    --set controlHub.url=https://cloud.streamsets.com \
    --set controlHub.token=${token} \
    --set ingress.enabled=true \
    --set transformer.baseHttpUrl=https://$(minikube ip)
```


### Deploying StreamSets Transformer in minishift

Route URL format - https:// <Service Name>-<Project/Namespace Name>.$(minishift ip).nip.io
e.g: https://transformer-myproject.192.168.64.59.nip.io

```bash
minishift start --memory 12GB --cpus 4
oc login -u system:admin
oc adm policy add-scc-to-user anyuid -z transformer
oc adm policy add-scc-to-group hostmount-anyuid system:serviceaccounts
oc login -u developer
helm install transformer streamsets-incubating/transformer \
    --set route.enabled=true \
    --set transformer.baseHttpUrl=https://transformer-myproject.$(minishift ip).nip.io
```


### Deploying StreamSets Transformer in minishift with Control Hub enabled

Generate StreamSets Transformer auth token from URL - https://cloud.streamsets.com/sch/security/transformers

Route URL format - https://<Service Name>-<Project/Namespace Name>.$(minishift ip).nip.io
e.g: https://transformer-myproject.192.168.64.59.nip.io


```bash
minishift start --memory 12GB --cpus 4
oc login -u system:admin
oc adm policy add-scc-to-user anyuid -z transformer
oc adm policy add-scc-to-group hostmount-anyuid system:serviceaccounts
oc login -u developer
token = <Transformer auth token from the Control Hub REST API or UI>
helm install transformer streamsets-incubating/transformer \
    --set controlHub.enabled=true \
    --set controlHub.url=https://cloud.streamsets.com \
    --set controlHub.token=${token} \
    --set route.enabled=true \
    --set transformer.baseHttpUrl=https://transformer-myproject.$(minishift ip).nip.io
```

### Deploying StreamSets Transformer in Azure Kubernetes Service (AKS)

Creating AKS cluster

```bash
az login
az aks create --name kubcluster-transformer --resource-group transformerResourceGroup
```

Install Traefik load balancer & StreamSets Transformer

```bash
## Install Traefik load balancer
helm install traefik stable/traefik \
    --set rbac.enabled=true \
    --set kubernetes.ingressClass=traefik \
    --set kubernetes.ingressEndpoint.useDefaultPublishedService=true \
    --set ssl.enabled=true \
    --set ssl.generateTLS=true \
    --version 1.85.0

## Get Traefik's load balancer IP/hostname:
external_ip=$(kubectl describe svc traefik | grep Ingress | awk '{print $3}')

# Extract certificate from Traefik Ingress IP
echo | openssl s_client -connect ${external_ip}:443 | sed -ne '/-BEGIN CERTIFICATE-/,/-END CERTIFICATE-/p' > ingress.crt

# Import Ingress HTTPs certificate in to the truststore.jks
keytool -import -file ingress.crt -trustcacerts -noprompt -alias IngressCA -storepass password -keystore truststore.jks

# Copy the CA certs from jre/lib/security/cacerts to etc/truststore.jks
keytool -importkeystore -srckeystore "$JAVA_HOME"/jre/lib/security/cacerts -srcstorepass changeit -destkeystore truststore.jks -deststorepass password

helm install transformer streamsets-incubating/transformer \
    --set ingress.enabled=true \
    --set ingress.annotations."kubernetes\.io/ingress\.class"="traefik" \
    --set transformer.baseHttpUrl=https://${external_ip} \
    --set transformer.truststoreFile=truststore.jks \
    --set transformer.truststorePassword=password
```
