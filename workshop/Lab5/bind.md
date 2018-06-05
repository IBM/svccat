# Open Service Broker API - Binding a Service Instance

Binding is the action by which a cloud platform requests for a set of credentials to access
a provisioned Instance. In the Redis example, this would be asking the Redis Broker for a set
of credentials to access Jonathan's v4.0.9 Redis store.

# /v2/service_instances/:service_id/service_bindings/:binding_id Endpoint

In Lab 5, we creating a Service Binding using svcat. When that happened, Service Catalog
issued a PUT request to the /v2/service_instances/:service_id/service_bindings/:binding_id
endpoint of Minibroker. The request contains the ID of the Instance we wish to create a Binding
for, a unique ID for the Binding we are creating, as well as the ID for the Class and Plan. Our
Broker then went and created a new set of credentials for our Redis instance, but other brokers
might dole out pre-existing credentials, or provide access to a single set of credentials, or
something else entirely. The request flow between the platform and the broker looks like this:

![/v2/service_instances/:service_id/service_bindings/:binding_id](../images/binding_request.png)


:service_id is the unique ID that the platform provided in the [provision request](provision.md). 
Similarly, :binding_id is some unique ID that the platform provides that will be used to reference
this particular Binding in future requests. The PUT request contains a JSON body that contains 
the additional information needed to create the Binding:
```
{
  "context": {
    "platform": "kubernetes",
    "some_field": "some-contextual-data"
  },
  "service_id": "abc",
  "plan_id": "123",
  "bind_resource": {
    "app_guid": "app-guid-here"
  },
  "parameters": {
    "parameter1-name-here": 1,
  }
}
```
This is similar to the JSON body for the provision request. Service ID and Plan ID are the IDs for
the Class and Plan we are creating a binding for. Context is a map containing information about the
requesting platform. Bind resource is a map containing additional information about the object the binding
is being created for. Parameters is a map where additional parameters specific to this Instance or Broker
can be provided.

The 201 CREATED response that the Broker can vary wildly. The credentials we get back from Minibroker
look something like this:
```
{
  "credentials": {
    "Protocol": "redis",
    "host": "jumpy-albatross-redis-master.default.svc.cluster.local",
    "password": "2dkVG7C4YQ"
    "port": 6379,
    "redis-password": "2dkVG7C4YQ",
    "redis-uri": "redis://:2dkVG7C4YQ@jumpy-albatross-redis-master.default.svc.cluster.local:6379"
  }
}
```

The only required field here is credentials - what it contains is entirely up to the broker. In this
example response, we have several fields detailing how to connect to a MySQL database. Other options
might be an OAuth token for connecting, or CA certificates for MTLS authentication. You need to be aware
of what the credentials for a particular Instance will look like to be able to consume them within
your cloud platform and from there within your application.
