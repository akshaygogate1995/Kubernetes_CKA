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




































