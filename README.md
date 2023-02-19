# elastic-jakarta-meetup-feb-2023


## deploy elastic operator
```bash
kubectl create -f https://download.elastic.co/downloads/eck/2.6.1/crds.yaml
kubectl apply -f https://download.elastic.co/downloads/eck/2.6.1/operator.yaml
```
`you can skip if you already have elastic operator installed`

## Deploy single-node elasticsearch

```bash
cat <<EOF | kubectl apply -f -
apiVersion: elasticsearch.k8s.elastic.co/v1
kind: Elasticsearch
metadata:
  name: my-elastic
spec:
  version: 8.5.0
  nodeSets:
  - name: default
    count: 1
    config:
      node.store.allow_mmap: false
EOF
```

add kibana
```bash
cat <<EOF | kubectl apply -f -
apiVersion: kibana.k8s.elastic.co/v1
kind: Kibana
metadata:
  name: my-kibana
spec:
  version: 8.5.0
  count: 1
  elasticsearchRef:
    name: my-elastic
EOF
```
## Login to Kibana

get elastic password

```
kubectl get secret/es-meetup-es-elastic-user --template={{.data.elastic}} | base64 
```

get Kibana IP

```
kubectl get pod -o wide
kubectl get svc/kibana-kb-http --no-headers | awk '{print $4}'
```


## Scaling Out Elasticsearch

`kubectl edit elasticsearch/es-meetup` and change `count: 1` to `count: 3`

or 
edit `01.elastic-deployment.yaml` and change `count: 1` to `count: 3` 

```
vim 01.elastic-deployment.yaml
change 'count: 1' to 'count: 3'
kubectl apply -f 01.elastic-deployment.yaml
```

## Enable Observability 
create cluster role & role binding for elastic-agent and fleet server
```bash
kubectl apply -f 03.rbac-elastic-agent.yaml
```
add fleet server
```bash
kubectl apply -f 04.fleet-server.yaml
```

add elastic-agent
```bash
kubectl apply -f 05.elastic-agent.yaml
```
## Upgrade the cluster

change version in every yaml file from `8.5.0` to `8.6.1`

## More Configuration

### Persistent Volume
this deployment doesn't have a persistent data. If you delete the cluster, the data will be deleted along with the cluster. To retain the data you need to specify volumeClaim from Kubernetes 

you can add this configuration on `01.elastic-deployment.yaml` in line 12
```yaml
    volumeClaimTemplates:
    - metadata:
        name: elasticsearch-data 
      spec:
        accessModes:
        - ReadWriteOnce
        resources:
          requests:
            storage: 5Gi
        storageClassName: standard
```