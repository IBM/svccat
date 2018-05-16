# Lab 3 Deploy an application with a secret containing creds to a backing store

In this lab we'll learn how to create a Kubernetes secret containing the credentials to connect to a service,
and attach that secret to an application to consume that service.

# Create a secret

[Secrets](https://kubernetes.io/docs/concepts/configuration/secret/) are a Kubernetes resource
intended to hold sensitive information such as usernames, passwords, or tokens. While your 
application may need this information to function, it is common that you wish to restrict
access to this information. Previously, we were hardcoding our sensitive info directly into
our deployment declaration, where it was readable by anyone with access to our deployment.

To create a secret, encode the values you wish it to contain in base64, and then add them to
the secrets.yaml file, like so:

```
apiVersion: v1
kind: Secret
metadata:
  name: guestbookcreds
type: Opaque
data:
  host: INSERT_HOST_HERE
  port: INSERT_PORT_HERE
  password: INSERT_PASSWORD_HERE
```

To encode a value in base64, use the `base64` terminal command:

```
echo -n "INSERT_HOST_HERE" | base64
```

This should output a long string such as
`cHViLXJlZGlzLTE3NjgxLmRhbC0wNS4xLnNsLmdhcmFudGlhZGF0YS5jb20K`. Note the -n
flag for echo so the resulting string doesn't contain a newline character.
Insert these values into the relevant fields, and then create the secret:

```
kubectl create -f secret.yaml
```

You should be able to see it by getting secrets:

```console
$ kubectl get secrets
NAME                                   TYPE                                  DATA      AGE
bluemix-default-secret                 kubernetes.io/dockercfg               1         23h
bluemix-default-secret-international   kubernetes.io/dockercfg               1         23h
bluemix-default-secret-regional        kubernetes.io/dockercfg               1         23h
default-token-5pkkn                    kubernetes.io/service-account-token   3         23h
guestbookcreds                         Opaque                                3         9m
```

If you `kubectl get secret guestbookcreds -o yaml`, you should be able to use base64 to decode
the fields back into the actual data.

```console
$ kubectl get secret guestbookcreds -o yaml
apiVersion: v1
data:
  host: cHViLXJlZGlzLTE3NjgxLmRhbC0wNS4xLnNsLmdhcmFudGlhZGF0YS5jb20K
  password: ekExNU40eXZ0bTJ0WnVJSUd3SDhiVUxRV2dZYmVXMlgK
  port: MTc2ODEK
kind: Secret
metadata:
  creationTimestamp: 2018-05-22T22:55:50Z
  name: guestbookcreds
  namespace: default
  resourceVersion: "9245"
  selfLink: /api/v1/namespaces/default/secrets/guestbookcreds
  uid: 461722a2-5e13-11e8-826c-7273b59914b4
type: Opaque
$ echo "cHViLXJlZGlzLTE3NjgxLmRhbC0wNS4xLnNsLmdhcmFudGlhZGF0YS5jb20K" | base64 -d
pub-redis-17681.dal-05.1.sl.garantiadata.com
```

Note that the OSX base command uses an uppercase `-D` flag.


# Attach the secret to your Guestbook deployment

Now that the secret is in Kubernetes, we need to attach it to our deployment. Run
`kubectl edit deployment guestbook` and replace the values in the env section we added last time with a
reference to the values in our new secret:

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
      containers:
      - env:
        - name: REDIS_MASTER_SERVICE_HOST<b>
          valueFrom:
            secretKeyRef:
              name: guestbookcreds
              key: host</b>
        - name: REDIS_MASTER_SERVICE_PORT<b>
          valueFrom:
            secretKeyRef:
              name: guestbookcreds
              key: port</b>
        - name: REDIS_MASTER_SERVICE_PASSWORD<b>
          valueFrom:
            secretKeyRef:
              name: guestbookcreds
              key: password</b>
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
    message: ReplicaSet "guestbook-7bdf589c97" has successfully progressed.
    reason: NewReplicaSetAvailable
    status: "True"
    type: Progressing
  observedGeneration: 1
  readyReplicas: 1
  replicas: 1
  updatedReplicas: 1
  </pre>

Alternatievely, you can fill in this information in secret-guestbook.yaml, delete your old deployment
with `kubectl delete deployment guestbook` and recreate it with `kubectl create -f secret-guestbook.yaml`.

In either case, if you refresh the Guestbook page in your web  browser, you should see Guestbook is still
connected to the database. The /info and /env pages should look the same as before. We've accomplished the
the same end state, but now your secret and deployments can be managed separately from each other.

In [next lab of this course](../Lab4/README.md) we will learn a little bit about the Open Service
Broker API, create a broker, and connect it to Kubernetes using the Service Catalog.
