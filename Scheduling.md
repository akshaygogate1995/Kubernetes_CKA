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









































