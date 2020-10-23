---
title: Game-Of-Pods-VotingApp
categories: ["Kubernetes"]
tags: ["K8S", "Kubernetes", "kodekolud", "gameofpod", "VotingApp"] 
toc: true
widgets:
  - type: toc
    position: right
sidebar:
  right:
    sticky: true
date: 2020-10-23 16:51:58
---

# KodeKloud-Game Of Pod
https://kodekloud.com/p/game-of-pods

## Solutions

## create namespace
```
kubectl create ns vote
```

## vote-deployment
> Create a deployment: name = 'vote-deployment'  
image = 'kodekloud/examplevotingapp_vote:before'  
status: 'Running'

- yaml
  ```
  apiVersion: apps/v1
  kind: Deployment
  metadata:
    name: vote-deployment
    namespace: vote  
  spec:
    selector:
      matchLabels:
        app: vote-deployment
    template:
      metadata:
        labels:
          app: vote-deployment
      spec:
        containers:
          - name: vote-deployment
            image: kodekloud/examplevotingapp_vote:before
  ```

## vote-service
> Create a new service: name = vote-service  
port = '5000'  
targetPort = '80'  
nodePort= '31000'  
service endpoint exposes deployment 'vote-deployment'  

- yaml
  ```
  apiVersion: v1
  kind: Service
  metadata:
    name: vote-service
    namespace: vote
  spec:
    ports:
      - protocol: TCP
        port: 5000
        targetPort: 80
        nodePort: 31000
    type: NodePort
    selector:     
      app: vote-deployment
  ```

## redis-deployment
> Create new deployment, name: 'redis-deployment'  
image: 'redis:alpine'  
Volume Type: 'EmptyDir'  
Volume Name: 'redis-data'  
mountPath: '/data'  
status: 'Running'

- yaml
  ```
  apiVersion: apps/v1
  kind: Deployment
  metadata:
    name: redis-deployment
    namespace: vote
  spec:
    selector:
      matchLabels:
        app: redis-deployment
    template:
      metadata:
        labels:
          app: redis-deployment
      spec:
        containers:
          - name: redis-deployment
            image: redis:alpine
            volumeMounts:
              - name: redis-data
                mountPath: /data
        volumes:
          - name: redis-data
            emptyDir: {}
  ```

## redis
> New Service, name = 'redis'  
port: '6379'  
targetPort: '6379'  
type: 'ClusterIP'  
service endpoint exposes deployment 'redis-deployment'

- yaml
  ```
  apiVersion: v1
  kind: Service
  metadata:
    name: redis
    namespace: vote
  spec:
    ports:
      - protocol: TCP
        port: 6379
        targetPort: 6379
    selector:
      app: redis-deployment
  ```

## worker
> Create new deployment. name: 'worker'  
image: 'kodekloud/examplevotingapp_worker'  
status: 'Running'

- yaml
  ```
  apiVersion: apps/v1
  kind: Deployment
  metadata:
    name: worker
    namespace: vote  
  spec:
    selector:
      matchLabels:
        app: worker
    template:
      metadata:
        labels:
          app: worker
      spec:
        containers:
          - image: 'kodekloud/examplevotingapp_worker'
            name: worker
  ```

## db-deployment
> Create new deployment. name: 'db-deployment'  
image: 'postgres:9.4'  
Volume Type: 'EmptyDir'  
Volume Name: 'db-data'  
mountPath: '/var/lib/postgresql/data'  
status: 'Running'

- yaml
  ```
  apiVersion: apps/v1
  kind: Deployment
  metadata:
    name: db-deployment
    namespace: vote  
  spec:
    selector:
      matchLabels:
        app: db-deployment
    template:
      metadata:
        labels:
          app: db-deployment
      spec:
        containers:
          - image: postgres:9.4
            name: db-deployment
            volumeMounts:
              - name: db-data
                mountPath: /var/lib/postgresql/data
            ports:
              - containerPort: 5432
                name: db
                protocol: TCP
            env:
              - name: POSTGRES_PASSWORD
                value: test
        volumes:
          - name: db-data
            emptyDir: {}
  ```

## db
> Create new service: 'db'  
port: '5432'  
targetPort: '5432'  
type: 'ClusterIP'

- yaml
  ```
  apiVersion: v1
  kind: Service
  metadata:
    name: db
    namespace: vote
  spec:
    ports:
      - protocol: TCP
        port: 5432
        targetPort: 5432
    selector:
      app: db-deployment
  ```

## result-deployment
> Create new deployment, name: 'result-deployment'  
image: 'kodekloud/examplevotingapp_result:before'  
status: 'Running'  

- yaml
  ```
  apiVersion: apps/v1
  kind: Deployment
  metadata:
    name: result-deployment
    namespace: vote  
  spec:
    selector:
      matchLabels:
        app: result-deployment
    template:
      metadata:
        labels:
          app: result-deployment
      spec:
        containers:
          - image: kodekloud/examplevotingapp_result:before
            name: db-deployment
  ```

## result-service  
> port: '5001'  
targetPort: '80'  
NodePort: '31001'

- yaml
  ```
  apiVersion: v1
  kind: Service
  metadata:
    name: result-service
    namespace: vote
  spec:
    ports:
      - protocol: TCP
        port: 5001
        targetPort: 80
        nodePort: 31001
    type: NodePort      
    selector:
      app: result-deployment
  ```