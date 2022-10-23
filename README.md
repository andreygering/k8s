# K8S Project




##### Comments | Convention
> name-objecttype-q6.yaml = name-objecttype-question6
##  Answers to task with imperative command 
How to execute manifest:
`kubectl apply -f mainifest/filename.yaml`


#####   Qestion 4: Get list of nodes in JSON format and store it in a file /tmp/nodes-andrew
`kubectl get nodes  -o json > /tmp/nodes-andrew.json`

##### Question 5: Create a service messaging-service to expose the messaging application within the cluster on port 6379. Use imperative commands - kubectl
`kubectl create service clusterip messaging-service --tcp 6379:6379 tier=msg`

##### Qestion 13: Create a new deployment called nginx-deploy, with image nginx:1.16 and 1 replica. Record the version. Next upgrade the deployment to version 1.17 using rolling update. Make sure that the version upgrade is recorded in the resource annotation.

`kubectl create deployment nginx-deploy --image=nginx:1.16  --record`
`kubectl set image deployment/nginx-deploy nginx=nginx:1.17 --record`

##### Question 14: 



#####    Expose nginx-resolver internally with a nginx-resolver-service
`kubectl expose pod/nginx-resolver --type="ClusterIP" --port 8080`


###  Pod Design Answers to Qestions:

#####    1. Type the command for: Get pods with label information
`kubectl get pods --show-labels`
#####    2. Create 5 nginx pods in which two of them is labeled env=prod and three of them is labeled env=dev

#####    3. Verify all the pods are created with correct labels
`kubectl get pods -l env=prod && kubectl get pods -l env=dev`
#####    4. Get the pods with label env=dev
`kubectl get pods -l env=dev`
#####    5. Get the pods with label env=dev and also output the labels
`kubectl get pods -l env=dev --show-labels`
#####    6. Get the pods with label env=prod
`kubectl get pods -l env=prod`
#####    7. Get the pods with label env=prod and also output the labels
`kubectl get pods -l env=prod --show-labels`
#####    8. Get the pods with label env
`kubectl get pods -l env`
#####    9. Get the pods with labels env=dev and env=prod
`kubectl get pods -l env`
#####    10.Get the pods with labels env=dev and env=prod and output the labels as well
`kubectl get pods -l env --show-labels`
#####    11. Change the label for one of the pod to env=uat and list all the pods to verify
`kubectl label --overwrite pods pod-name env=uat && k get pods --show-labels`
#####    12. Remove the labels for the pods that we created now and verify all the labels are removed
`kubectl label pods pod-name env- && k get pods --show-labels`
#####    13. Let’s add the label app=nginx for all the pods and verify (using kubectl)
`kubectl label pods --all app=nginx && kubectl get pods --show-labels ` 
#####    14.Get all the nodes with labels (if using minikube you would get only master node)
`kubectl get nodes --show-labels`
#####    15. Label the worker node nodeName=nginxnode
`kubectl label nodes worker nodeName=nginxnode && kubectl get nodes --show-labels`
#####    16.Create a Pod that will be deployed on the worker node with the label nodeName=nginxnode
Manifest name: node_affinity_nginx_pod.yaml
```
apiVersion: v1
kind: Pod
metadata:
  name: nginx-node-affinity
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: nodeName
            operator: In
            values:
            - nginxnode            
  containers:
  - name: nginx-node-affinity
    image: nginx
    imagePullPolicy: IfNotPresent

```
kubectl apply -f node_affinity_nginx_pod.yaml

#####    16.1 Add the nodeSelector to the below and create the pod
kubectl edit pod nginx-node-affinity 
Then add to node_affinity_nginx_pod.yaml udner spec affinity module.
```
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: topology.kubernetes.io/zone
            operator: In
            values:
            - antarctica-east1
            - antarctica-west1
```
#####    17. Verify the pod that it is scheduled with the node selector on the right node… fix it if it’s not behind scheduled.

#####    18.Verify the pod nginx that we just created has this label
`kubectl describe pod pod nginx` - verify and edit if required. 

