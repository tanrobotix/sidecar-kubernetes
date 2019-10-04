# TLS Generator

This is stuff useful for new users who need to generate a new TLS cert for deploying the sidecar injector.

## Tutorial

### Tạo TLS Cert:

Giả sử chúng ta có Cluster là k8s Cluster SANDBOX, generate TLS Cert như sau:

    $ cd ./tls/
    $ DEPLOYMENT=k8s-cluster CLUSTER=SANDBOX ./new-cluster-injector-cert.rb
    $ git ls-files -o
    .srl
    us-east-1/PRODUCTION/ca.crt
    us-east-1/PRODUCTION/ca.key
    us-east-1/PRODUCTION/sidecar-injector.crt
    us-east-1/PRODUCTION/sidecar-injector.csr
    us-east-1/PRODUCTION/sidecar-injector.key
