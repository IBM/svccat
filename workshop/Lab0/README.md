# Lab 0: Install prequisites

Before we begin, we need to install Service Catalog and a few other things
we'll need. Towards that end, this lab contains instructions to install:

* The Helm CLI, version 2.7.0 or later
* The Helm tiller pod inside your cluster
* Service Catalog, version 0.1.18 or later
* svcat, the Service Catalog CLI tool
* Download the source code for the workshop
* Create a PersistantVolume on our cluster

# Set KUBECONFIG

Make sure your KUBECONFIG environment variable is set to the output of

`$ bx cs cluster-config [cluster-name]`

or otherwise targeting your IBM Cloud Kubernetes Service cluster.

# Install Helm

1. Download and install [the Helm CLI](https://github.com/kubernetes/helm#install).
2. Run `helm init` to install the tiller pod into your kube cluster. This pod is Helm's entry point
for creating additional infrastructure inside your cluster.

# Install Service Catalog

Service Catalog is installed via a Helm chart, a manifest that describes what to install and how to run it.

1. First, add the Service Catalog repo to your helm by running 
```
$ helm repo add svc-cat https://svc-catalog-charts.storage.googleapis.com
```
You should see the repository listed in the output of `helm search service-catalog`.

2. Install Service Catalog by running 
```
$ helm install svc-cat/catalog --name catalog --namespace catalog
```

You should be able to see the helm tiller and service catalog pods like so:
```console
$ kubectl get pods --all-namespaces
NAMESPACE     NAME                                                  READY     STATUS    RESTARTS   AGE
catalog       catalog-catalog-apiserver-6bc6bb895d-f2xhp            2/2       Running   0          8m
catalog       catalog-catalog-controller-manager-5c7475f7d4-fwlfg   1/1       Running   1          8m
kube-system   kube-addon-manager-minikube                           1/1       Running   0          8m
kube-system   kube-dns-54cccfbdf8-vpvmn                             3/3       Running   0          8m
kube-system   kubernetes-dashboard-77d8b98585-sw5gc                 1/1       Running   0          8m
kube-system   storage-provisioner                                   1/1       Running   0          8m
kube-system   tiller-deploy-865dd6c794-88njn                        1/1       Running   0          9m
```

# Install svcat

svcat is a command line tool for interacting with Service Catalog. While Service Catalog
can be intereacted with solely with kubectl, svcat provides a nicer UX that is customized
to interact with Service Catalog specific resource types. It can be run as a kubectl plugin,
or as a standalone binary. Download the version that corresponds to your operating system:

* [Linux](https://download.svcat.sh/cli/latest/linux/amd64/svcat)
* [OS X](https://download.svcat.sh/cli/latest/darwin/amd64/svcat)
* [Windows](https://download.svcat.sh/cli/latest/windows/amd64/svcat.exe)

Once you've downloaded the binary, make sure it's executable and move it somewhere on
your PATH. Ensure that you can see svcat on your PATH and it can access your 
Kubernetes deployment:
```console
$ svcat get classes
  NAME   DESCRIPTION   UUID  
+------+-------------+------+
```

# Download the Workshop Source Code

Within this repository are the step by step instructions and some yaml manifests
needed to run the workshop. Clone it for later use:

```
$ git clone https://github.com/IBM/svccat.git
```

# Create a Private Volume

The service broker we will be deploying as part of this workshop will deploy resources into your Kubernetes cluster. In order
for these services to have a place to store data, we need to create a Persistant Volume for them to mount. Find the `pv.yaml`
file in the Lab0 directory of this repository, and then run 
```
kubectl create -f pv.yaml
```
This step is only needed to support the specific service (redis) we'll be deploying using a service broker on IBM Cloud IKS. It is not a normal step of the
Service Catalog workflow.


After that, you should be [good to go!](../Lab1)
