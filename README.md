# svccat

This is a collection of materials intended to educate about the use of [Service Catalog](https://github.com/kubernetes-incubator/service-catalog", a
Kubernetes extension project that implements the [Open Service Broker API](https://github.com/openservicebrokerapi/servicebroker). This allows
cloud application developers to easily provision web services and connect them to their application.

# Presentation

The [presentation](https://github.com/IBM/svccat/tree/master/presentation) section contains a slide deck covering the use case for OSBAPI, as well
as an overview of the interface between a cloud platform and an OSB broker. It then covers the details of Service-Catalog, the Kube implementation of OSBAPI.

# Workshop
The [workshop](https://github.com/IBM/svccat/tree/master/workshop) section contains materials for several guided-walkthroughs of running a [Guestbook](https://github.com/IBM/guestbook)
deployment on Kubernetes, first as a stand alone application storing state in-memory, and then progressing through various ways of automating and simplfying the provision and attachment
of a Redis backing store. It details how to manually provsion and attach a Redis backing store, how to use a Kube secret to contain these manually created credentials, the deployment
and registering of an OSB broker with Service Catalog, and finally the use of that broker to provision and bind a Redis service instance to your Guestbook application.

The various walkthroughs and demos of this repository can be used with a free-tier [IBM Cloud account](https://www.ibm.com/cloud/), so they can be done without the need to locally deploy
and manage a Kubernetes cluster.
