# Lab 2 Deploy an application with a manually provisioned backing store

In this lab we'll learn to manually provision a Redis instance, and manually connect it to an
application hosted on the IBM Kubernetes Service.

# Deploy your application

If you're continuing from Lab 1, you can skip this step. Otherwise, run

```console
$ kubectl run guestbook --image=ibmcom/guestbook:v1
$ kubectl expose deployment guestbook --type="NodePort" --port=3000
```

This will deploy Guestbook and expose a port so you can connect to it. Note that
it is storing state locally in memory.

# Manually provision a Redis instance

We want some sort of stable backing store for our application to store it's state in. To start with,
we'll manually create a Redis deployment within out Kubernetes cluster. As we're creating this ourselves,
we would also be resoponsible for the continued maintenance and support of this Redis deployment. To
provision it, run:
```console
$ kubectl create -f redis-master.yaml
deployment "redis-master" created
```
Then check to see if the redis pod is running:
```console
$ kubectl get pods
NAME                            READY     STATUS    RESTARTS   AGE
redis-master-6767cf65c7-qpnmd   1/1       Running   0          11s
```

# Connect Guestbook to your Redis instance.

Once Redis is up and running, we need to somehow get this information into our application. For this workshop,
we'll create a Kubernetes nodeport service, which will allow us to connect to our Redis pods. Note that this is
a stand-in for any externally provisioned service, such as an externally-hosted Redis, a Redis cluster running
in the same data center as our Kubernetes cluster, etc. To expose the Redis deployment, run:
```console
$ kubectl expose deployment redis-master --type Nodeport --port 6379
service "redis-master" exposed
```

Now that our Redis pods are external exposed, find the information needed to connect them to our application.
```console
$ bx cs workers osscluster
OK
ID                                                 Public IP        Private IP     Machine Type   State    Status   Zone    Version  
kube-hou02-pa1e3ee39f549640aebea69a444f51fe55-w1   173.193.99.136   10.76.194.30   free           normal   Ready    hou02   1.5.6_1500*
$ kubectl get services
NAME           TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)          AGE
guestbook      NodePort    172.21.44.221    <none>        3000:31084/TCP   3d
kubernetes     ClusterIP   172.21.0.1       <none>        443/TCP          3d
redis-master   NodePort    172.21.240.247   <none>        6379:30693/TCP   1m
```
The external IP address of our cluster is 173.193.99.136, the external port of our redis-master
service is 30693. By default the Redis deployment was deployed with a password of 'foobar'. We
can add this information to our applicaiton using `kubectl edit` and exposing them as
environment variables:

```
$ kubectl edit deployment guestbook
<pre>
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  annotations:
    deployment.kubernetes.io/revision: "1"
  creationTimestamp: 2018-05-22T00:14:04Z
  generation: 1
  labels:
    app: guestbook
  name: guestbook
  namespace: default
  resourceVersion: "597"
  selfLink: /apis/extensions/v1beta1/namespaces/default/deployments/guestbook
  uid: 099103c8-5d55-11e8-826c-7273b59914b4
spec:
  progressDeadlineSeconds: 600
  replicas: 1
  revisionHistoryLimit: 10
  selector:
    matchLabels:
      app: guestbook
  strategy:
    rollingUpdate:
      maxSurge: 25%
      maxUnavailable: 25%
    type: RollingUpdate
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: guestbook
    spec:
      containers:<b>   
      - env:
        - name: REDIS_MASTER_SERVICE_HOST
          value: "INSERT HOSTNAME HERE"
        - name: REDIS_MASTER_SERVICE_PORT
          value: "INSERT PORT HERE"
        - name: REDIS_MASTER_SERVICE_PASSWORD
          value: "INSERT PASSWORD HERE"</b>
        image: ibmcom/guestbook:v1
        imagePullPolicy: Always
        name: guestbook
        ports:
        - containerPort: 3000
          name: http-server
          protocol: TCP
        resources: {}
        terminationMessagePath: /dev/termination-log
        terminationMessagePolicy: File
      dnsPolicy: ClusterFirst
      restartPolicy: Always
      schedulerName: default-scheduler
      securityContext: {}
      terminationGracePeriodSeconds: 30
status:
  availableReplicas: 1
  conditions:
  - lastTransitionTime: 2018-05-22T00:14:09Z
    lastUpdateTime: 2018-05-22T00:14:09Z
    message: Deployment has minimum availability.
    reason: MinimumReplicasAvailable
    status: "True"
    type: Available
  - lastTransitionTime: 2018-05-22T00:14:04Z
    lastUpdateTime: 2018-05-22T00:14:09Z
    message: ReplicaSet "guestbook-6b556d4cf8" has successfully progressed.
    reason: NewReplicaSetAvailable
    status: "True"
    type: Progressing
  observedGeneration: 1
  readyReplicas: 1
  replicas: 1
  updatedReplicas: 1
  </pre>
```

Once you've done that, Kubernetes should automatically detect the change in your deployment,
and restard the pod. You can see this if you quickly type `kubectl get pods`.
```console
$ kubectl get pods
NAME                            READY     STATUS        RESTARTS   AGE
guestbook-6b556d4cf8-jdztk      1/1       Terminating   0          47s
guestbook-7dd97d9cb5-sk2jm      1/1       Running       0          2s
redis-master-7cfd6dfb86-wfgs7   1/1       Running       0          9m
```

Once the new pod is running, you should be able to head back to your webpage, and see 
Guestbook running. If you press the /info link, you should see the information about 
the Redis instance you're now using as a backing store. Try inputtig a few values into 
the application. Then, run `kubectl get pods` and grab the name of your pod. Run 
`kubectl delete pod PODNAME` and then wait for Kubernetes to deploy a new pod. 
When you refresh your page, the information should still be there.

So now we've connected our application to some stable storage to persist our data, but manually
provisioning the Redis instance and manually editing our deployment yaml is a bit cumbersome.
In [the next lab of this course](../Lab3/README.md) we'll learn to abstract some of that hassle
away by using a Kubernetes Secret to encapsulate our credentials.
