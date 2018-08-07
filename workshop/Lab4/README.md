# Lab 4 Deploy a service broker

Learn to deploy a service broker, connect it to Kubernetes using Service Catalog,
and view the catalog of services.

# Open Service Broker API

The [Open Service Broker API (OSBAPI)](https://www.openservicebrokerapi.org/) is an open
standard that allows users of cloud platforms to provision, manage, and consume services
in an automated fashion. We are going to create a service broker to automate deploying our
Redis for us, as well as creating the secret that we consume in our deployment. If you're
interested in what's going on behind the scenes, see [here](osbapi.md)

# Deploy a broker

For the purposes of this lab, we'll be using [Minibroker](https://github.com/osbkit/minibroker),
a broker that lives inside our Kubernetes cluster. In the real world, the broker could be deployed
within our cluster, or next to our cluster in the same data center, or out on the Internet, or anywhere
in between. We'll be using Helm to install Minibroker, like so:

```
helm repo add minibroker https://minibroker.blob.core.windows.net/charts
helm install --name minibroker --namespace minibroker minibroker/minibroker
```

You should be able to see the minibroker pods start up. Note that it's running in the `minibroker`
namespace, which will also be the namespace it deploys subsequent resources into.
```
$ kubectl get pods --all-namespaces
NAMESPACE     NAME                                                  READY     STATUS    RESTARTS   AGE
catalog       catalog-catalog-apiserver-85f87465b-zmtv9             2/2       Running   0          3m
catalog       catalog-catalog-controller-manager-674d789798-6nqwf   1/1       Running   0          3m
default       guestbook-c7ff4f448-vtsw9                             1/1       Running   0          18h
kube-system   calico-kube-controllers-5f4bbd9b76-7v2bj              1/1       Running   0          1d
kube-system   calico-node-z5z2s                                     2/2       Running   0          1d
kube-system   heapster-5656698f8b-zddjb                             2/2       Running   0          1d
kube-system   ibm-keepalived-watcher-7v29v                          1/1       Running   0          1d
kube-system   kube-dns-amd64-6d94d697d6-4m6cl                       3/3       Running   0          1d
kube-system   kube-dns-autoscaler-6d4cb99db5-ldxdz                  1/1       Running   0          1d
kube-system   kubernetes-dashboard-74847f67d6-cn8wz                 1/1       Running   0          1d
kube-system   tiller-deploy-865dd6c794-585fb                        1/1       Running   0          1h
kube-system   vpn-7cc5766698-f6m58                                  1/1       Running   0          1d
minibroker    minibroker-minibroker-754468f859-6wq7h                2/2       Running   0          48s
```

The helm chart that installs minibroker should also register it with Service Catalog. Check this like so:
```
$ svcat get brokers
     NAME                                 URL                              STATUS
+------------+-----------------------------------------------------------+--------+
  minibroker   http://minibroker-minibroker.minibroker.svc.cluster.local   Ready
```

What's going on in OSBAPI terms here is once the broker is running, Service Catalog sends a request to
the `/v2/catalog` endpoint on Minibroker. Minibroker returns information about all the services it offers.
You can see a condensed version of this information by running:
```
$ svcat get classes
     NAME             DESCRIPTION             UUID
+------------+---------------------------+------------+
  mariadb      Helm Chart for mariadb      mariadb
  mongodb      Helm Chart for mongodb      mongodb
  mysql        Helm Chart for mysql        mysql
  postgresql   Helm Chart for postgresql   postgresql
  redis        Helm Chart for redis        redis
```

This shows all the service classes currently available on our Service Catalog, which at the moment just has
Minibroker installed. In an actual environment, you'd likely have multiple brokers installed offering dozens
of services or more.

We'll use Minibroker in [the next lab](../Lab5/README.md), or you can remove the broker and stop here.

  1. To remove Minibroker, use `helm delete minibroker --purge`
