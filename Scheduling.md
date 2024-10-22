# Manual Scheduling

What you do when you do not have scheduler in cluster?
-> You probably dont want to rely on built-in scheduler, and instead schedule the pod yourself.

How scheduler works in the backend?
-> Every pod has a field called nodeName, which is by default not set. We dont typically specify this field, when we create the pod manifest file, infact kubernetes adds it automatically.

-> The Scheduler goes through all the pods and looks for those that do not have this property set. Those are candidates for scheduling. It then identifies right node for the pod by running the scheduling algorithm. 

-> Once identified, it schedules the pod on the node by setting the nodeName property to the name of the node by creating a binding object. So, if there is no scheduler to monitor and schedule nodes, What happens? 

-> The Pods continue to be in pending state. So what can be done? You can manually assign pods to nodes yourself. Without scheduler, easiest way to schedule pod is to simply set nodeName field to name of node in your pod specification file while creating the pod. 

-> You can only specify the node name at creation time. What if pod is already created and you want to assign pod to a node? Kubernetes will not allow you to modify the nodeName property of a pod. So, another way to assign a node to a existing pod, is to create a binding object and send a POST request to the pod's binding API. Thus, mimicking what the actual scheduler does. In binding object, you specify a target node with the name of the node then send a POST request to the pod's binding API with the dataset to the binding object in a JSON format.SO, you must convert YAML file into its equivalent JSON form.

To check if scheduler is present do -> k get pods --namespace kube-system
if scheduler, is not present, container will always be in pending state in that case you have to add nodeName field.

