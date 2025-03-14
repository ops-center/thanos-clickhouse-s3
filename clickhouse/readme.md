### Install Minio:

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

### Setup ClickHouse

#### Create custom config

```bash
kubectl create secret generic -n monitoring my-config-xml --from-file=./custom-config.xml

```
#### Deploy ClickHouse DB

```bash
kubectl apply -f ./clickhouse.yaml
```

### Verify the Setup

```bash
kubectl exec -it -n monitoring ch-0 -- bash
chi-clickhouse-cluster-0-0-0:/# clickhouse-client
```
```sql
CREATE TABLE my_table (
    id UInt32,
    data String,
    event_time DateTime DEFAULT now()
) ENGINE = MergeTree()
ORDER BY id
TTL event_time + INTERVAL 1 MINUTE TO VOLUME 'cold'
SETTINGS storage_policy = 'tiered';
```
```sql
INSERT INTO my_table (id, data) VALUES (1, 'example data');
```
```sql
SELECT
    name,
    disk_name,
    active
FROM system.parts
WHERE table = 'my_table';

```
