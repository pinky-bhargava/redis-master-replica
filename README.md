# Redis Master/Replica setup in Kubernetes

Follow below steps to create Redis Master/Replica setup in kubernetes

### Create a namespace 

```
[pinkybhargava@192 - Exit: 0] master (minikube) %
$ kubectl create namespace redis        
namespace/redis created
```
### Steps for creating Redis Master setup in Kubernetes 
* Goto master dir
* #### Create redis master config

```
[pinkybhargava@192 - Exit: 0] master (minikube) %
$ kubectl apply -f redis-config-map.yaml
configmap/redis-master created


[pinkybhargava@192 - Exit: 0] master (minikube) %
$ kubectl get configmaps -n redis
NAME               DATA   AGE
kube-root-ca.crt   1      16s
redis-master       1      11s

```

* #### Create Redis master pod

```
[pinkybhargava@192 - Exit: 0] master (minikube) %
$ kubectl apply -f deployment.yaml 
deployment.apps/redis-master created

[pinkybhargava@192 - Exit: 0] master (minikube) %
$ kubectl get pods -n redis       
NAME                            READY   STATUS    RESTARTS   AGE
redis-master-7bf4486dfd-rc9bs   1/1     Running   0          10s

```
* #### Create Redis master service

```
[pinkybhargava@192 - Exit: 0] master (minikube) %
$ kubectl apply -f redis-service.yaml 
service/redis-master created

[pinkybhargava@192 - Exit: 0] master (minikube) %
$ kubectl get svc -n redis       
NAME           TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)              AGE
redis-master   ClusterIP   10.109.90.135   <none>        6379/TCP,16379/TCP   6s

```

* #### Check Redis master pod logs 

```
[pinkybhargava@192 - Exit: 0] master (minikube) %
$ kubectl logs redis-master-7bf4486dfd-rc9bs -n redis
1:C 31 Oct 2022 04:20:07.431 # oO0OoO0OoO0Oo Redis is starting oO0OoO0OoO0Oo
1:C 31 Oct 2022 04:20:07.431 # Redis version=7.0.5, bits=64, commit=00000000, modified=0, pid=1, just started
1:C 31 Oct 2022 04:20:07.431 # Configuration loaded
1:M 31 Oct 2022 04:20:07.431 * monotonic clock: POSIX clock_gettime
1:M 31 Oct 2022 04:20:07.432 * Running mode=standalone, port=6379.
1:M 31 Oct 2022 04:20:07.432 # Server initialized
1:M 31 Oct 2022 04:20:07.433 * Ready to accept connections

```

### Steps for creating Redis Replica setup in Kubernetes
* Goto replica dir
* #### Create redis replica config

```
[pinkybhargava@192 - Exit: 0] redis-master-replica (minikube) %
$ cd replica 

[pinkybhargava@192 - Exit: 0] replica (minikube) %
$ ls
deployment.yaml		redis-config-map.yaml	redis-service.yaml

[pinkybhargava@192 - Exit: 0] replica (minikube) %
$ kubectl apply -f redis-config-map.yaml 
configmap/redis-replica created

[pinkybhargava@192 - Exit: 0] replica (minikube) %
$ kubectl get configmaps -n redis                
NAME               DATA   AGE
kube-root-ca.crt   1      2m10s
redis-master       1      2m5s
redis-replica      1      11s

```

* #### Create Redis replica pod

```
[pinkybhargava@192 - Exit: 0] replica (minikube) %
$ kubectl apply -f deployment.yaml 
deployment.apps/redis-replica created

[pinkybhargava@192 - Exit: 0] replica (minikube) %
$ kubectl get pods -n redis
NAME                             READY   STATUS    RESTARTS   AGE
redis-master-7bf4486dfd-rc9bs    1/1     Running   0          2m2s
redis-replica-6d6c8b5ff4-2wkwt   1/1     Running   0          9s
redis-replica-6d6c8b5ff4-fp6wl   1/1     Running   0          9s
redis-replica-6d6c8b5ff4-mzmh6   1/1     Running   0          9s

```
* #### Create Redis replica service

