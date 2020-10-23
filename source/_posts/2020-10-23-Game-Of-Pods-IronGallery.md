---
title: Game-Of-Pods-IronGallery
categories: ["Kubernetes"]
tags: ["K8S", "Kubernetes", "kodekolud", "gameofpod", "IronGallery"] 
toc: true
widgets:
  - type: toc
    position: right
sidebar:
  right:
    sticky: true
date: 2020-10-23 16:52:16
---

# KodeKloud-Game Of Pod
https://kodekloud.com/p/game-of-pods

## Solutions

- 주의할점
  iron-gallery-service 생성시 nodePort 를 꼭 추가해야한다. iron-gallery-service 의 조건에는 없지만 iron-gallery-ingress 생성시 접속 주소에 nodePort 가 명시되어있다.

## iron-gallery
> New Deployment, name: 'iron-gallery'  
Image: 'kodekloud/irongallery:2.0'  
volume, name = config, type: emptyDir  
volume, name = images, type: emptyDir  
volumeMount, name: 'config', mountPath: '/usr/share/nginx/html/data'  
volumeMount, name: 'images', mountPath: '/usr/share/nginx/html/uploads'  
Replicas: 1  
Pod Label: 'run=iron-gallery'

- yaml
  ```
  apiVersion: apps/v1
  kind: Deployment
  metadata:
    name: iron-gallery
  spec:
    replicas: 1
    selector:
      matchLabels:
        run: iron-gallery
    template:    
      metadata:
        labels:
          run: iron-gallery
      spec:
        containers:
          - image: kodekloud/irongallery:2.0
            name: iron-gallery
            volumeMounts:
              - name: config
                mountPath: /usr/share/nginx/html/data
              - name: images
                mountPath: /usr/share/nginx/html/uploads
        volumes:
          - name: config
            emptyDir: {}
          - name: images
            emptyDir: {}
  ```

## iron-gallery-limits
> Deployment: 'iron-gallery', container has CPU limit: '50m'  
Deployment: 'iron-gallery', container has Memory limit: '100Mi'

```
kubectl set resources deploy iron-gallery --limits=cpu=50m,memory=100Mi
```
## netpol-iron-gallery
> NetworkPolicy, name: 'iron-gallery-firewall'  
Ingress Rule - from Pod labeled: 'run=iron-gallery'  
Applied to Pod with label: 'db=mariadb'  
Applied to allow access to port : '3306'

- yaml
  ```
  apiVersion: networking.k8s.io/v1
  kind: NetworkPolicy
  metadata:
    name: iron-gallery-firewall
  spec:
    podSelector:
      matchLabels:
        db: mariadb      
    ingress:
      - from:
        - podSelector:
            matchLabels:
              run: iron-gallery            
        ports:
          - protocol: TCP
            port: 3306
  ```

## iron-gallery-service
> Service: 'iron-gallery-service' has 'one' endpoint for pods in deployment 'iron-gallery'?  
targetPort: 80  
port: 80

- yaml
  ```
  apiVersion: v1
  kind: Service
  metadata: 
    name: iron-gallery-service
  spec:
    ports:
      - protocol: TCP
        port: 80
        targetPort: 80
        nodePort: 30099
    selector:
      run: iron-gallery
  ```

## gallery-of-braavos
> Ingress resource configured correctly and application accessible at 'http://iron-gallery-braavos.com:30099/'  
Ingress Resource, name: 'iron-gallery-ingress'  
host: iron-gallery-braavos.com  
http parth: '/'  
http backend serviceName: 'iron-gallery-service'  
Name: ingress-spacehttp backend servicePort: '80'

- yaml
  ```
  apiVersion: networking.k8s.io/v1beta1
  kind: Ingress
  metadata:
    name: iron-gallery-ingress
  spec:
    rules:
    - host: iron-gallery-braavos.com
      http:
        paths:
        - path: /
          backend:
            serviceName: iron-gallery-service
            servicePort: 80
  ```

## iron-db
> New Deployment, name: 'iron-db'  
Image: 'kodekloud/irondb:2.0'  
volume, name = db, type: emptyDir  
volumeMount, name: 'db', mountPath: '/var/lib/mysql'  
Replicas: 1  
env, name: 'MYSQL_ROOT_PASSWORD', value: 'Braavo'  
env, name: 'MYSQL_DATABASE', value: 'lychee'  
env, name: 'MYSQL_USER', value: 'lychee'  
env, name: 'MYSQL_PASSWORD', value: 'lychee'  
Pod Label: 'db=mariadb'

- yaml
  ```
  apiVersion: apps/v1
  kind: Deployment
  metadata:
    name: iron-db
  spec:
    replicas: 1
    selector:
      matchLabels:
        db: mariadb
    template:
      metadata:
        labels:
          db: mariadb
      spec:
        containers:
          - image: kodekloud/irondb:2.0
            name: mariadb
            volumeMounts:
              - name: db
                mountPath: /var/lib/mysql
            env:
              - name: MYSQL_ROOT_PASSWORD
                value: Braavo
              - name: MYSQL_DATABASE
                value: lychee
              - name: MYSQL_USER
                value: lychee
              - name: MYSQL_PASSWORD
                value: lychee
        volumes:
          - name: db
            emptyDir: {}
  ```

## iron-db-service
> Service: 'iron-db-service' has 'one' endpoint for pods in deployment 'iron-db'?  
targetPort: 3306  
Service Port: '3306'

- yaml
  ```
  apiVersion: v1
  kind: Service
  metadata: 
    name: iron-db-service
  spec:
    ports:
      - protocol: TCP
        port: 3306
        targetPort: 3306
    selector:
      db: mariadb
  ```