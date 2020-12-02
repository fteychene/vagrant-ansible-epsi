---
title: "Getting Started with k3s"
date: 2020-12-02T16:00:00+02:00
order: 0
---

In this hands-on we will work using [k3s](https://k3s.io/), it's a lightweight Kubernetes distribution easy to install.  
We will not work with vanilla kubernetes as its installation could be challenging.

Using k3s should not change your experience since we will stay at a beginer level and asics of this distribution work as a vanilla kubernetes.

# Setup a VM

We will use Vagrant to spawn a VM a work with k3s.
```ruby
Vagrant.configure("2") do |config|
    config.vm.box = "ubuntu/bionic64"
    config.vm.hostname = "kubemaster"
    config.vm.network  :private_network, ip: "10.0.0.11"
end
```

Start the VM and ssh into it
```bash
vagrant up
vagrant ssh
```

# Install k3s

To install k3s from the install scripty (not recommended in production) : 
```bash
curl -sfL https://get.k3s.io | sh -
```

Wait for the script to finish then watch nodes through kubernetes until the node is Ready :
```bash
sudo kubectl get nodes -w
```

Congratulation you have now a working kubernetes cluster ready to work on.  
If you want to learn more about Kubernetes components read [the doc](https://kubernetes.io/docs/concepts/overview/components/).

# Pods

In kubernetes world, containers running are named pods, there are more than just running container and I invite you to read the [official documentation](https://kubernetes.io/docs/concepts/workloads/pods/) about it.

Let's start by starting a Job with a pod from a busybox that just print something and sleep.
Create a file `basic.yml` with content :
```yml
apiVersion: batch/v1
kind: Job
metadata:
  name: hello
spec:
  template:
    # This is the pod template
    spec:
      containers:
      - name: hello
        image: busybox
        command: ['sh', '-c', 'echo "Hello, Kubernetes!" && sleep 3600']
      restartPolicy: OnFailure
    # The pod template ends here
```

And apply it by running
```
sudo kubectl apply -f basic.yml
```

Congratulation, you have now launched a Job in Kubernetes ! Let's have a look at that.

Let's start by looking at our pods by running
```bash
sudo kubectl get pod
```
You should seen a pod named `hello-xxxxx`, that's your pod 👏.  
We could look at its logs by running.
```bash
sudo kubectl logs <pod-name>
```

If I run it I got :
```
vagrant@kubemaster:~$ sudo kubectl logs hello-7kn7h
Hello, Kubernetes!
```
And that's what we want.

# Exercice

Create a job to run a busybox printing in the log a message configured by environment variable.  
Search in the documentation of kubernetes what need to be modify based on `basic.yml` file.

<!-- 
apiVersion: batch/v1
kind: Job
metadata:
  name: hello-env
spec:
  template:
    # This is the pod template
    spec:
      containers:
      - name: hello
        image: busybox
        command: ['sh', '-c', 'echo $DEMO_GREETING && sleep 3600']
        env:
        - name: DEMO_GREETING
          value: "Hello from the environment"
      restartPolicy: OnFailure
    # The pod template ends here
     -->

# Deployments

If pods are the basic unit in kubernetes, Deployment are the main unit for pushing deployment units. 
In the previous section to run a pod we start a Job, a Job is a one of pod execution, Deployments are a deployment unit to maintain a set of pod up & running.

Let's create a simple deployment without a file :
```bash
sudo kubectl create deployment kubernetes-bootcamp --image=gcr.io/google-samples/kubernetes-bootcamp:v1
```

Let's have a look at it :
```bash
sudo kubectl get deployments
```
We see our deployment 👍

Search associated pod name.  
When you have it let's se what we deployed. There is a `kubectl port-forward` that permit to redirect network form a port on the host to a pod (or a service) let's have a look.
```
sudo kubectl port-forward <pod-name> 8080:8080 --address=0.0.0.0
```
This command redirect the traffic from your host on 8080 listening to 0.0.0.0 (as from everywhere) into the port 8080 of the pod configured.  
Now we have this command running you can open your browser on your machine and go to http://10.0.0.11:8080 and you should see waht the application print.

# Exercice

Create a `deployment.yml` file that create a deployment of image "gcr.io/google-samples/kubernetes-bootcamp:v1" like we just do.


# Services

An abstract way to expose an application running on a set of Pods as a network service.  
Take some time to read the [official documentation](https://kubernetes.io/docs/concepts/services-networking/service/)

Next, let’s list the current Services from our cluster:
```bash
sudo kubectl get services
```

Let's expose our previous deployment.  
```
sudo kubectl expose deployment/kubernetes-bootcamp --type="NodePort" --port 8080
```
We just created a service of type NodePort (aka route all the traffic on a port from the host into deployment kubernetes-bootcamp).

We cand look at our service by running :
```bash
sudo kubectl get service
```
On my test run it look like
```
sudo kubectl get service
NAME                  TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
kubernetes            ClusterIP   10.43.0.1       <none>        443/TCP          148m
kubernetes-bootcamp   NodePort    10.43.246.196   <none>        8080:30325/TCP   2m21s
```
We know have a service that expose our deployment as a port, in my example : 30325.
Open now a browser on your machine and opne http://10.0.0.11/30325 (change with your own port) and your should see the application.

Let's look at how the service choose to which pod it send the traffic to.
```
sudo kubectl describe svc kubernetes-bootcamp
```
You should see a line `Selector:                 app=kubernetes-bootcamp`. It's this line that define how a service choose the pods to send to network.
By doing a selection based on their labels. To be sure about this, look as the description of the kuberneets-bootcamp pod to see that the label `app` exist with the value `kubernetes-bootcamp`.

# Exercice

Create a `service.yml` file that create the same service that we just created.

# Scale 

Let's scale our deployment.
```bash
sudo kubectl scale deployments/kubernetes-bootcamp --replicas=4
```
If you look at the pods, now the kubernetes-bootcamp is deployed with 4 pods each running the application. 

Let's look at how the service manage this. let's run a for loop and a curl to reuse the service (don't forget to set your port):
```bash
for i in {1..10}; do curl http://localhost:30325/; done
```
We see that the service load balance between the pods the calls and adapted to our scale. This is possible du to the selection of the pods with labels.


# Update

We can edit the deployment to change all the parameters. And we could also change the image to run.  
Let's try that and what happen by watching the pods :
```bash
sudo kubectl set image deployments/kubernetes-bootcamp kubernetes-bootcamp=jocatalin/kubernetes-bootcamp:v2
sudo kubectl get pods -w
```

We see that the pods are created then the old one are terminated.  
If we run a `curl http://localhost:30325/;` we should see that the server respond a `v=2` text.

# Exercice
Define a `complete.yml` file that define a deployment with 3 replicas of `gcr.io/google-samples/kubernetes-bootcamp:v1` and a associated NodePort service (in the same file).
Deploy it check that everything run
Update the file to set the replicas to 5 and the image to `jocatalin/kubernetes-bootcamp:v2` and apply the file to see the modifications applied

# To look next

 - [Namespaces](https://kubernetes.io/docs/concepts/overview/working-with-objects/namespaces/) 
 - [DaemonSet](https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/)
 - [ConfigMaps](https://kubernetes.io/docs/concepts/configuration/configmap/)
 - [Secrets](https://kubernetes.io/docs/concepts/configuration/secret/) 
 - [Networking after NodePort](https://kubernetes.io/docs/concepts/services-networking/)