```
[pinkybhargava@192 - Exit: 0] replica (minikube) %
$ kubectl apply -f redis-service.yaml 
service/redis-replica created

[pinkybhargava@192 - Exit: 0] replica (minikube) %
$ kubectl get svc -n redis       
NAME            TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)              AGE
redis-master    ClusterIP   10.109.90.135   <none>        6379/TCP,16379/TCP   2m2s
redis-replica   ClusterIP   10.101.173.5    <none>        7000/TCP,17000/TCP   9s

```
* #### Create Redis replica pod logs

```
[pinkybhargava@192 - Exit: 0] replica (minikube) %
$ kubectl logs redis-replica-6d6c8b5ff4-mzmh6 -n redis
1:C 31 Oct 2022 04:22:00.240 # oO0OoO0OoO0Oo Redis is starting oO0OoO0OoO0Oo
1:C 31 Oct 2022 04:22:00.240 # Redis version=7.0.5, bits=64, commit=00000000, modified=0, pid=1, just started
1:C 31 Oct 2022 04:22:00.240 # Configuration loaded
1:S 31 Oct 2022 04:22:00.241 * monotonic clock: POSIX clock_gettime
1:S 31 Oct 2022 04:22:00.241 * Running mode=standalone, port=6379.
1:S 31 Oct 2022 04:22:00.241 # Server initialized
1:S 31 Oct 2022 04:22:00.242 * Ready to accept connections
1:S 31 Oct 2022 04:22:00.242 * Connecting to MASTER redis-master.redis.svc.cluster.local:6379
1:S 31 Oct 2022 04:22:00.244 * MASTER <-> REPLICA sync started
1:S 31 Oct 2022 04:22:00.244 * Non blocking connect for SYNC fired the event.
1:S 31 Oct 2022 04:22:00.244 * Master replied to PING, replication can continue...
1:S 31 Oct 2022 04:22:00.245 * Partial resynchronization not possible (no cached master)
1:S 31 Oct 2022 04:22:05.395 * Full resync from master: 72cec6e7308a188446daacb368e9587e4961af4e:0
1:S 31 Oct 2022 04:22:05.396 * MASTER <-> REPLICA sync: receiving streamed RDB from master with EOF to disk
1:S 31 Oct 2022 04:22:05.397 * MASTER <-> REPLICA sync: Flushing old data
1:S 31 Oct 2022 04:22:05.398 * MASTER <-> REPLICA sync: Loading DB in memory
1:S 31 Oct 2022 04:22:05.401 * Loading RDB produced by version 7.0.5
1:S 31 Oct 2022 04:22:05.401 * RDB age 0 seconds
1:S 31 Oct 2022 04:22:05.401 * RDB memory usage when created 0.93 Mb
1:S 31 Oct 2022 04:22:05.401 * Done loading RDB, keys loaded: 0, keys expired: 0.
1:S 31 Oct 2022 04:22:05.401 * MASTER <-> REPLICA sync: Finished with success

```

### Verify Replication from redis-master to replica

* #### Create a key/value in Redis master pod

```
[pinkybhargava@192 - Exit: 126] replica (minikube) %
$ kubectl -n redis -it exec redis-master-7bf4486dfd-rc9bs -c redis -- redis-cli
127.0.0.1:6379> AUTH Securep@55Here
OK
127.0.0.1:6379> get foo
(nil)
127.0.0.1:6379> set foo bar
OK
127.0.0.1:6379> get foo
"bar"
127.0.0.1:6379> exit
```

* #### Check if the key create in above step got replicated to the replica pod

```
[pinkybhargava@192 - Exit: 0] replica (minikube) %
$ kubectl get pods -n redis                           
NAME                             READY   STATUS    RESTARTS   AGE
redis-master-7bf4486dfd-rc9bs    1/1     Running   0          5m51s
redis-replica-6d6c8b5ff4-2wkwt   1/1     Running   0          3m58s
redis-replica-6d6c8b5ff4-fp6wl   1/1     Running   0          3m58s
redis-replica-6d6c8b5ff4-mzmh6   1/1     Running   0          3m58s

[pinkybhargava@192 - Exit: 0] replica (minikube) %
$ kubectl -n redis -it exec redis-replica-6d6c8b5ff4-2wkwt -c redis -- redis-cli
127.0.0.1:6379> get foo
"bar"
127.0.0.1:6379> exit

```
