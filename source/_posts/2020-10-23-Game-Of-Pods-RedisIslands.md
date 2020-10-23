---
title: Game-Of-Pods-RedisIslands
categories: ["Kubernetes"]
tags: ["K8S", "Kubernetes", "kodekolud", "gameofpod", "Pento", "RedisIslands"] 
toc: true
widgets:
  - type: toc
    position: right
sidebar:
  right:
    sticky: true
date: 2020-10-23 13:46:20
---

# KodeKloud-Game Of Pod
https://kodekloud.com/p/game-of-pods

## Solutions

Statefulset 에 대한 내용을 알고 있어야 한다.

## redis01~06
> PersistentVolume - Name: redis01  
Access modes: ReadWriteOnce  
Size: 1Gi  
hostPath: /redis01, directory should be created on worker node  

- ssh node01
- make /redis01~06
- yaml (숫자만 01에서 06까지 바꿔주면 된다)
  ```
  apiVersion: v1
  kind: PersistentVolume
  metadata:
    name: redis01
  spec:
    accessModes:
      - ReadWriteOnce
    hostPath:
      path: /redis01
    capacity:
      storage: 1Gi
  ```

## redis-cluster
> StatefulSet - Name: redis-cluster  
Replicas: 6  
Pods status: Running (All 6 replicas)  
Image: redis:5.0.1-alpine  
container name: redis, command: ["/conf/update-node.sh", "redis-server", "/conf/redis.conf"]  
Env: name: 'POD_IP', valueFrom: 'fieldRef', fieldPath: 'status.podIP' (apiVersion: v1)  
Ports - name: 'client', containerPort: '6379'  
Ports - name: 'gossip', containerPort: '16379'  
Volume Mount - name: 'conf', mountPath: '/conf', readOnly:'false' (ConfigMap Mount)  
Volume Mount - name: 'conf', mountPath: '/conf', defaultMode = '0755' (ConfigMap Mount)  
Volume Mount - name: 'data', mountPath: '/data', readOnly:'false' (volumeClaim)  
volumes - name: 'conf', Type: 'ConfigMap', ConfigMap Name: 'redis-cluster-configmap',
volumeClaimTemplates - name: 'data'  
volumeClaimTemplates - accessModes: 'ReadWriteOnce'  
volumeClaimTemplates - Storage Request: '1Gi'

- yaml
  ```
  apiVersion: apps/v1
  kind: StatefulSet
  metadata:
    name: redis-cluster
  spec:
    selector:
      matchLabels:
        app: redis-cluster
    serviceName: redis-cluster
    replicas: 6
    template:
      metadata:
        labels:
          app: redis-cluster
      spec:
        containers:
        - name: redis
          image: redis:5.0.1-alpine
          command: ["/conf/update-node.sh", "redis-server", "/conf/redis.conf"]
          env:
          - name: POD_IP
            valueFrom:
              fieldRef:
                fieldPath: status.podIP
          ports:
          - name: client
            containerPort: 6379
          - name: gossip
            containerPort: 16379
          volumeMounts:
          - name: conf
            mountPath: /conf
            readOnly: false
          - name: data
            mountPath: /data            
            readOnly: false
        volumes:
        - name: conf
          configMap:
            name: redis-cluster-configmap
            defaultMode: 0755          
    volumeClaimTemplates:
    - metadata:
        name: data
      spec:
        accessModes:
        - ReadWriteOnce
        resources:
          requests:
            storage: 1Gi   
  ```

## redis-cluster-config
> Configure the Cluster. Once the StatefulSet has been deployed with 6 'Running' pods, run the below commands and type 'yes' when prompted.  
Command: kubectl exec -it redis-cluster-0 -- redis-cli --cluster create --cluster-replicas 1 $(kubectl get pods -l app=redis-cluster -o jsonpath='{range.items[*]}{.status.podIP}:6379 ')

- 아래 명령어 실행
```
kubectl exec -it redis-cluster-0 -- redis-cli --cluster create --cluster-replicas 1 $(kubectl get pods -l app=redis-cluster -o jsonpath='{range.items[*]}{.status.podIP}:6379 ')
```

## redis-cluster-service
> Ports - service name 'redis-cluster-service', port name: 'client', port: '6379'  
Ports - service name 'redis-cluster-service', port name: 'gossip', port: '16379'  
Ports - service name 'redis-cluster-service', port name: 'client', targetPort: '6379'  
Ports - service name 'redis-cluster-service', port name: 'gossip', targetPort: '16379'

- yaml
  ```
  apiVersion: v1
  kind: Service
  metadata:
    name: redis-cluster-service
  spec:
    ports:
      - protocol: TCP
        name: client
        port: 6379
        targetPort: 6379
      - protocol: TCP
        name: gossip
        port: 16379
        targetPort: 16379
  ```