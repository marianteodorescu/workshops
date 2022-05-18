# Kubernetes for Production

# Prerequisites

- See [Prerequisites section from part 1](../k8s-part1/README.md).
- Open a terminal and cd into `k8s-part2` (the directory where this Readme is found).

## Vertical scaling - Pod/container resources

[DOCS](https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/)
2 types of resource management configuration:

- request: the minimum allocated to the container
- limit: maximum allowed

Most important resources:

- memory - measured in bytes. Use prefixes (E, P, T, G, M, k) for larger quantities: 400M, 1.5GB.
- CPU - measured in CPU units (1 unit = 1 core). You can specify fractional values: 0.5, 0.1 = 100m (100 millicpu).

We can specify `request` and `limit` for each container, see below the path in the `yaml` files:

- spec.containers[].resources.limits.cpu
- spec.containers[].resources.limits.memory
- spec.containers[].resources.requests.cpu
- spec.containers[].resources.requests.memory

### Errors caused by insufficient resources

- FailedScheduling - kubernetes cannot find a node where the Pod can fit.

  - Create deployment: `kubectl apply -f high_cpu_deployment.yaml` [file](./high_cpu_deployment.yaml)
  - Get the pods: `kubectl get pods`
  - Check the pod: `kubectl describe pod <pod_name>`. Notice the `Events` section and the message. Also here you can check how many resources the pod requested. Maybe we had a typo and asked for too much CPU :).
  - We can also check the nodes to see user resources. Get nodes: `kubectl get nodes`, describe a node: `kubectl describe node <node_name>`.
  - Change the previous deployment to a reasonable value for CPU (100m), deploy it, check pod status then node status.

- Container terminated - the container tried to get more resources than the limits - `OOMKilled` in the Events section.

## Horizontal Pod Autoscaling

### Prerequisite

For metrics to run locally we have to enable metrics server in kubernetes [Source](https://dev.to/docker/enable-kubernetes-metrics-server-on-docker-desktop-5434). Run the command below to install it.

- Run `kubectl apply -f metrics-server.yaml`
- Run `kubectl top node` & `kubectl top pod -A` to resource usage

### Create Deployment and HPA

[DOCS](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale/)
Automatically update the workload resources to match demand. Load increases -> create more Pods.

- Create deployment and service: `kubectl apply -f low_mem_deployment.yaml` [file](./low_mem_deployment.yaml)
- Create hpa: `kubectl apply -f low_mem_hpa.yaml`. [file](./low_mem_hpa.yaml)
  [More details and metrics to scale](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale-walkthrough/#autoscaling-on-multiple-metrics-and-custom-metrics).
- Run `kubectl get hpa low-mem --watch` to see how the scaler is working and how the number of pods is increased based on the load.
- Run in another terminal `kubectl run -i --tty load-generator --rm --image=busybox:1.28 --restart=Never -- /bin/sh -c "while sleep 0.01; do wget -q -O- http://low-mem-service; done"` to generate load on our service.
- Stop the load generator and the scaler should drop the number of pods (it will take a while).

## Create and apply a cronjob

[DOCS](https://kubernetes.io/docs/concepts/workloads/controllers/cron-jobs/)
Perform scheduled actions (backups, reports, emails etc).

- Run `kubectl apply -f cronjob.yaml` [file](./cronjob.yaml)
- Get the list of pods `kubectl get pods` and check the logs for the pod created by the cronjob `kubectl logs <pod_name>`.
- Run `kubectl get pods` again to check that the cronjob created another pod after a minute.
- Run `kubectl delete cronjob hello` to clean up.

## Connect to a pod and check logs

- Run `kubectl apply -f low_mem_deployment.yaml`
- Get the list of pods
- Connect to a pod `kubectl exec -it <pod_name> -- bash`. You can now run commands in that pod: list env variables (`env`), run a script etc.
- Run directly a command `kubectl exec -it <pod_name> -- /bin/sh -c "echo test"`

## Jobs

[DOCS](https://kubernetes.io/docs/concepts/workloads/controllers/job/)
Used for one off jobs. It creates one or more Pods and will continue to retry execution of the Pods until a specified number of them successfully terminates.
Also useful to run specific commands which need more resources than a normal pod created by your deployment.

- Run `kubectl apply -f job.yaml`. It has a `sleep 3600` command which allows us to connect to the pod and run what we need inside it.
- Connect to the pod created by the job
- !!! Always clean-up after you have run jobs. `kubectl delete job low-mem-job`

## Using labels for grouping and interrogating resources

[DOCS](https://kubernetes.io/docs/concepts/overview/working-with-objects/labels/)

Used for grouping/identifying/querying related resources in an organization.

- Run `kubectl apply -f low_mem_deployment.yaml`
- Get the list of pods/services/deployments etc for a specific label: `kubectl get pods -l app=low-mem`
- Also used for linking resources. For example the way a service is attached to pods using the `selector` in [low_mem_deployment.yaml](./low_mem_deployment.yaml)

## Contexts

[DOCS](https://kubernetes.io/docs/tasks/access-application-cluster/configure-access-multiple-clusters/)

- Contexts are used to store connection details to a kubernetes cluster.
- View current context: `kubectl config current-context`
- View all contexts: `kubectl config get-contexts`
- Switch context: `kubectl config use-context docker-desktop`

## Namespaces

[DOCS](https://kubernetes.io/docs/concepts/overview/working-with-objects/namespaces/)
Namespaces are intended for use in clusters with many users spread across multiple teams, or projects. Use namespaces to isolate resources in the cluster. Cluster admins create namespace and give access to users [docs](https://kubernetes.io/docs/tasks/administer-cluster/namespaces/).

- Create namespace: `kubectl apply -f test_namespace.yaml`
- View namespaces: `kubectl get namespace`
- Set the namespace for a request:
  - Create a deployment in the namespace: `kubectl apply -f low_mem_deployment.yaml --namespace=test` . <b>!!! Bad practice for production. When creating resources we have to specify the namespace in the metadata for that resource.</b>.
  - View all the pods in the namespace: `kubectl get pods --namespace=test`
- Set namespace preference (for all requests in the current context): `kubectl config set-context --current --namespace=test`

## Context helpers

- https://github.com/jonmosco/kube-ps1 - for bash/zsh - display the k8s context and namespace.
- https://github.com/ahmetb/kubectx - manage contexts and namespaces easier.

## Configmaps

https://kubernetes.io/docs/concepts/configuration/configmap/

## Secrets

https://kubernetes.io/docs/tasks/inject-data-application/distribute-credentials-secure/

## Cheatsheeet

https://kubernetes.io/docs/reference/kubectl/cheatsheet/
