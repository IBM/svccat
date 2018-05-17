# Lab 0: Install Service Catalog and svcat

Before we begin, we need to install Service Catalog and its associated CLI utility, svcat.
Towards that end, this lab contains instructions to install:

* Configure your IBM Cloud Kubernetes cluster's config map
* The Helm CLI, version 2.7.0 or later
* The Helm tiller pod inside your cluster
* Service Catalog, version 0.1.18 or later
* svcat, the Service Catalog CLI tool

# Configure Configmap

In order for the API aggregation for Service Catalog to work, we need to configure Kubernetes
to allow connections to the Service Catalog apiserver. When you run 

`kubectl -n kube-system get configmap extension-apiserver-authentication -o yaml`

you should get something that looks like this:

```
apiVersion: v1
data:
  client-ca-file: |
    [ca cert contents]
kind: ConfigMap
metadata:
  creationTimestamp: 2018-05-15T21:54:32Z
  name: extension-apiserver-authentication
  namespace: kube-system
  resourceVersion: "40"
  selfLink: /api/v1/namespaces/kube-system/configmaps/extension-apiserver-authentication
  uid: 8cffeb79-588a-11e8-a7c5-080027622ad8
```

We need to add a few additional entries for the extension server. Do this by running the command

`kubectl -n kubesystem edit configmap extension-apiserver-authentication`

and inserting the bolded sections. Note that you will have to copy the pre-existing CA cert from the
client-ca-file section into the request-header-client-ca-file section.

<pre>
apiVersion: v1
data:
  client-ca-file: |
    [ca cert contents]
  <b>requestheader-allowed-names: '[]'
  requestheader-client-ca-file: |
    [copy ca cert from above]
  requestheader-extra-headers-prefix: '["X-Remote-Extra-"]'
  requestheader-group-headers: '["X-Remote-Group"]'
  requestheader-username-headers: '["X-Remote-User"]'</b>
kind: ConfigMap
metadata:
  creationTimestamp: 2018-05-15T21:54:32Z
  name: extension-apiserver-authentication
  namespace: kube-system
  resourceVersion: "40"
  selfLink: /api/v1/namespaces/kube-system/configmaps/extension-apiserver-authentication
  uid: 8cffeb79-588a-11e8-a7c5-080027622ad8
</pre>

After doing this, you should be able to run 

`kubectl -n kube-system get configmap extension-apiserver-authentication -o yaml`

and see the newly added sections in the output.

# Install Helm

1. Download and install [the Helm CLI](https://github.com/kubernetes/helm#install).
2. Run `helm init` to install the tiller pod into your kube cluster. This pod is Helm's entry point
for creating additional infrastructure inside your cluster.
3. To allow the tiller pod permissions to perform changes inside your cluster, run
`kubectl create clusterrolebinding tiller-cluster-admin \
    --clusterrole=cluster-admin \
    --serviceaccount=kube-system:default
`

# Install Service Catalog

Service Catalog is installed via a Helm chart, a manifest that describes what to install and how to run it.

1. First, add the Service Catalog repo to your helm by running `helm repo add svc-cat https://svc-catalog-charts.storage.googleapis.com`
You should see the repository listed in the output of `helm search service-catalog`.
2. Install Service Catalog by running 
`helm install svc-cat/catalog \
    --name catalog --namespace catalog`
You should be able to see the helm tiller and service catalog pods like so:
```
> kubectl get pods --all-namespaces
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

svcat is a command line tool for interacting with Service Catalog. It can be run as a kubectl plugin, or as a standalone binary.
Download the version that corresponds to your operating system:

* [Linux](https://download.svcat.sh/cli/latest/linux/amd64/svcat)
* [OS X](https://download.svcat.sh/cli/latest/darwin/amd64/svcat)
* [Windows](https://download.svcat.sh/cli/latest/windows/amd64/svcat.exe)

Make the binary executable:

* (Linux/OS X) `chmod +x svcat`
* (Windows) Make sure the filename ends in .exe

Move the file somewhere on your PATH:

* (Linux/OS X) `mv svcat /usr/local/bin/`
* (Windows) `mkdir -f ~\bin
$env:PATH += ";${pwd}\bin"`

Ensure that you can see svcat and it can access your Kubernetes deployment:
```
> svcat get classes
  NAME   DESCRIPTION   UUID  
+------+-------------+------+
```

# Download the Workshop Source Code

Repo `guestbook` has the application that we'll be deploying. 
While we're not going to build it we will use the deployment configuration files from that repo. 
Guestbook application has two versions v1 and v2 which we will use to demonstrate some rollout 
functionality later. All the configuration files we use are under the directory guestbook/v1.

Repo `svccat` contains the step by step instructions to run the workshop.

```console
$ git clone https://github.com/IBM/guestbook.git
$ git clone https://github.com/IBM/svccat.git
```

After that, you should be good to go!
