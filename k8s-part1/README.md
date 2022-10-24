# Intro into Kubernetes

## Prerequisites

- Linux or Windows
- Install [Docker](https://docs.docker.com/desktop/windows/install/)
- Test that docker is correctly installed by running: `docker –-version` and `docker-compose –-version`. These should show valid versions.
- [Enable Kubernetes](https://docs.docker.com/desktop/kubernetes/) in Docker Desktop.
- Test that you can run `kubectl version`. You should get a valid output with the version. If the command is not found check the link above and add `kubectl` to your `PATH`.
- Install [VS Code](https://code.visualstudio.com/download)
- Open a terminal and cd into `k8s-part1` (the directory where this Readme is found)

## What is Kubernetes – basic concepts

Kubernetes is an open source platform for managing containerized workloads and services, that facilitates both declarative configuration and automation.

### Kubernetes cluster overview

A Kubernetes cluster consists of a set of worker machines, called [nodes](https://kubernetes.io/docs/concepts/architecture/nodes/), that run containerized applications. Every cluster has at least one worker node.

The worker node(s) host the [Pods](https://kubernetes.io/docs/concepts/workloads/pods/) that are the components of the application workload. Pods are the smallest deployable units of computing that you can create and manage in Kubernetes. A Pod is a group of one or more [containers](https://kubernetes.io/docs/concepts/containers/). Containers are created using container images.

![Kubernetes Cluster structure](./components-of-kubernetes.svg)

### Kubernetes Workload Resources

- [Deployment](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/) - This is what we use to declare how to deploy our images.
- [Pod](https://kubernetes.io/docs/concepts/workloads/pods/) - smallest deployable unit (with 1 or more containers).
- [Service](https://kubernetes.io/docs/concepts/services-networking/service/) - An abstract way to expose an application running on a set of Pods as a network service.
- [ConfigMap](https://kubernetes.io/docs/concepts/configuration/configmap/) - store non-confidential data in key-value pairs.
- [Secret](https://kubernetes.io/docs/concepts/configuration/secret/) - contains a small amount of sensitive data such as a password, a token, or a key.
- [Ingress](https://kubernetes.io/docs/concepts/services-networking/ingress/) - manages external access to the services in a cluster, typically HTTP.

## Creating and applying a deployment file in Kubernetes

[DOCS](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/)

- [nginx-deployment.yaml](./nginx-deployment.yaml) - sample deployment file
- Create the deployment: `kubectl apply -f nginx-deployment.yaml`
- Check if deployment was created: `kubectl get deployments`
- Check rollout status: `kubectl rollout status deployment/nginx-deployment`
- Check again if deployment was created: `kubectl get deployments`
- Get ReplicaSet: `kubectl get rs`
- Get the pods: `kubectl get pods --show-labels`
- Update the image in the deployment: `kubectl set image deployment/nginx-deployment nginx=nginx:1.16.1` - or edit the version of the image in the yaml file and apply it again
- Get the ReplicaSet, the previous replica has now 0 pods: `kubectl get rs`. We can also check the pods to see their status
- Get the details of the deployment: `kubectl describe deployments nginx-deployment`
- Delete a deployment: `kubectl delete deployment nginx-deployment`

## Creating and exposing a service for a deployment

[DOCS](https://kubernetes.io/docs/concepts/services-networking/service/)

The deployment creates the pods with the container(s), but we can't access them since they are not exposed. [ServiceTypes](https://kubernetes.io/docs/concepts/services-networking/service/#publishing-services-service-types)

- [nginx-service-loadBalancer.yaml](./nginx-service-loadBalancer.yaml) - sample service file
- Create the service: `kubectl apply -f nginx-service-loadBalancer.yaml`
- Access http://localhost - you should see the nginx home page

## Creating an ingress to expose services

[DOCS](https://kubernetes.io/docs/concepts/services-networking/ingress/)
Manages external access to services in the cluster. Handle hosts, SSL, load balancing.

### Prerequisite - Install controller

- Before creating an ingress, we need to setup an ingress controller in the cluster: `kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.2.0/deploy/static/provider/cloud/deploy.yaml` [Source](https://kubernetes.github.io/ingress-nginx/deploy/#quick-start).
- Check that the controller pods were started: `kubectl get pods --namespace=ingress-nginx`
- Creating the controller is a one time only operation. You don't have to do it everytime you want to define an ingress.
- Check what resources were created for the controller: `kubectl get all --namespace=ingress-nginx`

### Create an ingress for our service(s)

- Modify our service to use type=ClusterIP (exposed only in the cluster): `kubectl apply -f nginx-service-clusterIP.yaml`
- Create the ingress for our service: `kubectl apply -f nginx-ingress.yaml`
- Get the ingress: `kubectl get ingress nginx-ingress`
- Access http://localhost - you should see the nginx home page

### References

- https://kubernetes.io/docs/concepts/overview/
- https://kubernetes.io/docs/concepts/overview/components/
- https://kubernetes.io/docs/concepts/
