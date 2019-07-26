# General info
- Minikube is a lightweight Kubernetes implementation that creates a VM on your local machine and deploys a simple cluster containing only one node. 
- kubectl is the client of kubernetes.
- kubeadm helps to setup the kubernetes cluster.
- kubernetes architecture consists of a master node which is used for scheduling and, worker nodes to which the tasks are scheduled.

# Why are pods better than containers?
	Pods are better than individual containers to have failure management.

## What is minikube used for?
	minikube is used for single node cluster

## What is kubeadm used for?
	kubeadm can used for single or multi node cluster

## How to execute a command inside a pod?
	kubectl exec <pod name> -- <command>
	Double dash is required to indicate that the following isn't a part of kubectl command.

## How to run pods without writing a yaml?
	kubectl run busybee --image=busybox --generator=run-pod/v1 --command -- sleep 999999

## What to consider when running two containers in the same pod?
- Scaling
- Utilization of resources

## How to create a CI/CD pipeline?
	Store yaml/json descriptors in a VCS

## How to get the yaml description of a running pod?
	kubectl get po kubia-zxzij -o yaml	

## How to write a yaml descriptor?
	For any kubernetes API object:
- kubernetes API version being used
- metadata - includes the name, namespace, labels, and other information aboout the pod.
- Spec contains the actual description of the pod’s contents, such as the pod’s containers, volumes, and other data.
- Status(Not meant to  be written but is a part of an API object): contains the current information about the running pod, such as   what condition the pod is in, the description and status of each container, and the pod’s internal IP and other basic info.

## How does a sample yaml descriptor look like?
	Use kubectl explain pods
	kubectl explain pod.spec
	apiVersion: v1
	kind: Pod
	metadata:
		name: kubia-manual
	spec:
		containers:
			- image: luksa/kubia
		name: kubia
		ports:
			- containerPort: 8080
			  protocol: TCP

## How to create any pod/resource from yaml descriptor?
	kubectl create -f kubia-manual.yaml

## How to list running pods?
	kubectl get pods

## How to view logs of a pod?
	kubectl logs kubia-manual
	kubectl logs kubia-manual -c kubia ## for a particular container

## How to forward a local network port to a port in the pod?
	kubectl port-forward kubia-manual 8888:8080

## How to create a replication controller?
	kubectl run kubia --image=luksa/kubia --port=8080 --generator=run/v1

## How to expose the rc?
	kubectl expose rc kubia --type=LoadBalancer --name kubia-http ## this creates a service object

## What is the advantage of using a service?
 services are used to have a static ip address.

## How to scale rc?
	kubectl scale rc kubia --replicas=3

# Labels

## How to organise pods and other kubernetes objects?
	By using labels
		- app: which specifies which app, component, or microservice the pod belongs to
		- rel(release) : which shows whether the application running in the pod is a stable, beta or a canary release.

## Where to mention the labels?
	In the metadata section 

## 	How to look at the labels?
	kubectl get pods --show-labels
	If you’re only interested in certain labels, you can specify them with the -L switch:
	kubectl get po -L creation_method,env

## How to add a label to existing pod?
	kubectl label command
	ex: kubectl label po kubia-manual creation_method=manual

## How to change the label?
	kubectl label po kubia-manual-v2 env=debug --overwrite

## How to delete a label?
	kubectl label po <pod name> <label name>-

## How to add labels to nodes?
	kubectl label node gke-kubia-85f6-node-0rrx gpu=true
	kubectl get nodes -l gpu=true

## How to schedule pods to specific nodes?
	Add this line under spec of the yaml.
	spec:
		nodeSelector:
			gpu: "true"
	
	This schedules the pods to nodes labeled to gpu=true. 

# Namespaces 

## What is the need for namespaces?
	Namespaces enable you to separate resources that don’t belong together into non-overlapping groups. Besides isolating resources, namespaces are also used for allowing only certain users
	access to particular resources and even for limiting the amount of computational resources available to individual users

## How to create a namespace from a yaml file?
	apiVersion: v1
	kind: Namespace
	metadata:
		name: custom-namespace

