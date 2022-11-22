# Multi-Container Pod Design Patterns in Kubernetes

Kubernetes is an open-source major player container orchestration engine for automating deployments, scaling and management of containerized applications.

A pod is the basic building block of kubernetes application. Pods encapsulate containers. A pod may have one or more containers, storage, IP Addresses and some other options that govern how containers should run inside the pod.

A pod that has one container is called a single container pod. It's the most common kubernetes use case. A pod that has multiple co-related containers is called multi-container pod. You don't always need multi-container pods. When do you need to use them is the question. Some of the cases where you can may need to use them are:
- When the containers have the exact lifecycle or when the containers must run on the same node. A scenarion where you have a helper process that needs to be located and managed on the same node as the primary container.
- For simpler communication between containers in the pod. These containers can communicate through shared volumes (writing to a shared file or directory) and through inter-process communication (semaphores or shared memory)
When the containers have the exact same lifecycle, or when the containers must run on the same node. The most common scenario is that you have a helper process that needs to be located and managed on the same node as the primary container.

There are three common design patterns and use-cases for combining multiple containers into a single pod:

- [Sidecar pattern](https://github.com/NYARAS/k8s-sidecar-container-pattern)
- Adapter pattern
- Ambassador pattern

## Adapter pattern
This pattern is majorly used to standardize application output to a desired format for logging or monitoring data for aggregation.
Say we have a cluster level monitoring or logging agent that tracks the response times. Say we have a python application in our cluster that writes requests response times in the format [DATE] - [HOST] - [DURATION], while another Java application writes the same in information in [HOST] - [START_DATE] - [END_DATE].

The logging or monitoring agent we have can only accept output in the format [HOST] - [DATE] - [DURATION]. With the help of adapter pattern we can tranform these outputs int a format ours our monitoring agent understands without changing forcing the applications to write the outputs into that format.
You can have any number of adapter containers and the main application container works with them successfully..

#### Points to note on this pattern
* All the containers run or get executed parallelly and it will only work if both containers are running successfully.
* Configuring [resource limits](https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/) is very crucial for this pattern since all the containers run parallel - The sum of all the resource limits the main container plus the adapter containers

### Example
Let deploy a simple pod to understand this pattern. A pod that has main and adapter containers.

The main container is a simple busybox that writes system usage information (`top`) to a status file every five seconds.

The adapter container takes the output format of the main application (the current date and system) simplifies and reformats it to the desired format for the logging or monitoring service.

We can use the same images for both the containers but I just wanted to give an example using different images.

For you to apply this example, you need to to install [Minikube](https://minikube.sigs.k8s.io/docs/start/) as a prerequisite
Apply the manifest
```bash
 kubectl apply -f adapter-pod.yaml
```
Once the pod is running, connect to the adapter pod:
```bash
kubectl exec adapter-container-example -c main-container -it sh
```
Take a look at what the main application is writing.
```bash
cat /var/log/top.txt
```
Take a look at what the adapter has reformatted the output to
```bash
cat /var/log/status.txt
```
