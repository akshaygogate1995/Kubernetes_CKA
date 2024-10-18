# Manual Scheduling

What you do when you do not have scheduler in cluster?
-> You probably dont want to rely on built-in scheduler, and instead scheduler the pod yourself.

How scheduler works in the backend?
-> Every pod has a field called nodeName, which is by default not set. We dont typically specify this field, when we create the pod manifest file, infact kubernetes adds it automatically.
-> The Scheduler goes through all the pods and looks for those that do not have this property set. Those are candidates for scheduling. It then identifies right node for the pod by running the scheduling algorithm. 
-> Once identified, it schedules the pod on the node by setting the nodeName property to the name of the node by creating a binding object. So, if there is no scheduler to monitor and schedule nodes, What happens? 
-> The Pods continue to be in pending state. So what can be done? You can manually assign pods to nodes yourself. Without scheduler, easiest way to schedule pod is to simply set nodeName field to name of node in your pod specification file while creating the pod. 
-> You can only specify the node name at creation time. What if pod is already created and you want to assign pod to a node? Kubernetes will not allow you to modify the nodeName property of a pod. So, another way to assign a node to a existing pod, is to create a binding object and send a POST request to the pod's binding API. Thus, mimicking what the actual scheduler does. In binding object, you specify a target node with the name of the node then send a POST request to the pod's binding API with the dataset to the binding object in a JSON format.SO, you must convert YAML file into its equivalent JSON form.

To check if scheduler is present do -> k get pods --namespace kube-system
if scheduler, is not present, container will always be in pending state in that case you have to add nodeName field.

---
apiVersion: v1
kind: Pod
metadata: 
  name: nginx
spec:
  nodeName: node-1
  containers: 
  - image: nginx
    name: nginx

Above is the example of how to manually schedule.
k replace --force -f nginx.yaml    -> This will delete and reapply in single command
k get pods --watch    (To watch it continously)

You cannot move running pod from one node to other. You have to delete it and recreate.