![image](https://github.com/user-attachments/assets/a0da8024-332d-46bb-8df0-c2f677fee3a7)

Above is the example of how to manually schedule.

k replace --force -f nginx.yaml    -> This will delete and reapply in single command

k get pods --watch    (To watch it continously)

You cannot move running pod from one node to other. You have to delete it and recreate.

The Kubernetes scheduler is a crucial component of the Kubernetes control plane. Its primary role is to assign newly created pods to nodes within the cluster. Here’s a high-level overview of how it works:

Queueing: When a new pod is created, it is added to a scheduling queue. The scheduler continuously monitors this queue for pods that need to be assigned to nodes.

Filtering: The scheduler first filters out nodes that do not meet the pod’s requirements. This includes checking for resource availability (CPU, memory), node taints and tolerations, and other constraints specified in the pod’s configuration.

Scoring: After filtering, the scheduler scores the remaining feasible nodes. This scoring is based on various factors such as resource utilization, affinity/anti-affinity rules, and data locality. The node with the highest score is selected.

Binding: Once a node is selected, the scheduler binds the pod to that node by updating the pod’s specification in the Kubernetes API server. This process is known as binding.

Plugins and Extensibility: Kubernetes also supports a scheduling framework that allows for custom plugins. These plugins can be used to extend the scheduler’s functionality, enabling more complex scheduling behaviors and policies.

# Labels and Selectors

Labels and selectords are standard method to group things together.

Labels are properties attached to each item. So, you add properties to each item for their class, kind and color. Selectors help you filter these items. When you say class = Mammal & Color = Green, it will filter
and give you result. 

We have created lot of different types of object in kubernetes like pods, services, replica sets, deployments, etc. Overtime you could end up having 100s or 1000s of objects in the cluster. Then you need a way to
filter and view different object by categories such as to group objects by their type, application, functionality, etc.

For each object, attach labels as per your needs, like app, function, etc. Then while selecting, specify condition to filter object. 

In Pod definition file, under metadata create a section called labels.

Command -> kubectl get pods --selector app=App1

Command -> Kubectl get all --selector env=prod

Command -> kubectl get all --selector env=prod,bu=finance,tier=frontend    (This is for multiple labels)

![image](https://github.com/user-attachments/assets/4b243f91-2660-4ac3-91d5-0f4fbe477684)

In above image, matchLabels under spec should match with labels under template section.


# Annotations

While labels and Selectors are used to group and select objects, annotations are used to record other details for informatory purpose. For eg, tool details like name, version, contact details ,email id, etc.

# Taints and Tolerations

## Taints

This is about Pod to Node relationship, and how you can restrict what pods are placed on what nodes. The concept of taints and tolerations, can be a bit confusing. 

For eg -> Person is a node and bug is a pod. Taints and Tolerations has nothing to do with security or intusion on the cluster. It is actually used to set restrictions on what pods can be scheduled on a node.

If there are three worker nodes - node a, b, c and we also have set of pods 1, 2, 3, 4. When Pods are created, kubernetes scheduler tries to place these pods on the available worker nodes. As of now there are 
no restrictions or limitations. That is why scheduler places pods across all of the nodes to balance them out equally.

Now, let us assume, we have dedicated resources on node A for particular use case or application. So we would like only those pods that belong to this application to be placed on Node A. First, we prevent all 
pods from being placed on the node by placing a taint on that node. for eg Taint=blue by doing this no unwanted pods are going to be placed on this node. Other half of job is to enable certain pods to be placed 
on Node A. 

For this, we must specify which pods are tolerant to this particular taint.Add toleration to any Pod you want to.

kubectl taint nodes node-name key=value:taint-effect

For eg - If you would like to dedicate the node to pods in application blue, then the key value pair would be app=blue

What happens to pods that Do Not Tolerate this taint? -> There are three taint effects such as 

No Schedule - which means pods will not be scheduled on the node.

Prefer No Schedule - Which means the system will try to avoid placing a pod on the node, but that is not guarenteed.

No Execute - Which means new pods will node be scheduled on the node and existing pods on the node, if any, will be evicted if they do not tolerate the taint. These pods may have been scheduled on the node
before the taint was applied to the node.

kubectl taint nodes node1 app=blue:NoSchedule

In Kubernetes, taints and tolerations are used to control which nodes can accept which pods. Taints are applied to nodes, and tolerations are applied to pods. Here are the three taint effects:

NoSchedule: This effect ensures that no new pods that do not tolerate the taint will be scheduled on the node. However, existing pods on the node are not affected.

PreferNoSchedule: This is a softer version where the system tries to avoid placing a pod that does not tolerate the taint on the node, but it is not a strict rule.

NoExecute: This effect is the strongest. It evicts any existing pods that do not tolerate the taint and prevents any new pods that do not tolerate the taint from being scheduled on the node.

When a pod gets evicted from a node due to a taint with the NoExecute effect, it means the pod is forcibly removed from that node. The Kubernetes scheduler will then try to find another suitable node for the pod to run on, provided there is a node that meets the pod’s requirements and tolerates its taints.

If no suitable node is found, the pod will remain in a pending state until a suitable node becomes available. This ensures that the pod can eventually be scheduled and run, but it might experience some downtime during the eviction and rescheduling process.

## Tolerations

To add tolerations to any pod you need to add as it is shown in below image. It should be under spec section.

Kubectl taint nodes node1 app=blue:NoSchedule

![image](https://github.com/user-attachments/assets/c5727a37-d94f-46e1-8c50-62f311534366)

Important - Taint and Tolerations does not tell the pod to go to a particular node. Instead, it tells the node to only accept pods with certain tolerations. If your requirement is to restrict a pod to a certain nodes, it is achieved through another concept called as node affinity.

Scheduler does not schedule any pods on the master node. Why is that? -> When the kubernetes cluster is first set up, a taint is set on the master node automatically that prevents any pods from being scheduled on this node. 

kubectl describe node kubemaster | grep Taint   ( Will show taint on master node) 

k taint nodes node01 spray=mortein:NoSchedule

kubectl run pod-01 --image=nginx

kubectl run my-pod --image=nginx --dry-run=client -o yaml > my-pod.yaml

![image](https://github.com/user-attachments/assets/f8905dec-2026-436c-a892-238fdf33962c)

To remove taint -> kubectl taint nodes controlplane node-role.kubernetes.io/control-plane:NoSchedule-     (Its minus sign at end)

# Node Selectors

If you have three node cluster, out of which two have lower hardware resources, one of them is a larger node configured with higher resources. You have different kind of workload running in the cluster. You would like to
dedicate the data processing workloads that require higher horsepower to the larger node as that is the only node that will not run out of resource in case the job demands extra resources. 

However, in the current default setup, any pods can go to any nodes. So, Po C for eg can end up on nodes two or three, which is not desired. To solve this, we can set a limitation on the pods, so that they only run on 
particular nodes.

There are two ways to do this - 

1. Using Node Selectors, which is the simple and easier method. For this, we look at the pod definition file we created earlier. This file has simple definition to create a pod with a data processing image. To limit this
   pod to run on the larger node, we add a new property called node selector to the spec section. specify the size as Large. But from where did this size Large come from and how does kubernetes know which is large node?

   --------
   
   apiVersion:
   
   kind: Pod

   metadata:

     name: myapp-pod

   spec:

     containers:
     
     - name: data-processor
     
       image: data-processor
     
     nodeSelector:

       size: Large
   
    ------------------

The key value pair of size and large are in fact labels assigned to the nodes. The scheduler uses these labels to match and identify the right node to place pods on. 

To use labels in a node selector you must have first labeled your nodes prior to creating this pod. 

How to label a node?

kubectl label nodes node-01 size=Large

Node selectors served our purpose, but it has limitations. We used a single label and selector to achieve our goal here. But what if our requirement is much more complex?

For eg - if we want to say place a pod on a large or medium node, or something like place the pod on any node that are not small. You cannot achieve this using Node Selectors. 

For this, node affinity and anit-affinity features were introduced

# Node Affinity

The primary purpose of node affinity feature is to ensure that pods are hosted on particular nodes, for eg to ensure that large data processing pod ends up on node-01.

You cannot use advanced expressions like Or / Not with Node Selectors.

The node affinity feature, provides us with advanced capabilities to limit pod placement on specific nodes. With great power comes great complexity, it looks like

-------------------------------------------------

affinity:

  nodeAffinity:

    requiredDuringSchedulingIgnoreDuringExecution:

      nodeSelectorTerms:

      - matchExpressions:

        - key: size

          operator: In       (NotIn for small can also be one use case)      (Exist - will check if label size exists)

          values:

          - Large

          - Medium   (You can add remove as per need)

------------------------------------------------------

WHat if node affinity could not match a node with a given expression?

In case - where there are no nodes with the label called size.

Say we had the labels and the pods are scheduled. What if someone changes the label on the node at the future point in time?

Will the pod continue to stay on the node?

The Type of node affinity defines the behavior of the scheduler with respect to node affinity and the stages in the life cycle of the pod.

------------------------------------------------------------

There are currently two types of node affinity available -

Available -

requiredDuringSchedulingIgnoredDuringExecution

preferredDuringSchedulingIgnoredDuringExecution

Planned -

requiredDuringSchedulingRequiredDuringExecution

----------------------------------------------------------

There are two states in the lifecycle of a Pod, when considering node affinity, during scheduling and during execution.

During Scheduling is the state where a pod does not exist and si created for first time. We have no doubt that when a pod is first created, the affinity rules specified are considered to place the pods on right nodes.

Now, what if nodes with matching labels are not available? For eg - if we forgot to label the node as Large. That is where type of node affinity use comes into play. 

If you choose requiredDuringScheduling, the scheduler will mandate that the pod be placed on a node with a given affinity rule. If it cannot find one, the pod will not be scheduled. This tpe will be used in cases, where the placement of the pod is crucial. If a matching node does not exist, the pod will not be scheduled. 

But lets say, if pod placement is less important, than running the workload itself. In that case, you could set it to preferredDuringScheduling and in cases where a matching node is not found, the scheduler will simply ignore node affinity rules and place the pod on any available node. 

This is a way of telling scheduler, you tried finding matching, if you dont find anything just place it anywhere. 

What if admin removes the label on node which has Large as a label, what will happen to the pod running on the node?

As you can see, there are two types of node affinity available todays has this value set to ignored, which means pods will continue to run and any changes in node affinity will not impact them once they are scheduled. 

The new types expected in future, only have a difference in the during execution phase. In this case, if label is removed from node, pod will also be deleted.

to check labels on nodes  -> k describe nodes node-01

to add labels -> k label nodes node-01 color=blue

kubectl create deployment blue --image=nginx --replicas=3

# Node Affinity vs Taints & Tolerations

For eg - If we have three pods Red, Green & Blue. Also, we have three nodes Red, Green & Blue. We want Red to go in Red Node and likewise. We dont want any other pods to be placed on our nodes and also I dont want to place our pods to any nodes instead in their own color node. 

Using Taints & Tolerations - We apply a taint to the nodes marking them with their colors blue, red and green and, we then set a toleration on the Pods to tolerate the respective colors. When the pods are now created, the nodes ensure they only accept the pods with the right toleration. So, green pods ends up on green node and likewise for other colors. 

However, taints and tolerations does not guarentee that the pods will only prefer these nodes. So, Red pod can be placed on node which has no taint and tolerations set. This is not desired. 

What if we use node affinity for this problem, with node affinity, we first label the nodes with their respective colors blue, red and green. We then set node selectors on the pods, to tie the pods to nodes. As such, the pods end up on the right nodes. However, that does not guarentee, that other pods are not placed on these nodes. In this case, there is a chance that one of the other pods may end up on our nodes. This is not something we desire.

As such a combination of taints and tolerations, and node affinity rules can be used together to completely dedicate nodes for specific pods. 

We first use taints and tolerations to prevent other pods from being placed on our nodes, and then we use node affinity to prevent our pods from being placed on their nodes. 

# Resource Requirements and Limits

Each node has a set of CPU and memory resources available. Now every pod requires a set of resources to run. For eg if pod required 2 CPU and 1 memory unit. Whenever pod is placed on a node, it consumes the resources available on that node. 

It is kube scheduler which decides which node a pod goes to. Scheduler takes into consideration, the amount of resources required by a pod and those available on the nodes and idenitifies the best node to place a pod on. 

If nodes have no sufficient resources available, the scheduler avoids placing the pods on those nodes and instead places the pod on one where sufficient resources are available. If there is no sufficent resources available on any nodes, then scheduler holds back scheduling the pod and you will see pod is in a pending state. If you look at the events, using kubectl describe pod command, you will see reason there is an insufficient CPU.

What are the blocks and what their values in resource requirement of each pod. Now you can specify the amount of CPU and memory required for a pod when creating one. For eg it could be one CPU and one gibibyte of memory and this is known as resource request for a container. So, minimum amount of CPU and memory requested by the container. 

So, when the scheduler tries to place the pod on a node, it uses these numbers to identify a node which has sufficient amount of resources available. So, to do this in the sample pod-definition file, you need to add a section called resources under which add requests and specify the new values for memory and CPU usage.

----------------------------------------------------------

spec:

   containers:

   - name: sample-webapp
     
     image: sample-webapp

     ports:

       - containerPort: 8080
    
     resources:

       requests:

         memory: "4Gi"

         cpu = 2

---------------------------------------------------

When a scheduler get a request to place this pod, it looks for a node that has this amount of resources available. When pods gets placed, it gets a guarenteed amount of resources available for it.

1 CPU is equivalent to - 1 AWS vCPU, 1 GCP core, 1 Azure Core, 1 HyperThread.

--------------------------------------------------

limits:

  memory: "2Gi"

  cpu: 2 

  ------------------------------

  What happens when a pod tries to exceed resources beyond its specified limit. In case of CPU, the system throttles the CPU so that it does not go beyond the specified limit.

  A container cannot use more CPU resources that its limit. However, this is not the case with memory. A container can use more memory resources that its limit. So if a pod tries to consume more memory than its limit constantly, the pod will be terminated an you will see that the pod terminated with an OOM error in the logs. OOM means out of memory kill. 

  By default, Kubernetes does not have a CPU or memory request or limit set. So this means that any pod can consume as much resources as required on any node and suffocate other pods or processes that are running on the node of resources. 

How CPU requests and limits work -

No Requests - No Limits

If there are two pods competing for CPU resources on the cluster, With Pod means container within pod. Without resource or limit set, 1 pod can consume all the CPU resource on the node and prevent the second pod of required resources. Which is not ideal. 

No Requests - Limits are Set ( Requests = Limits )

In this case, kubernetes automatically sets requests to the same as limits. 

Request Set - Limits Set

In this scenario, if Pod A uses 3 Vcpu's but needs more for some reason he can't use CPU allocated for Pod B.

Request Set - No Limits   ( Most Ideal SetUp )

Because requests are set each pod is guarenteed 1 Vcpu, however because limits are not set when available any pod can consume as many CPU cycles as available. Make sure all the pods have some requests set.

It works same for memory requests as well.

Bu default, kubernetes does not have resource requests or limits configured for pods. But then how do we ensure that every pod created has some default set. Now this is possible using limitrange. This is applicable on namespace level. 

LimitRange (limit-range-cpu.yaml)

apiVersion: v1

kind: LimitRange

metadata:

  name: cpu-resource-constraint

spec:

  limits:

  - default:

      cpu: 500m

    defaultRequest:

      cpu: 500m

    max:

      cpu: "1"

    min:

      cpu: 100m

    type: container


    --------------------------------------------------------------------------- (do memory instead of cpu)

    It will affect only newer pods and not older pods which are already created.

    Is there any way to restrict the total amount of resources that can be consumed by applications deployed in a kubernetes cluster?

    -> Create Quoatas at namespace level ( Resource Quotas)

    So, resource quota is a namespace level object that can be created to set hard limits for requests and limits.

    Resource-Quota.yaml

    ----------------------------------------------------

    apiVersion: v1

    kind: ResourceQuota

    metadata:

      name: my-resource-quota

    spec:

      hard:

        requests.cpu: 4

        requests.memory: 4Gi

        limits.cpu: 10

        limits.memory: 10Gi

    --------------------------------------------------------

    # Daemon Set

    Daemon Set are like Replica set, as in it helps you deploy multiple instances of pods but it runs one copy of your pod on each node in your cluster. Whenever a new node is added to cluster, a replica of pod is automatically added to that node and when node is removed, the pod is automatically removed. The Daemon set ensures one copy of the pod is always present in all nodes in the cluster.

What are some use case of DaemonSet?

You can deploy monitoring agent or log collector on each of your node in cluster. Daemon set is perfect for this. Kube-proxy is a good example of daemonset. Another use is for networking ( weave-net ). Creating a Daemon Set is similar to the ReplicaSet creation process. It has nested pod specification under the template section and selectors to link the DaemonSet to the pods.















































































 












































