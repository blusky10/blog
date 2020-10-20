---
title: ReplicaSet
tags: ["K8S", "ReplicaSet", "RS", "쿠버네티스"] 
categories: ["Kubernetes"]
date: 2020-10-20 13:33:16
---
## ReplicaSet vs Replication Controller

ReplicaSet 은 Replicatation Controller 의 새로운 버전이다.

ReplcaSet : Set-based Selectors

Replicatation Controller : Equality-based Selectors

|           | support | Operation | Example | Command Line | manifest |
|:----------|:----------|:------------- | ------------ |------------- | -------------|
| Equality-based | Service, Replication Controller | = == != | enviroment=prd | kubectl get pods -l enviroment=prd | selector: <br/> enviroment: prd |
| Set-based  | Job, Deployment, ReplicaSet, Daemon Set| in notin exists | enviroment in (prd) | kubectl get pods -l 'enviroment in (prd)' | selector:<br/> matchExpressions:<br/>- {key: enviroment, operation: In, values: [prd]}|

### matchLabels

selectors 에 matchLabels 가 존재할 경우 새로운 리소스들까지 지원을 해준다. 

| manifest | support |
|:----------|:----------|
| selector:<br/> app: nginx <br/> | Services, Replication Controller |
| selector:<br/> matchLabels: <br/>app: nginx <br/>| ReplicaSets, Deployments, Jobs, DaemonSet|

## Persistent Volume(PV) vs Psersistent Volume Claim(PVC)

### PV : Piece of Storage in Cluster

lifecycle : Provisioning -> Binding -> Using -> Reclaiming

Type : static, dynamic

static : PV needs to be created before PVC
dynamic : PV is created at same time of PVC

PVC : Request for storage