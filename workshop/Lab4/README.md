# Lab 4: Deploy a service broker

In this lab we'll learn how to deploy a service broker, connect it to Kubernetes using Service Catalog,
and view the catalog of services.

# Open Service Broker API

The [Open Service Broker API (OSBAPI)](https://www.openservicebrokerapi.org/) is an open
standard that allows users of cloud platforms to provision, manage, and consume services
in an automated fashion. We are going to create a service broker to automate deploying our
Redis for us, as well as creating the secret that we consume in our deployment. If you're
interested in what's going on behind the scenes, see [here](osbapi.md).

# Deploy a broker

For the purposes of this lab, we'll be using [Minibroker](https://github.com/osbkit/minibroker),
a broker that we will deploy to our Kubernetes cluster. In the real world, the broker could be deployed
within our cluster, or next to our cluster in the same data center, or out on the Internet, or anywhere
in between. We'll be using Helm to install Minibroker, like so:

```
helm repo add minibroker https://minibroker.blob.core.windows.net/charts
helm install --name minibroker --namespace minibroker minibroker/minibroker
```

You should be able to see the minibroker pods start up. Note that it's running in the `minibroker`
namespace, which will also be the namespace it deploys subsequent resources into.
```console
$ kubectl get pods -n minibroker
NAMESPACE     NAME                                                  READY     STATUS    RESTARTS   AGE
minibroker    minibroker-minibroker-754468f859-6wq7h                2/2       Running   0          48s
```

The helm chart that installs minibroker should also register it with Service Catalog. Check this like so:
```console
$ svcat get brokers
     NAME                                 URL                              STATUS
+------------+-----------------------------------------------------------+--------+
  minibroker   http://minibroker-minibroker.minibroker.svc.cluster.local   Ready
```

What's going on in OSBAPI terms here is once the broker is running, Service Catalog sends a request to
the `/v2/catalog` endpoint on Minibroker. Minibroker returns information about all the services it offers.
You can see a condensed version of this information by running:
```console
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

We'll use Minibroker in [the next lab](../Lab5/README.md), where we will use it to deploy a Redi Service
Instance, and connect it to our Guestbook deployment.


