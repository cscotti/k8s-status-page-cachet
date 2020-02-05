# cachet-kubernetes

A Kubernetes version of Cachet. https://cachethq.io/


## Introduction

Following the documentation below, I've migrated cachet-docker Docker Compose project to Kubernetes. 
https://www.digitalocean.com/community/tutorials/how-to-migrate-a-docker-compose-workflow-to-kubernetes

## Prerequisites
- A Kubernetes cluster (eg KIND / Kubernetes In Dockers)
- kubectl command-line tool to connect to cluster
- Docker and docker-compose 
- A Docker Hub account
- kompose

## Usage

### Step 1: Edit parameter

*postgres-volume0-persistentvolume.yaml*
```
storage: 1Gi
storageClassName: local-storage
path: /var/tmp
k8s-kubenode1
```

*postgres-claim0-persistentvolumeclaim.yaml*
```
storageClassName: local-storage
storage: 1Gi
```

*secret.yaml*
```
APP_KEY: xxxxxxxxxxxxxxxxxxxxxxxx
```
**Note**: I've used the default values on the cachet-docker project.

```
echo -n 'postgres' | base64
cG9zdGdyZXM=

echo -n 'pgsql' | base64
cGdzcWw=
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

$ kubectl get pods -n default
NAME                      READY   STATUS    RESTARTS   AGE
cachet-5c8649744-qw7ds    1/1     Running   0          2d3h
postgres-89fdc8fc-zhqfm   1/1     Running   0          2d3h

$ kubectl get svc -n default
NAME         TYPE           CLUSTER-IP       EXTERNAL-IP   PORT(S)        AGE
cachet       LoadBalancer   10.108.29.172    172.17.255.3     8000:30978/TCP   2d3h
kubernetes   ClusterIP      10.96.0.1        <none>         443/TCP          3h43m
postgres     LoadBalancer   10.101.54.34     172.17.255.4   5432:32761/TCP   33m
```

Done. Navigate to it in your browser (http://172.17.255.3:8000/).

## Troubleshooting

If you see the below problem;

```
  [RuntimeException] No supported encrypter found. The cipher and / or key length are invalid.
```

**Workaround; **

```bash
$ kubectl get pods -n default
NAME                      READY   STATUS    RESTARTS   AGE
cachet-5c8649744-qw7ds    1/1     Running   0          2d23h
postgres-89fdc8fc-zhqfm   1/1     Running   0          2d23h

$ kubectl exec -it cachet-5c8649744-qw7ds -- /bin/bash
bash-4.4$
bash-4.4$ php artisan key:generate
Application key [base64:YRfxkub3SLCgPSNFBcY6SyInA7kFY0KKJXdG2wgsFnQ=] set successfully.

bash-4.4$ APP_KEY=base64:YRfxkub3SLCgPSNFBcY6SyInA7kFY0KKJXdG2wgsFnQ=

bash-4.4$ php artisan config:clear
Configuration cache cleared!

bash-4.4$ php artisan config:cache
Configuration cache cleared!
Configuration cached successfully!

bash-4.4$ php artisan app:update
Clearing settings cache...
Settings cache cleared!
Backing up database...
Dumping database and uploading...

Successfully dumped pgsql, compressed with gzip and store it to local at /var/www/html/database/backups/2020-02-05 21.50.28
Backup completed!
Configuration cache cleared!
Configuration cached successfully!
Route cache cleared!
Routes cached successfully!
Copied Directory [/vendor/roumen/feed/src/views] To [/resources/views/vendor/feed]
Publishing complete for tag []!
Nothing to migrate.
Clearing cache...
Application cache cleared!
Cache cleared!

```

Change your file cachet-deployment.yaml in order to update the APP_KEY and apply the deployment like this

```bash
$ kubectl apply -f cachet-deployment.yaml

$ kubectl get svc -n default
NAME         TYPE           CLUSTER-IP       EXTERNAL-IP   PORT(S)        AGE
cachet       LoadBalancer   10.108.29.172    172.17.255.3     8000:30978/TCP   2d3h
kubernetes   ClusterIP      10.96.0.1        <none>         443/TCP          3h43m
postgres     LoadBalancer   10.101.54.34     172.17.255.4   5432:32761/TCP   33m
```

Done. Navigate to it in your browser again (http://172.17.255.3:8000/).