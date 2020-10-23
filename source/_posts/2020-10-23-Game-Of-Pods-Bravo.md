---
title: Game-Of-Pods-Bravo
categories: ["Kubernetes"]
tags: ["K8S", "Kubernetes", "kodekolud", "gameofpod", "Bravo"] 
toc: true
widgets:
  - type: toc
    position: right
sidebar:
  right:
    sticky: true
date: 2020-10-23 09:27:58
---

# Solutions

## drpal-pv-hostpath, drupal-mysql-pv-hostpath

> /drupal-mysql-data (create the directory on Worker Nodes)  
/drupal-data (create the directory on Worker Nodes)

  * connect node01 : ssh node01
  * mkdir /drupal-mysql-data
  * mkdir /drupal-data

## drupal-mysql-pv, drupal-mysql-pvc
> Volume Name: drupal-mysql-pv  
Storage: 5Gi  
Access modes: ReadWriteOnce  
- yaml
  ```
  apiVersion: v1
  kind: PersistentVolume
  metadata:
  name: drupal-mysql-pv
  spec:
  accessModes:
    - ReadWriteOnce
  capacity:
    storage: 5Gi
  hostPath:
    path: /drupal-mysql-data
  ```
## drupal-mysql-pvc      
> Claim Name: drupal-mysql-pvc  
Storage Request: 5Gi  
Access modes: ReadWriteOnce
- yaml
  ```
  apiVersion: v1
  kind: PersistentVolumeClaim
  metadata:
    name: drupal-mysql-pvc
  spec:
    accessModes:
      - ReadWriteOnce
    resources:
      requests:
        storage: 5Gi
## drupal-mysql-secret

> Secret Name: drupal-mysql-secret  
Secret: MYSQL_ROOT_PASSWORD=root_password  
Secret: MYSQL_DATABASE=drupal-database  
Secret: MYSQL_USER=root  

- 값 인코딩
  ```
  echo -n "root_password" | base64
  echo -n "drupal-database" | base64
  echo -n "root" | base64
  ```
- yaml
  ```
  apiVersion: v1
  kind: Secret
  metadata:
    name: drupal-mysql-secret
  type: Opaque
  data:
    MYSQL_ROOT_PASSWORD: cm9vdF9wYXNzd29yZA==
    MYSQL_DATABASE: ZHJ1cGFsLWRhdGFiYXNl
    MYSQL_USER: cm9vdA==
  ```
- command line 으로 입력 하는 방법
  ```
  kubectl create secret generic drupal-mysql-secret --from-literal=MYSQL_ROOT_PASSWORD=root_password --from-literal=MYSQL_DATABASE=drupal-database --from-literal=MYSQL_USER=root  
  ```
## drupal-mysql
> Name: drupal-mysql  
Replicas: 1  
Image: mysql:5.7  
Deployment Volume uses PVC : drupal-mysql-pvc  
Volume Mount Path: /var/lib/mysql, subPath: dbdata  
Deployment: 'drupal-mysql' running  
- yaml
  ```
  apiVersion: apps/v1
  kind: Deployment
  metadata:
    name: drupal-mysql
  spec:
    replicas: 1
    selector:
      matchLabels:
        app: drupal-mysql
    template:
      metadata:
        labels:
          app: drupal-mysql
      spec:
        containers:
        - name: drupal-mysql
          image: mysql:5.7
          env:
          - name: MYSQL_ROOT_PASSWORD
            valueFrom:
              secretKeyRef:
                name: drupal-mysql-secret
                key: MYSQL_ROOT_PASSWORD
          - name: MYSQL_DATABASE
            valueFrom:
              secretKeyRef:
                name: drupal-mysql-secret
                key: MYSQL_DATABASE
          - name: MYSQL_USER   
            valueFrom:
              secretKeyRef:
                name: drupal-mysql-secret
                key: MYSQL_USER    
          ports:
          - containerPort: 3306
            name: mysql
            protocol: TCP    
          volumeMounts: 
          - mountPath: "/var/lib/mysql"
            name: mysql-volume
            subPath: dbdata
  ```
## drupal-mysql-service
> Name: drupal-mysql-service  
Type: ClusterIP  
Port: 3306  
- yaml
  ```
  apiVersion: v1
  kind: Service
  metadata:
    name: drupal-mysql-service
  spec:
    ports:
      - protocol: TCP
        port: 3306
        targetPort: 3306
    selector:
      app: drupal-mysql   
  ```
## drupal-pv
> Access modes: ReadWriteOnce  
Volume Name: drupal-pv  
Storage: 5Gi
- yaml
  ```
  apiVersion: v1
  kind: PersistentVolume
  metadata:
  name: drupal-pv
  spec:
  accessModes:
    - ReadWriteOnce
  capacity:
    storage: 5Gi
  hostPath:
    path: /drupal-data
  ```
## drupal-pvc
> Claim Name: drupal-pvc  
Storage Request: 5Gi  
Access modes: ReadWriteOnce

- yaml
  ```
  apiVersion: v1
  kind: PersistentVolumeClaim
  metadata:
    name: drupal-pvc
  spec:
    accessModes:
      - ReadWriteOnce
    resources:
      requests:
        storage: 5Gi      
  ```

### drupal

> Deployment Name: drupal  
Replicas: 1  
Image: drupal:8.6  
Deployment has an initContainer, name: 'init-sites-volume'  
initContainer 'init-sites-volume', image: drupal:8.6  
initContainer 'init-sites-volume', persistentVolumeClaim: drupal-pvc  
initContainer 'init-sites-volume', mountPath: /data  
initContainer 'init-sites-volume', Command: [ "/bin/bash", "-c" ],  
initContainer: Args: [ 'cp -r /var/www/html/sites/ /data/; chown www-data:www-data /data/ -R' ]
Deployment 'drupal' uses correct pvc: drupal-pvc  
Deployment has a regular container, name: 'drupal', image: 'drupal:8.6'  
container: 'drupal', Volume mountPath: /var/www/html/modules, subPath: modules  
container: 'drupal', Volume mountPath: /var/www/html/profiles, subPath: profiles  
container: 'drupal', Volume mountPath: /var/www/html/sites, subPath: sites  
container: 'drupal', Volume mountPath: /var/www/html/themes, subPath: themes  
Deployment: "drupal" running  
Deployment: 'drupal' has label 'app=drupal'  

- yaml
  ```
  apiVersion: apps/v1
    kind: Deployment
  metadata:
    name: drupal
    labels:
      app: drupal  
  spec:
    replicas: 1
    selector:
      matchLabels:
        app: drupal  
    template:
      metadata:
        labels:
          app: drupal
      spec:
        initContainers:
        - name: init-sites-volume
          image: drupal:8.6
          command: [ "/bin/bash", "-c" ]
          args: [ 'cp -r /var/www/html/sites/ /data/; chown www-data:www-data /data/ -R' ]
          volumeMounts:
          - mountPath: "/data"
            name: drupal-volume         
        containers:
        - name: drupal
          image: drupal:8.6
          ports:
          - containerPort: 80
          volumeMounts:
          - mountPath: "/var/www/html/modules"
            name: drupal-volume
            subPath: modules
          - mountPath: "/var/www/html/profiles"
            name: drupal-volume
            subPath: profiles
          - mountPath: "/var/www/html/sites"
            name: drupal-volume
            subPath: sites
          - mountPath: "/var/www/html/themes"
            name: drupal-volume
            subPath: themes   
        volumes:
          - name: drupal-volume
            persistentVolumeClaim:
              claimName: drupal-pvc
  ```
## drupal-service
> frontend service name: drupal-service  
drupal-service configured as NodePort  
drupal-service uses NodePort 30095
- yaml
  ```
  apiVersion: v1
  kind: Service
  metadata:
    name: drupal-service
  spec:
    type: NodePort
    ports:
      - protocol: TCP
        port: 80
        nodePort: 30095
    selector:
      app: drupal 
  ```