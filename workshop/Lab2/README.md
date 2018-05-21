# Lab 2 Deploy an application with a manually provisioned backing store

Learn to manually provision a Redis instance, and manually connect it to an
application hosted on the IBM Kubernetes Service.

# Deploy your application

If you're continuing from Lab 1, you can skip this step. Otherwise, run

```
$ kubectl run guestbook --image=ibmcom/guestbook:v1
$ kubectl expose deployment guestbook --type="NodePort" --port=3000
```

This will deploy Guestbook and expose a port so you can connect to it. Note that
it is storing state locally in memory.

# Manually provision a Redis instance

We want some sort of stable backing store for our application to store it's state in. Look on
the [IBM Cloud Catalog](https://console.bluemix.net/catalog/) for the Redis Cloud service. Provision
and instance of the free 30 MB tier service.

![Redis Service](link here)
