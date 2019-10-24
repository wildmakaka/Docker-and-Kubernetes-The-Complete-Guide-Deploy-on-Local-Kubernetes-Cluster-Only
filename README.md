# Deploy application from video course [Stephen Grider] Docker and Kubernetes: The Complete Guide [2018, ENG] on local kubernetes cluster

**Full version**:  
https://github.com/marley-nodejs/Docker-and-Kubernetes-The-Complete-Guide

<br/>

IN { ALPHA || BETA }

<br/>

### 14 A Multi-Container App with Kubernetes

<br/>

![Application](/img/pic-14-01.png?raw=true)

<br/>

![Application](/img/pic-14-09.png?raw=true)


<br/>

Prepare cluster and environment as <a href="https://sysadm.ru/linux/servers/containers/kubernetes/kubeadm/prepared-cluster/">here</a>.

Prepare Dynamic NFS as <a href="https://sysadm.ru/linux/servers/containers/kubernetes/kubeadm/persistence/dynamic-nfs-provisioning/">here</a>.


<br/>

    $ kubectl get nodes
    NAME         STATUS   ROLES    AGE     VERSION
    master.k8s   Ready    master   11m     v1.16.2
    node1.k8s    Ready    <none>   8m29s   v1.16.2
    node2.k8s    Ready    <none>   5m41s   v1.16.2


<br/>

### Download codes from github

    $ cd ~/tmp
    $ git clone https://github.com/marley-nodejs/Docker-and-Kubernetes-The-Complete-Guide-Deploy-on-Local-Kubernetes-Cluster-Only

    $ cd Docker-and-Kubernetes-The-Complete-Guide-Local-Kubernetes-Only/kubernetes

<br/>


### Client (React App)

    $ kubectl apply -f client-deployment.yaml
    $ kubectl apply -f client-cluster-ip-service.yaml

<br/>

    $ kubectl get deployments
    NAME                READY   UP-TO-DATE   AVAILABLE   AGE
    client-deployment   3/3     3            3           22s

<br/>

    $ kubectl get pods
    NAME                                 READY   STATUS    RESTARTS   AGE
    client-deployment-64d5846fbd-2bms4   1/1     Running   0          46s
    client-deployment-64d5846fbd-v7s5b   1/1     Running   0          46s
    client-deployment-64d5846fbd-z59px   1/1     Running   0          46s


<br/>

    $ kubectl get services
    NAME                        TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
    client-cluster-ip-service   ClusterIP   10.102.33.117   <none>        3000/TCP   65s
    kubernetes                  ClusterIP   10.96.0.1       <none>        443/TCP    33m

<br/>

### Server (Express App)

<br/>

    $ kubectl create secret generic pgpassword --from-literal PGPASSWORD=12345asdf
    $ kubectl get secret

<br/>

    $ kubectl apply -f server-deployment.yaml
    $ kubectl apply -f server-cluster-ip-service.yaml

<br/>

### Worker (Node.js App)

    $ kubectl apply -f worker-deploymant.yaml

<br/>

### Redis

<br/>

    $ kubectl apply -f redis-deployment.yaml
    $ kubectl apply -f redis-cluster-ip-service.yaml

<br/>

    $ kubectl get storageclass
    NAME                            PROVISIONER       AGE
    managed-nfs-storage (default)   example.com/nfs   3m52s

<br/>

    $ kubectl apply -f database-persistent-volume-claim.yaml


<br/>

### Postgres

<br/>

    $ kubectl apply -f postgres-deployment.yaml
    $ kubectl apply -f postgres-cluster-ip-service.yaml


<br/>

    $ kubectl get pv
    NAME                                       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                                      STORAGECLASS          REASON   AGE
    pvc-36c424a0-a1cb-42ec-81ac-cf2327629af4   2Gi        RWO            Delete           Bound    default/database-persistent-volume-claim   managed-nfs-storage            61s


<br/>

    $ kubectl get pvc
    NAME                               STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS          AGE
    database-persistent-volume-claim   Bound    pvc-36c424a0-a1cb-42ec-81ac-cf2327629af4   2Gi        RWO            managed-nfs-storage   77s

<br/>

    $ kubectl get pods
    NAME                                     READY   STATUS    RESTARTS   AGE
    client-deployment-64d5846fbd-2bms4       1/1     Running   0          60m
    client-deployment-64d5846fbd-v7s5b       1/1     Running   0          60m
    client-deployment-64d5846fbd-z59px       1/1     Running   0          60m
    nfs-client-provisioner-b48654857-flmqb   1/1     Running   0          18m
    postgres-deployment-789f77969f-lvtzh     1/1     Running   0          13m
    redis-deployment-5f458546b8-kzwwh        1/1     Running   0          42m
    server-deployment-5d5c98df75-7h4q2       1/1     Running   0          51m
    server-deployment-5d5c98df75-bk5bp       1/1     Running   0          51m
    server-deployment-5d5c98df75-r4tsm       1/1     Running   0          51m
    wroker-deployment-58f6c66f6f-p9j8f       1/1     Running   0          51m

<br/>

### 15 Handling Traffic with Ingress Controllers


<br/>

**Here we use:**  
nginxinc-kubernetes-ingress

<br/>

### Install and setup HAProxy as <a href="https://sysadm.ru/linux/servers/containers/kubernetes/kubeadm/ingress/haproxy/">here</a>.

### Create NginxInc Kubernetes Ingress Controller as <a href="https://sysadm.ru/linux/servers/containers/kubernetes/kubeadm/ingress/nginxinc-kubernets-ingress-install/">here</a>.

<br/>

### Run our ingress config on local kubernetes cluster

<br/>

```
$ cat <<EOF | kubectl apply -f -
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: nginxinc-kubernetes-ingress
  annotations:
    nginx.org/rewrites: "serviceName=server-cluster-ip-service rewrite=/"
spec:
  rules:
  - host: grider-app.com
    http:
      paths:
      - path: /
        backend:
          serviceName: client-cluster-ip-service
          servicePort: 3000
      - path: /api/
        backend:
          serviceName: server-cluster-ip-service
          servicePort: 5000
EOF

```

<br/>

    $ curl -I http://192.168.0.5 -H 'Host:grider-app.com'
    HTTP/1.1 200 OK
    Server: nginx/1.17.4
    Date: Thu, 24 Oct 2019 01:24:38 GMT
    Content-Type: text/html
    Content-Length: 548
    Last-Modified: Wed, 23 Oct 2019 03:13:41 GMT
    ETag: "5dafc565-224"
    Accept-Ranges: bytes

<br/>

### On local host

    # vi /etc/hosts
    192.168.0.5 grider-app.com

<br/>

**Checks:**

    http://grider-app.com


<br/>

![Application](/img/result-01.png?raw=true)

<br/>

![Application](/img/result-02.png?raw=true)

<br/>

    http://grider-app.com/api/

<br/>

![Application](/img/result-03.png?raw=true)

<br/>

    http://grider-app.com/api/values/current

<br/>

![Application](/img/result-04.png?raw=true)

---

**Marley**

<a href="https://jsdev.org">jsdev.org</a>

Any questions on eng: https://t.me/jsdev_org  
Любые вопросы на русском: https://t.me/jsdev_ru

