# Overview 

This document will help you to build elasticsearch cluster on Kubernetes. 

# Setup

If you have already kubernetes cluster running then you can simply go to the Deployment steps. 

And if you want to quickly setup Dev kubernetes on your laptop, you can use following command to spin Kubernetes. 


```shell
minikube start  --kubernetes-version v1.8.0 --cpus 6 --memory 8192 --extra-config=apiserver.GenericServerRunOptions.AuthorizationMode=RBAC --extra-config=apiserver.GenericServerRunOptions.AuthorizationRBAC,SuperUser=minikube
```


# Architecture

Elasticsearch best-practices recommend to separate nodes in three roles:

 - **Master nodes** :  intended for clustering management only, no data, no HTTP API
 - **Client nodes** : intended for client usage, no data, with HTTP API
 - **Data nodes** : intended for storing and indexing data, no HTTP API


# Deployment 

### Setup Master

```shell
kubectl create -f es-discovery-svc.yml
kubectl create -f es-svc.yml
kubectl create -f es-master-deployment.yml
```

### Setup Client

```shell
kubectl create -f es-client-deployment.yml
```

### Setup data nodes

```shell
kubectl create -f es-data-statefulset.yml
```

# Access the service

```shell
kubectl run -it  --restart=Never --rm  --image=tutum/curl   curl-tmp -- curl http://elasticsearch:9200/_cluster/health?pretty
```


You will get following output

```
{
  "cluster_name" : "picnic-es",
  "status" : "green",
  "timed_out" : false,
  "number_of_nodes" : 4,
  "number_of_data_nodes" : 2,
  "active_primary_shards" : 0,
  "active_shards" : 0,
  "relocating_shards" : 0,
  "initializing_shards" : 0,
  "unassigned_shards" : 0,
  "delayed_unassigned_shards" : 0,
  "number_of_pending_tasks" : 0,
  "number_of_in_flight_fetch" : 0,
  "task_max_waiting_in_queue_millis" : 0,
  "active_shards_percent_as_number" : 100.0
}
```


# Scale ES cluster 

```shell
kubectl scale --replicas=3 deployment es-master
kubectl scale --replicas=2 deployment es-client
kubectl scale --replicas=2 statefulset es-data
```