## How to run a pod in other namespace?
	kubectl create -f kubia-manual.yaml -n custom-namespace

## How to delete  a pod?
	kubectl delete po kubia-gpu

## How to delete all resources in a namespace?
	kubectl delete all --all

# Liveness probes

## Why to use liveness probes?
	To check whether the pods are healthy from the outside because checking internally isn't reliable as app could be in an infinite loop or deadlock. 

## What are the different liveness probes?
- HTTP requests- Other than 2xx or 3xx , all other responses are considered unsuccesfull. Even timeouts are considered. 
- TCP socket 
- Exec probe executes a command and check for status code of 0(successful)

## How to see a sample yaml descriptor for liveness probes?
	kubectl explain pod.spec.containers.livenessProbe

## How to obtain the logs of the crashed container?
	kubectl logs pod-health

## How to know why a container failed?
	kubectl desribe po pod-health 

## Why can't we create a custom logic to check liveness of a probe?
	Liveness probes are meant to be lightweight. Having a custom logic will require more CPU which gets added to the container's CPU limit.

# Replication Controller

## What's the job of a replication controller?
	A ReplicationController constantly monitors the list of running pods and makes sure
	the actual number of pods of a particular label always matches the desired number.

## What are the three parts of a replication controller?
- Label Selector
- Replica Count
- Pod template

## What are the advantages of a Replication Controller?
- It makes sure a pod (or multiple pod replicas) is always running by starting a
	new pod when an existing one goes missing.
- When a cluster node fails, it creates replacement replicas for all the pods that
	were running on the failed node (those that were under the Replication-Controller’s control).
- It enables easy horizontal scaling of pods—both manual and automatic

# ReplicaSet

## Why is ReplicaSet better than ReplicationController?
	ReplicaSet lets us set multiple labels and also lets us specify just the label key and omit the value. 

## Which api group and version are ReplicaSet a part of ?
	apps/v1betav2

## How does ReplicaSet offer better control?
	In the yaml descriptor, under selector, we can use matchExpressions. Ex:
	selector:
		matchExpressions:
		- key: app
		  operator: In
          values:
        - kubia

## What are the different kinds of operators in ReplicaSet.matchExpressions?
- In — Label’s value must match one of the specified values
- NotIn— Label’s value must not match any of the specified values .
- Exists— Pod must include a label with the specified key
- DoesNotExist— Pod must not include a label with the specified key.

# DaemonSet

## Why do we need DaemonSets?
	There are cases where every node should have a particular pod running. For instance, log collectors and resource monitoring.

## How to apply DaemonSets to run pods on certain nodes?
	Using the nodeSelector property.

## Do DaemonSets rely on Scheduler?
	No, system services need to run on nodes even if they're marked as unschedulable.

# Jobs

## Why do we need Job ?
	To run a process a finite number of times. 

## Which api are jobs in?
	batch/v1

## Why should be explicitly mentioned in the Job's yaml descriptor?
	Under  Job.spec.template.spec, restartPolicy: OnFailure should be mentioned. The other option for restartPolicy is Never. 

## How do you run a pod sequentially for a specific number of times?
	Under Job.spec, specify completions: <number of times>

## How do you specify a job to run pods parallely?
	Under Job.spec , specify parallelism: <number of pods you want to run parallely>

## How to scale a job(parallely)?
	Like rc/rs, jobs can be scaled as well. For ex: kubectl scale job multi-completion-batch-job --replicas 3

# CronJob

## Which api do CronJob belong to?
	batch/v1beta1

# Services

## Why do we need servicees?
- Pods are ephemeral 
- IP address of a Pod isn't known until the pod is run on the node.
- Service provides one static IP address

## What is sessionAffinity and why to use it?
	By default, a request to a service IP is redirected to a random pod which is part of the service.
	By setting sessionAffinity, a clientIP can be associated with a consistent pod.
	sessionAffinity has to be declared in the yaml descriptor under Service.spec. 
	It can either be none or ClientIP

## Can a service expose multiple ports?
	Yes. Each port in Service.spec is supposed to have a unique name.

