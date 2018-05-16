# An introduction to Kubernetes Service Catalog

[Service Catalog](https://github.com/kubernetes-incubator/service-catalog) lets you provision cloud services and attach them to running Kubernetes deployments 
without having to manage those resources directly. This is done through the use of a broker, a web
server that implements the [Open Service Broker API](https://www.openservicebrokerapi.org/).

# Objectives
These labs are a introduction to the installation and use of Service Catalog inside a Kubernetes cluster. By the end of this course, you'll:
* Understand the core concepts of the Open Service Broker API
* Deploy an application with in memory state
* Manually deploy a backing store
* Manually attach the backing store to your application
* Deploy a service broker, attach it to your Kubernetes deployment, and use it to automatically provision a backing store
* Bind this OSB-provisioned backing store to your application running on Kubernetes

# Prerequesites
* Have completed [Kube101](https://github.com/IBM/kube101/tree/master/workshop).
* Have a running Kubernetes cluster as per Kube101
* A free-tier [IBM Cloud account](https://console.bluemix.net/registration/)

# Open Service Broker API (OSBAPI)
Cloud native applications are ephemeral and often have no local stable storage, but still require access to databases, caches,
email systems, billing systems, etc. in order to function. OSBAPI is an open API that allows the concerns
of provisioning and maintaining these services to be abstracted away from the application developer and the cloud platform provider.

Instead of having to manually provision a database and get the connection information into your application somehow, you can
use an Open Service Broker to provision a database for you, to allocated credentials for accessing it, and inject that information into your app.
While the archetypal example is a relational database, any dependent service that can be decomposed into the five OSB actions can be provided as an
OSB service. 

# Open Service Broker Abstractions
OSBAPI defines five object types - Brokers, Services, Plans, Instances, and Bindings.
* Broker - A server that implements OSB API. Offers a catalog of services via the endpoints outlined in the OSB API specification, e.g. MySQL Broker.
* Service - A category of services offered by a broker, e.g. MySQL databases.
* Plan - A specific type of an offered service. Plans are often used to represent different quality levels of a service. e.g. 100 MB MySQL databases.
* Instance - A specific instantiation of a plan, e.g. Jonathan's 100 MB MySQL database.
* Binding - A unique set of credentials to access a specific instance , e.g. username/password/hostname/port to access Jonathan's 100 MB MySQL database.

To get started, head over to [Lab0](./Lab0) to setup some prerequisites for the first lab. 
