---
title: Game-Of-Pods-Tyro
categories: ["Kubernetes"]
tags: ["K8S", "Kubernetes", "kodekolud", "gameofpod", "Tyro"] 
toc: true
widgets:
  - type: toc
    position: right
sidebar:
  right:
    sticky: true
date: 2020-10-23 14:00:53
---

# KodeKloud-Game Of Pod
https://kodekloud.com/p/game-of-pods

## Solutions

## create namespace
```
kubectl create ns development
```

## admin
> Certificate and key pair for user drogo is created under /root. Add this user to kubeconfig = /root/.kube/config, User = drogo, client-key = /root/drogo.key client-certificate = /root/drogo.crt  
Create a new context in the default config file (/root/.kube/config) called 'developer' with user = drogo and cluster = kubernetes

```
kubectl config set-credentials drogo --client-key=/root/drogo.key --client-certificate=/root/drogo.crt
kubectl config set-context developer --cluster=kubernetes --user=drogo
```

## developer-role
> 'developer-role', should have all(*) permissions for services in development namespace  
'developer-role', should have all permissions(*) for persistentvolumeclaims in development namespace  
'developer-role', should have all(*) permissions for pods in development namespace

- yaml
  ```
  apiVersion: rbac.authorization.k8s.io/v1
  kind: Role
  metadata:
    name: developer-role
    namespace: development
  rules:
    - apiGroups:
        - ""
      resources:
        - svc
        - pvc 
        - pods
      verbs:
        - "*"
  ```

## developer-rolebinding
> create rolebinding = developer-rolebinding, role= 'developer-role', namespace = development  
rolebinding = developer-rolebinding associated with user = 'drogo'

- yaml 
  ```
  apiVersion: rbac.authorization.k8s.io/v1
  kind: RoleBinding
  metadata:
    name: developer-rolebinding
    namespace: development
  subjects:
    - kind: User
      name: drogo
      apiGroup: rbac.authorization.k8s.io
  roleRef:
    kind: Role
    name: developer-role
    apiGroup: rbac.authorization.k8s.io
  ```

## kube-config
> set context 'developer' with user = 'drogo' and cluster = 'kubernetes' as the current context.

```
kubectl config use-context developer
```

## jekyll-pvc
> Storage Request: 1Gi  
Access modes: ReadWriteMany  
pvc name = jekyll-site, namespace development

- yaml 
  ```
  apiVersion: v1
  kind: PersistentVolumeClaim
  metadata:
    name: jekyll-site
    namespace: development
  spec:
    accessModes:
      - ReadWriteMany
    resources:
      requests:
        storage: 1Gi
  ```

## jekyll
> pod: 'jekyll' has an initContainer, name: 'copy-jekyll-site', image: 'kodekloud/jekyll'
initContainer: 'copy-jekyll-site' command: [ "jekyll", "new", "/site" ] (command to run: jekyll new /site)  
pod: 'jekyll', initContainer: 'copy-jekyll-site', mountPath = /site  
pod: 'jekyll', initContainer: 'copy-jekyll-site', volume name = site  
pod: 'jekyll', container: 'jekyll', volume name = site  
pod: 'jekyll', container: 'jekyll', mountPath = /site  
pod: 'jekyll', container: 'jekyll', image = kodekloud/jekyll-serve  
pod: 'jekyll', uses volume called 'site' with pvc = 'jekyll-site'  
pod: 'jekyll' uses label 'run=jekyll'

- yaml
  ```
  apiVersion: v1
  kind: Pod
  metadata:
    namespace: development
    name: jekyll
    labels:
      run: jekyll
  spec: 
    initContainers:
    - name: copy-jekyll-site
      image: kodekloud/jekyll
      command: [ "jekyll", "new", "/site" ]
      volumeMounts:
      - mountPath: /site
        name: site
    containers:
    - name: jekyll
      image: kodekloud/jekyll-serve
      volumeMounts:
      - mountPath: /site
        name: site      
    volumes:
    - name: site
      persistentVolumeClaim:
        claimName: jekyll-site
  ```

## jekyll-node-service
> Service 'jekyll' uses targetPort: '4000' , namespace: 'development'  
Service 'jekyll' uses Port: '8080' , namespace: 'development'  
Service 'jekyll' uses NodePort: '30097' , namespace: 'development'

- yaml
  ```
  apiVersion: v1
  kind: Service
  metadata:
    name: jekyll
    namespace: development
    labels: 
      run: jekyll
  spec:
    ports:
      - protocol: TCP
        port: 8080
        targetPort: 4000
        nodePort: 30097
    selector:
      run: jekyll
    type: NodePort
  ```