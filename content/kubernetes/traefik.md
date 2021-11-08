---
title: "Traefik"
date: 2020-12-02T16:00:00+02:00
order: 2
---


Traefik is a load balancer with dynamic configuration reload that can be plugged with various backend to enable routing automatically and it work with Kubernetes.

Traefik is installed by default on `k3s` clusters so we could use it.

# Accessing dashboard

Traefik dashboard is not exposed through the exterior of the cluster by default. We need to add a `IngressRoute` configuration to enable it.

```yaml
apiVersion: traefik.containo.us/v1alpha1
kind: IngressRoute
metadata:
  name: traefik-external-dashboard
  namespace: kube-system
spec:
  entryPoints:
  - web
  routes:
  - kind: Rule
    match: PathPrefix(`/dashboard`) || PathPrefix(`/api`)
    services:
    - kind: TraefikService
      name: api@internal
```

Apply this ressource in your cluster and then test to access the Traefik dashboard on http://10.0.0.11/dashboard/#/.  
You should access the traefik dashboard.

![traefik dashboard](traefik_dash.png)


# Expose a service

Let's first create a deployment with multiple replicas.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: whoami-deployment
  labels:
    app: whoami
spec:
  replicas: 3
  selector:
    matchLabels:
      app: whoami
  template:
    metadata:
      labels:
        app: whoami
    spec:
      containers:
      - name: whoami
        image: containous/whoami:v1.5.0
        imagePullPolicy: IfNotPresent
        ports:
        - containerPort: 80
```

Check the deployment and that all the pods are healthy.

Now that we have our application deployed, let's create a kubernetes Service to provide an entrypoint in the server for our application.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: whoami-service
spec:
  selector:
    app: whoami
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
```

At this point, it's standard Kubernetes ressoucres. We created an application deployment with 3 replicas and a service to be able to communicate in the cluster with this application.  
Now let's create a specific traefik `IngressRoute` to expose our Service to the external world.

```yaml
apiVersion: traefik.containo.us/v1alpha1
kind: IngressRoute
metadata:
  name: whoami-ingress
spec:
  entryPoints:
    - web
  routes:
  - match: PathPrefix(`/whoami`)
    kind: Rule
    services:
    - name: whoami-service
      namespace: default
      port: 80
```

As you can see in the `apiVersion` this ressources is not a standard Kubernetes one, it's a Traefik specific one.  
Kubernetes allow third party system to create and manage ressources in the cluster with a mecanism named `Custom Ressource Definition`. We will not cover this in this course, but you could look about it by yourself.

This ressource create a descriptor for an `IngressRoute` as in route for incoming traffic to define some rule and send them to a target service.  
Here we defined that all request atching a path starting with `/whoami` should be sent to `whoami-service`.  

We could check that our application is availble from the outside of our cluster : http://10.0.0.11/whoami  
Check also the Treafik dashboard to look on how it's configured inside Traefik.

# Conclusion

We just have used a solution of Ingress controller that is available on Kubernetes. There are other solutions avaiables like NGinx, HAProxy, ...  You can find a more exhaustive list on the [official documentation](https://kubernetes.io/docs/concepts/services-networking/ingress-controllers/)

To find more information on Traefik, please look at the [official website](https://traefik.io/traefik/)