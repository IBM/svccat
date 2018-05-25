# Lab 5 Deploy an OSB service and attach it to a deployment

Learn to provision a service instance, create a binding for your service instance,
and use that binding to connect your deployment to that service.

# Before we begin

If you skipped Lab0 or otherwise didn't create a Persistant Volume, we need one for Minibroker
to bind the Redis instance to. Find the pv.yaml file in the Lab0 directory, and run
`kubectl create -f pv.yaml`. Note that while we need to do this because our broker is
provisionin resources inside our Kubernetes cluster, this is not part of the normal
OSB service workflow.

# Provision a Service Instance

Now that we have Minibroker up and running in our Kube cluster, we can use it to provision
a redis instance. If we run `svcat get classes` we can see all the services offered by Minibroker:
```
> svcat get classes
     NAME             DESCRIPTION             UUID     
+------------+---------------------------+------------+
  mariadb      Helm Chart for mariadb      mariadb     
  mongodb      Helm Chart for mongodb      mongodb     
  mysql        Helm Chart for mysql        mysql       
  postgresql   Helm Chart for postgresql   postgresql  
  redis        Helm Chart for redis        redis    
```

The same can be done for the Plans of those Classes by running `svcat get plans`. Note that
this response contains a bit more info, so it can display quite a lot of text.
```
> svcat get plans
   NAME       CLASS               DESCRIPTION                   UUID        
+---------+------------+--------------------------------+------------------+
.
.
.
  3-2-9     redis        Open source, advanced            redis-3-2-9
                         key-value store. It is
                         often referred to as a data
                         structure server since keys
                         can contain strings, hashes,
                         lists, sets and sorted sets.
  4-0-2     redis        Open source, advanced            redis-4-0-2
                         key-value store. It is
                         often referred to as a data
                         structure server since keys
                         can contain strings, hashes,
                         lists, sets and sorted sets.
  4-0-6     redis        Open source, advanced            redis-4-0-6
                         key-value store. It is
                         often referred to as a data
                         structure server since keys
                         can contain strings, hashes,
                         lists, sets and sorted sets.
  4-0-7     redis        Open source, advanced            redis-4-0-7
                         key-value store. It i
                         often referred to as a data
                         structure server since keys
                         can contain strings, hashes,
                         lists, sets and sorted sets.
  4-0-8     redis        Open source, advanced            redis-4-0-8
                         key-value store. It is
                         often referred to as a data
                         structure server since keys
                         can contain strings, hashes,
                         lists, sets and sorted sets.
  4-0-9     redis        Open source, advanced            redis-4-0-9
                         key-value store. It is
                         often referred to as a data
                         structure server since keys
                         can contain strings, hashes,
                         lists, sets and sorted sets.
```

The numbered tuples are the version of the helm chart minibroker uses to deploy the services we ask it to provison.
For the purposes of this workshop we'll be using the redis Class and the 3-2-9 plan.

You can use svcat to provision an instance by running `svcat provisison myredis --class=redis --plan=3-2-9`.
After that, 
