# poc-dataproc-trino

## Set up variables
```bash
export PROJECT=<project_id>
export REGION=europe-west1
export BUCKET_NAME=dataproc-trino-poc
```

## Create a Dataproc cluster with trino

```bash
gcloud beta dataproc clusters create trino-cluster \
    --project=${PROJECT} \
    --region=${REGION} \
    --zone="" \
    --num-workers=2 \
    --scopes=cloud-platform \
    --optional-components=TRINO \
    --image-version=2.1 \
    --enable-component-gateway \
    --master-machine-type=n1-standard-2 \
    --worker-machine-type=n1-standard-2
```

## Run a query

### Determining Trino server port

The [official tutorial](https://cloud.google.com/dataproc/docs/tutorials/trino-dataproc#trino_cli_queries) only shows us how to query Trino from the coordinator node directly.
To connect from an external client we need to know the Trino server port.

SSH to the coordinator node:
```bash
# zone is allocated automatically when the cluster is created (due to the '--zone=""' flag)
# check the cluster to determine the zone
gcloud compute ssh --zone "europe-west1-d" "trino-cluster-m" --project "${PROJECT}"
```

By inspecting the config file, we can see that Trino server is exposed on the port 8060 on the coordinator node. 
```bash
cat /etc/trino/conf/config.properties
```
Output:
```
coordinator=true
node-scheduler.include-coordinator=false
http-server.http.port=8060
query.max-memory=999TB
discovery-server.enabled=true
discovery.uri=http://trino-cluster-m:8060
web-ui.authentication.type=fixed
web-ui.user=dataproc
#Mon Jan 15 14:03:09 UTC 2024
```

### Connect from an external client via SSH tunnel

Forward local port 8060 to the port 8060 on the coordinator node. 

```bash
gcloud compute ssh trino-cluster-m \
  --project=${PROJECT} \
  --zone=europe-west1-d -- \
    -L 8060:trino-cluster-m:8060 -N -n
```

Now, connect to Trino from your local client:
```bash
trino 'http://localhost:8060' --network-logging=BODY --catalog hive --schema default 
```

Note that requests sent by the trino cli are identifed using the operating system user name (i.e. `X-Trino-User` header is populated):
```
--> GET http://localhost:8060/v1/statement/executing/20240115_151601_00033_jhp3y/yc0d337c6bd7f8ed326e1dddb9483c7749c3b0c09/1 http/1.1
User-Agent: StatementClientV1/436
X-Trino-User: antonpopov
X-Trino-Original-User: antonpopov
Host: localhost:8060
Connection: Keep-Alive
Accept-Encoding: gzip
--> END GET
```

## Hive metastore (HMS)

Config is located at `/usr/lib/trino/etc/catalog/hive.properties`.
Config example:
```
connector.name=hive-hadoop2
hive.metastore.uri=thrift://trino-cluster-m:9083
```
