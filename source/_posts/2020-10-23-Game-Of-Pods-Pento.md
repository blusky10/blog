---
title: Game-Of-Pods-Pento
categories: ["Kubernetes"]
tags: ["K8S", "Kubernetes", "kodekolud", "gameofpod", "Pento", "cordon", "uncordon"] 
toc: true
widgets:
  - type: toc
    position: right
sidebar:
  right:
    sticky: true
date: 2020-10-23 11:22:09
---

# KodeKloud-Game Of Pod
https://kodekloud.com/p/game-of-pods

## Solutions

- cordon
  
  지정된 노드에 더이상 Pod 들이 스케쥴링 되지 않도록 해준다. 

  cordon 을 실행하면 node 의 STATUS 에 SchedulingDisabled 가 표시된다.

- uncordon
  
  지정된 노드에 Pod 들이 스케쥴링 될수 있도록 해준다.

## master-node
> Master node: coredns deployment has image: 'k8s.gcr.io/coredns:1.3.1'  
Fix kube-apiserver. Make sure its running and healthy.  
kubeconfig = /root/.kube/config, User = 'kubernetes-admin' Cluster: Server Port = '6443'

- ca-file 경로 변경
  
  /etc/kubernetes/manifests/kube-apiserver.yaml

  kube-apiserver.yaml 파일의 --client-ca-file 을 변경해준다.

  변경전 : --client-ca-file=/etc/kubernetes/pki/ca-authority.crt

  변경후 : --client-ca-file=/etc/kubernetes/pki/ca.crt

- ~/.kube/config 변경

  cluster port 를 6443 으로 변경한다.

  변경전 : server: https://172.17.0.28:2379

  변경후 : server: https://172.17.0.28:6443

- coredns Deployment 의 이미지를 변경한다. 
  ```
  kubectl edit deployment coredns
  ```

## node01
> node01 is ready and can schedule pods?  

- uncordon 명령어 실행
  ```
  kubectl uncordon node01
  ```

## web
> node01 has hostPath created = '/web'  

  * connect node01 : ssh node01
  * mkdir /web

## data-pv
> Create new PersistentVolume = 'data-pv'  
PersistentVolume = data-pv, accessModes = 'ReadWriteMany'  
PersistentVolume = data-pv, hostPath = '/web'  
PersistentVolume = data-pv, storage = '1Gi'

- yaml
  ```
  apiVersion: v1
  kind: PersistentVolume
  metadata:
    name: data-pv
  spec:
    accessModes:
      - ReadWriteMany
    hostPath:
      path: /web
    capacity:
      storage: 1Gi
  ```

## data-pvc
> Create new PersistentVolumeClaim = 'data-pvc'  
PersistentVolume = 'data-pvc', accessModes = 'ReadWriteMany'  
PersistentVolume = 'data-pvc', storage request = '1Gi'  
PersistentVolume = 'data-pvc', volumeName = 'data-pv'

- yaml
  ```
  apiVersion: v1
  kind: PersistentVolumeClaim
  metadata:
    name: data-pvc
  spec:
    accessModes:
      - ReadWriteMany
    resources:
      requests:
        storage: 1Gi
    volumeName: data-pv
  ```

## gop-file-server
> Create a pod for fileserver, name: 'gop-fileserver'  
pod: gop-fileserver image: 'kodekloud/fileserver'  
pod: gop-fileserver mountPath: '/web'  
pod: gop-fileserver volumeMount name: 'data-store'  
pod: gop-fileserver persistent volume name: data-store  
pod: gop-fileserver persistent volume claim used: 'data-pvc'

- yaml
  ```
  apiVersion: v1
  kind: Pod
  metadata:
    labels:
      app: fs
    name: gop-fileserver
  spec:
    containers:
    - name: fs
      image: kodekloud/fileserver
      volumeMounts:
      - mountPath: /web
        name: data-store
    volumes:
    - name: data-store
      persistentVolumeClaim:
        claimName: data-pvc
  ```

## gop-fs-service
> New Service, name: 'gop-fs-service'  
Service name: gop-fs-service, port: '8080'  
Service name: gop-fs-service, targetPort: '8080'

- yaml
  ```
  apiVersion: v1
  kind: Service
  metadata:
    labels:
      app: fs
    name: gop-fs-service
  spec:
    ports:
      - protocol: TCP
        port: 8080
        targetPort: 8080
        nodePort: 31200
    selector:
      app: fs
    type: NodePort
  ```