# Side car for Kubenetes

## Introduce

Khái niệm side-car, kubernetes,... tham vấn anh TiếnDK và anh BéoCNN (ThanhCNN)

## Tutorial

### Tạo TLS Cert:

Giả sử chúng ta có Cluster là MEP Cluster SANDBOX, generate TLS Cert như sau:

    $ cd ./tls/
    $ DEPLOYMENT=mep-cluster CLUSTER=SANDBOX ./new-cluster-injector-cert.rb
    $ git ls-files -o
    .srl
    us-east-1/PRODUCTION/ca.crt
    us-east-1/PRODUCTION/ca.key
    us-east-1/PRODUCTION/sidecar-injector.crt
    us-east-1/PRODUCTION/sidecar-injector.csr
    us-east-1/PRODUCTION/sidecar-injector.key

### Mutating Webhook Configuaration

Sau đó chúng ta tạo ra 1 CA Cert dưới dạng mã hoá BASE64 như sau:

    > $ CABUNDLE_BASE64="$(cat ./tls/mep-cluster/SANDBOX/ca.crt |base64|tr -d '\n')"
    > $ echo $CABUNDLE_BASE64
    > LS0tLS........=

Done~! Chúng ta có 1 Key CA.

Thay key vừa tạo ra vào vị trí __CA_BUNDLE_BASE64__ trong file ./kubernetes/mutating-webhook-configuration.yaml

### Deployment

Tạo ra 1 secret key với câu lệnh sau:

    $ kubectl create secret generic devops-sidecar-injector --from-file=./tls/mep-cluster/SANDBOX/sidecar-injector.crt --from-file=./tls/mep-cluster/SANDBOX/sidecar-injector.key --namespace=devops

Sau đó deploy tất cả các file .yaml trong folder kubernetes, file debug-pod.yaml deploy sau cùng, sau khi tất cả các resource kia đã **ready**

Chú ý logs của devops-sidecar-injector:


    $ kubectl logs --tail=60 -n kube-system -l k8s-app=k8s-sidecar-injector
    ...
    I1119 16:25:10.782478       1 main.go:124] triggering ConfigMap reconciliation
    I1119 16:25:10.782536       1 watcher.go:140] Fetching ConfigMaps...
    I1119 16:25:10.792451       1 watcher.go:147] Fetched 1 ConfigMaps
    I1119 16:25:10.792757       1 watcher.go:168] Loaded InjectionConfig test1 from ConfigMap sidecar-test:test1
    I1119 16:25:10.792778       1 watcher.go:153] Found 1 InjectionConfigs in sidecar-test
    I1119 16:25:10.792788       1 main.go:130] got 1 updated InjectionConfigs from reconciliation
    I1119 16:25:10.792800       1 main.go:144] updating server with newly loaded configurations (5 loaded from disk, 1 loaded from k8s api)
    I1119 16:25:10.792813       1 main.go:146] configuration replaced
    ...

Logs ra như trên là ConfigMap sidecar đã hoạt động

Deploy file debug-pod.yaml

     > $ kubectl apply -f debug-pod.yaml

XONG~! Xem desciption của *POD* vừa tạo xem thử **Sidecar** đã hoạt động chưa:

    $ kubectl describe -f debug-pod.yaml
    ***
    Name:               nginx-debug
    Namespace:          devops
    Priority:           0
    PriorityClassName:  <none>
    Node:               lab-kubernetes-node-54/10.205.21.54
    Start Time:         Thu, 03 Oct 2019 16:42:48 +0700
    Labels:             <none>
    Annotations:        injector.tumblr.com/request=sidecar-debian
                        injector.tumblr.com/status=injected
                        kubectl.kubernetes.io/last-applied-configuration={"apiVersion":"v1","kind":"Pod","metadata":{"annotations":{"injector.tumblr.com/request":"sidecar-debian"},"name":"nginx-debug","namespace":"devops"},"...
    Status:             Pending
    IP:
    Containers:
    nginx-debug:
        Container ID:
        Image:          nginx:latest
        Image ID:
        Port:           <none>
        Host Port:      <none>
        State:          Waiting
        Reason:       ContainerCreating
        Ready:          False
        Restart Count:  0
        Environment:
        HELLO:  world
        TEST:   test_that
        Mounts:
        /tmp/test from test-vol (rw)
        /var/run/secrets/kubernetes.io/serviceaccount from default-token-m7jqs (ro)
    sidecar-debian:
        Container ID:
        Image:          debian:latest
        Image ID:
        Port:           80/TCP
        Host Port:      0/TCP
        State:          Waiting
        Reason:       ContainerCreating
        Ready:          False
        Restart Count:  0
        Environment:
        ENV_IN_SIDECAR:  test-in-sidecar
        HELLO:           world
        TEST:            test_that
        Mounts:
        /tmp/test from test-vol (rw)
    Conditions:
    Type              Status
    Initialized       True
    Ready             False
    ContainersReady   False
    PodScheduled      True
    ***

**Chúng ta đã có 1 sidecar debian chạy cùng nginx trong POD**

Tham vấn thêm: *www.google.com*