###  Deployments:
#####    1. Create a deployment called webapp with image nginx with 5 replicas
#####    a. Use the below command to create a yaml file.
#####    i. kubectl create deploy webapp --image=nginx --dry-run=client -o yaml > webapp.yaml
#####    ii. Edit it and add 5 replica’s
`kubectl create deployment webapp --image=nginx --replicas=5` ( also in webapp-deployment-q1.yaml)
#####    2. Get the deployment rollout status
`kubectl rollout status deployment/webapp`
#####    3. Get the replicaset that created with this deployment
`kubectl get rs | grep webapp `
#####    4. EXPORT the yaml of the replicaset and pods of this deployment
`kubectl describe rs/webapp-6684ccd7b8 > webapp_replicaset.yaml`
#####    5. Delete the deployment you just created and watch all the pods are also being deleted
`kubectl delete deployment webapp && kubectl get pods -l app=webapp`
#####    6. Create a deployment of webapp with image nginx:1.17.1 with container port 80 and verify the image version
`kubectl create deployment webapp --image=nginx:1.17.1 --port=80 && kubectl get deployment -o wide`
#####    a. kubectl create deploy webapp --image=nginx:1.17.1 --dry-run=client -o yaml > webapp.yaml
#####    b. add the port section (80) and create the deployment
`kubectl edit deployment/webapp` and add in container sepcs ports module 
```
ports:
        - containerPort: 8081
```
#####    7. Update the deployment with the image version 1.17.4 and verify
`kubectl set image deployment/webapp nginx=nginx:1.17.4`
#####    8. Check the rollout history and make sure everything is ok after the update
`kubectl rollout history deployment/webapp  && kubectl rollout status deployment/webapp `
#####    9. Undo the deployment to the previous version 1.17.1 and verify Image has the previous version
`kubectl rollout undo deployment/webapp --to-revision=2`
#####    10.Update the deployment with the wrong image version 1.100 and verify something is wrong with the deployment
`kubectl set image deployment/webapp nginx=nginx:1.100`
#####    a. Expect: kubectl get pods (ImagePullErr)
`kubectl get pods -o wide`
#####    b. Undo the deployment with the previous version and verify everything is Ok
`kubectl rollout undo deployment/webapp --to-revision=3`
#####    c. kubectl rollout history deploy webapp --revision=7
`kubectl rollout undo deployment/webapp --to-revision=7`
#####    d. Check the history of the specific revision of that deployment
`kubectl rollout history deployment/webapp --revision=4`
#####    e. update the deployment with the image version latest and check the history and verify nothing is going on
`kubectl set deployment/webapp nginx=nginx:latest && kubectl rollout history deployment/webapp && kubectl rollout status deployment/webapp` 
#####    11. Apply the autoscaling to this deployment with minimum 10 and maximum 20 replicas and target CPU of 85% and verify hpa is created and replicas are increased to 10 from 1
Answer in yaml file webapp-scaling-deployment-q11.yaml
#####    12. Is no Question. 
#####    13. Clean the cluster by deleting deployment and hpa you just created
`kubectl delete deployment nginx-deployment --cascade=foreground`
#####    14. Create a job and make it run 10 times one after one (run > exit > run >exit ..) using the following configuration: kubectl create job hello-job --image=busybox --dry-run -o yaml -- echo "Hello I am from job" > hello-job.yaml”
#####    a. Add to the above job completions: 10 inside the yaml
Answer at yaml file: hello-job.yaml

###  CONFIG MAP:
#####    1. Create a file called config.txt with two values key1=value1 and key2=value2 and verify the file
File created, filename: config.txt
#####    2. Create a configmap named keyvalcfgmap and read data from the file config.txt and verify that configmap is created correctly
Manifest filename: keyvalcfgmap-configmap.yaml
#####    3. Create an nginx pod and load environment values from the above configmap keyvalcfgmap and exec into the pod and verify the environment variables and delete the pod // first run this command to save the pod yml kubectl run nginx --image=nginx --restart=Never --dry-run -o yaml > nginx-pod.yml
Manifests yaml file names: keyvalcfgmap-configmap.yaml and nginx-pod.yaml







###### Notes
$(kubectl get pods | awk '/worker/ {print $1;exit}')