## Why do we name ports?
	To change the port number without disrupting the services.

## How to discover services with environment variables?
	Each pod contains the environment variable which has all the service IP and ports.
	Ex: kubectl exec <pod name> env 

## Is it possible to discover services through DNS?
	Yes, kube-dns is a pod that runs under kube-system namespace which knows all the services running on the system and every 
	pod created is configured to use this DNS.

## How to discover services through kube-dns?
	Each service is associated with a FQDN. Ex: backend-database.default.svc.cluster.local
	backend-database corresponds to the service name, default stands for the namespace the service is defined in, and svc.cluster.local is a configurable cluster domain suffix used in all cluster local service names. 

## What are service endpoints?
	Although the pod selector is defined in the service spec, it’s not used directly when
	redirecting incoming connections. Instead, the selector is used to build a list of IPs
	and ports, which is then stored in the Endpoints resource


## How to expose services externally?
- Setting the service type to NodePort
- Setting the service type to loadbalancer
- Creating an ingress resource


## What is the difference between readiness probes and liveness probes?
	If a readiness probe fails, it doesn't restart the pod. It is just removed from the endpoints resource.

## What is a headless service and why do we need it?
	Headless service is a service which has no Cluster Ip associated with it. The advantage of this is that when a DNS lookup
	is performed on this IP, all the endpoints associated with the service are listed rather than the Cluster IP.


# Volumes

## Why do we need volumes?
	To share data among containers

## What are the types of volumes in kubernetes?
- emptyDir —A simple empty directory used for storing transient data.
- hostPath —Used for mounting directories from the worker node’s filesystem into the pod.
- gitRepo —A volume initialized by checking out the contents of a Git repository.
- nfs —An NFS share mounted into the pod.
- gcePersistentDisk (Google Compute Engine Persistent Disk), awsElasticBlockStore (Amazon Web Services Elastic Block Store Volume),  azureDisk(Microsoft Azure Disk Volume)
- cinder , cephfs , iscsi , flocker , glusterfs , quobyte , rbd , flexVolume , vsphere-Volume , photonPersistentDisk.scaleIO—Used    for mounting other types of network storage.
- configMap , secret , downwardAPI —Special types of volumes used to expose certain Kubernetes resources and cluster information     to the pod.
- persistentVolumeClaim

## Can emptyDir use RAM instead of disk?
	Yes, by specifing medium: Memory under emptyDir.

## Why is emptyDir used?
	To share data between containers of the same pod. When a pod goes down, the emptyDir data goes down as well.

## What are gitRepo volumes?
	It's an empty volume in which a git repo is cloned.

## How to keep the container in sync with the git repo?
	Using a sidecar container which has git sync in it.

## Why it might be better to use git sync sidecar containers?
	It has functionality to provide access to private repos as well.

## What is hostPath Volume?
	A hostPath volume points to a specific file or directory on the node’s filesystem. They are persistenet storage.

## Why to use Persistent volumes?
	Developers need not know about the underlying infrastructure. Administrators can just create a Persistent volume which 
	developers can claim when they need it. These are cluster level resources.

## What is the procedure with persitent volumes?
	Persistent volumes are created by administrators.
	Persistent volume claims are created by developers
	Pods are deployed which utilise the PVC.

## What are the access modes available?
- RWO: ReadWriteOnce- only a single node can mount for reading and writing
- ROX : Read only Many- many nodes can mount the volume for reading
- RWX : Many nodes can mount the volume for reading and writing

## How to recycle persistent volumes?
	By default, the retain policy of a PV is to retain. So when the PVC is deleted, the PV cannot be deleted. This gives
	the admin to decide what to do with the data before another pod is made to utilise the same volume.
	There are two other Retain Policies:
- Delete
- Recyle
	Delete policy deletes the underlying storage and makes it available for the next claim
	Recycle policy makes the data available on the volume to next pod without deleting the underlying storage.

# ConfigMap and Secrets

## How are apps configured?
- Passing CLI arguments to containers
- Setting custom environment variables for each container
- Mounting configuration files into containers through a special type of volume

