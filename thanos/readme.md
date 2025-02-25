### Install Minio

```bash
helm repo add minio https://operator.min.io/
````
```bash
helm upgrade --install --namespace minio-operator \
  --create-namespace minio minio/operator \
  --set operator.replicaCount=1 \
  --wait
```


```bash
helm upgrade --install --namespace minio --create-namespace minio-tenant minio/tenant \
  --set tenant.pools[0].servers=1 \
  --set tenant.pools[0].volumesPerServer=1 \
  --set tenant.pools[0].size=1Gi \
  --set tenant.certificate.requestAutoCert=false \
  --set tenant.pools[0].name="default" \
  --set tenant.buckets[0].name="test" \
  --wait
```

#### Accessing MinIO

- Access Key: `minio`
- Secret Key: `minio123`
- Endpoint: `http://minio.minio.svc.cluster.local:80`


### Install Kube Prometheus Stack

```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
```

```bash
helm install kube-prometheus-stack prometheus-community/kube-prometheus-stack \
  --namespace monitoring \
  --create-namespace \
  -f kps-values.yaml
```

### Create Thanos Storage Secret

```bash
kubectl -n monitoring create secret generic thanos-objstore-config --from-file=thanos-storage-config.yaml=s3.yaml
```

### Install Thanos Components

```bash
git clone git@github.com:thanos-io/kube-thanos.git
```

```bash
kubectl create namespace thanos

```

#### Configure Thanos Query to Connect to Thanos Sidecars

To enable communication between thanos-query and thanos-sidecar, add the following argument to the Thanos Query container's args:

```bash
--store=dnssrv+_grpc._tcp.kube-prometheus-stack-thanos-discovery.monitoring.svc.cluster.local:10901

```

This ensures that Thanos Query can discover and connect to Thanos Sidecar instances via GRPC.


#### Create Thanos Storage Secret

```bash
kubectl -n thanos create secret generic thanos-objstore-config --from-file=thanos-storage-config.yaml=s3.yaml
```

#### Deploy Thanos Manifests

```bash
kubectl apply -f .kube-thanos/manifests -n thanos
```

### Verify the Setup

#### Query metrics via Thanos Querier

```bash
 kubectl port-forward svc/thanos-query -n thanos 9090:9090
```
Visit http://localhost:9090 and execute a test query (e.g., up).


### Configure Grafana to Use Thanos Querier
Add Thanos Querier as a Prometheus data source in Grafana:

URL: `http://thanos-query.thanos.svc.cluster.local:9090`
