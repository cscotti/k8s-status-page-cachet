# cachet-kubernetes

A Kubernetes version of Cachet. https://cachethq.io/


## Introduction

Following the documentation below, I've migrated cachet-docker Docker Compose project to Kubernetes. 
https://www.digitalocean.com/community/tutorials/how-to-migrate-a-docker-compose-workflow-to-kubernetes

## Prerequisites
- A Kubernetes cluster
- kubectl command-line tool to connect to cluster
- Docker and docker-compose 
- A Docker Hub account
- kompose

## Usage

### Step 1: Edit parameters

*postgres-volume0-persistentvolume.yaml*
```
storage: 1Gi
storageClassName: local-storage
k8s-kubenode1
```

*postgres-claim0-persistentvolumeclaim.yaml*
```
storageClassName: local-storage
storage: 1Gi
```

*secret.yaml*
```
APP_KEY:
DB_DRIVER:
DB_HOST:
DB_DATABASE:
DB_USERNAME:
DB_PASSWORD:
POSTGRES_USER:
POSTGRES_PASSWORD:
```

**Note**: I've used the default values on the cachet-docker project.

```
echo -n 'postgres' | base64
cG9zdGdyZXM=
```

*cachet-deployment.yaml*
```
image: vahittabak/cachet-kubernetes
```

### Step 2: Create a storage class

I've used local volume plugin of Kubernetes, 
https://kubernetes.io/docs/concepts/storage/storage-classes/#local

```bash
$ kubectl -f storage-class.yaml
$ kubectl get storageclass
```

### Step 3: Create persistent volume and claim for DB

Create the objects;

```bash
$ kubectl -f postgres-volume0-persistentvolume.yaml
$ kubectl -f postgres-claim0-persistentvolumeclaim.yaml

$ kubectl get persistentvolume
$ kubectl get persistentvolumeclaim
```

### Step 4: Start the application

```bash
$ kubectl create -f cachet-deployment.yaml,cachet-service.yaml,postgres-deployment.yaml,postgres-service.yaml,secret.yaml

$ kubectl get pods
NAME                      READY   STATUS    RESTARTS   AGE
cachet-5c8649744-qw7ds    1/1     Running   0          2d3h
postgres-89fdc8fc-zhqfm   1/1     Running   0          2d3h

$ kubectl get svc
NAME         TYPE           CLUSTER-IP       EXTERNAL-IP   PORT(S)        AGE
cachet       LoadBalancer   10.108.29.172    <pending>     80:31080/TCP   2d3h
kubernetes   ClusterIP      10.96.0.1        <none>        443/TCP        3d
postgres     ClusterIP      10.108.151.113   <none>        5432/TCP       2d3h
```

Done. Navigate to it in your browser.