## What is the kubernetes equivalent of docker's ENTRYPOINT and CMD?
	ENTRYPOINT equivalent is command
	CMD equivalent is args

## What is the drawback of hardcoding environment variables?
	Hardcoding environment variables in the pod definition would result in having seperate pod definitons for development and 
	production pods. 

## What is ConfigMap?
	It is a kubernetes object which is a map containing key-value pairs where values can be short literals or config files.

## What is the advantage of using ConfigMap?
	Reusing the same pod definition in multiple environments before the pod is deployed.

## How to create  a configmap from CLI?
	kubectl create configmap <name of the map> --from-literal=<key>=<value>
	For multiple entries, use "--from-literal " as many times as needed. 
	For a file, use "--from-file=<filename of the config>". In this case, file name is the key and the value is the contents.
	To have a custom key name use "--from-file=<custom key name>=<file name>". 

## Why is mounting a filesystem to /etc a bad idea?
	Mounting a filesystem to a particular non-empty directory makes the contents of the directory inaccessible as long as the filesystem is mounted. If a filesystem is mounted upon /etc , it would break the container.

## How to add files from configmap without making the existing files inaccessible?
	By using the subpath property of volumeMounts.

## What is the default permissions of a configmap volume?
	644. It can be changed under the property of defaultMode. ex: defaultMode:"660"

## What is the drawback of using environment varibales and CLI arguements?
	Inability to update them when the container is running.
	Using a volume, updates the container without having to restart.

# Secrets

## How are secrets better than configmap for sensitive information?
	Secrets are only exposed to those nodes that run the pods that need access to the secrets.
	They're stored in memory(tmpfs).

##	How to generate private key and certificate?
	openssl genrsa -out https.key 2048
	openssl req -new -x509 -key https.key -out https.cert -days 3650 

## Why are the content of secret's entries base64 encoded?
	Base64 encoding allows binary data to be formatted in yaml or json.

## Does the app need to decode the secret's entries?
	No, the app can use environment variables or read files from secret volume without the need to decode it.

## Why is it a bad idea to expose secrets with environment variables?
	Apps dump environment variables in error reports/logs . If the app runs a third party binary, secrets might get compromised.

# Deployments

## What are the two ways to update the pods to a newer application version?
- Delete existing version pods and replace that with newer version. Drawback is short amount of downtime.
- Start new ones and, once they're up, delete the old ones. Drawback is two versions running simultaneously might affect other components.

## How to change from old version to new at once?
	By performing a blue-green deployement. In this, a service exposes pods of previous version until the new version pods are up. Later, the replicatiion controller of previous version can be deleted.
	Drawback: This would require a lot of hardware resources. To overcome this, we can use rolling updates.

## How to perform a rolling update?
	kubectl rolling-update kubia-v1 kubia-v2 --image=luksa/kubia:v2

## What is the drawback of rolling update?
	Rolling update uses the REST API to perform the update without letting scheduler take care of it. If network connectivity 
	was lost with kubectl, the update wouldn't be succesfull. 

## What are the deployement strategies?
- Rolling Update
- Recreate 

## How to change or add a particular attribute on existing resources?
	kubectl patch deployment kubia -p '{"spec": {"minReadySeconds": 10}}'

## How to trigger the rolling update from a deployment resource?
	kubectl set image deployment kubia nodejs=luksa/kubia:v2

## How to see the history of the updates?
	kubectl rollout history deployment kubia

## How to undo the previous update?
	kubectl rollout undo deployment kubia --to-revision=1

## How to pause/resume the update?
	kubectl rollout pause/resume deployment kubia
 



# Questions
- How to share the pid namespace among containers in the same pod?
- Are sniffers used in post exploitation of clusters?


## Resources
- https://kubernetes.io/docs/tutorials/kubernetes-basics/
- https://matthewpalmer.net/kubernetes-app-developer/articles/kubernetes-apiversion-definition-guide.html
- https://stackoverflow.com/questions/47369351/kubectl-apply-vs-kubectl-create
- https://kubernetes.io/docs/concepts/services-networking/service/
- vs code: Extension to use : "ext install ipedrazas.kubernetes-snippets"