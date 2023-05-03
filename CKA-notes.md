# Core Concepts/Practice Test - Pods SOLUTION

2. I created a manifest and ran `kubectl apply -f pods.yaml`; better approach would have been to run `kubectl run nginx --image=nginx`, which corresponds to `kubectl run <container_name> --image=<image_name>`

4. `kubectl describe pod <pod_name>`

5. I used `kubectl describe pod <pod_name>` for each of the pods to see what node it was placed on; better solution is `kubectl get pods -o wide` would show all the nodes the pods are on

11. `kubectl delete pod <pod_name>`

12. I created a manifest and ran `kubectl apply -f <manifest_file_name.yaml>`; better approach is to run `kubectl run <container_name> --image=<image_name> --dry-run=client -o yaml > redis.yaml`

13. I edited the YAML manifest, but I could have also run `kubectl edit pod/nginx` 


# Core Concepts/Practice Test - ReplicaSets SOLUTION

`kubectl explain replicaset` to get syntax of replicaset manifest
`kubectl get rs` is short form of replicaset


# Core Concepts/Practice Test - Deployments SOLUTION

11. I created an entire manifest from scratch; a better way to answer would be to run `kubectl create deployment --help` and see the third command from the top: 
`kubectl create deployment httpd-frontend --image=httpd:2.4-alpine --replicas=3`
then run `kubectl get deploy` to ensure that it took and that there are 3/3 pods all in READY state

GOTTA STOP CREATING MANIFESTS BY HAND; too time-consuming to do on the exam!!!!


# Core Concepts/Practice Test - Services SOLUTION

1. can use `kubectl get svc` as shorthand


# Core Concepts/Practice Test - Namespaces SOLUTION

1. can use `kubectl get ns` as shorthand

2. can use `kubectl get pods -n=research` as shorthand to get pods in the research namespace

3. can use `kubectl run redis --image=redis -n=finance` as shorthand to create a pod from the redis image in the finance namespace

4. can use `kubectl get pods -A` as shorthand to get all pods in all namespaces

6. an application can access another service in the same namespace just by using its name

7. in another namespace, an application must use the service's FQDN, which is of the form <service_name>.<namespace>.svc.cluster.local, e.g., `db-service.dev.svc.cluster.local`


# Core Concepts/Practice Test - Imperative Commands SOLUTION

2. `kubectl run <pod_name> --image=<image_name>`

3. `kubectl run <pod_name> --image=<image_name> --labels="<label_name>"`

4. `kubectl expose po <pod_name> --port <port_num> --name <service_name>`
- only use `kubectl create service` when you need to specify a particular `--node-port`
  - there is no way to specify the `--node-port` in the `kubectl expose pod` command
    - have to export the command to a YAML file and edit the file to add the node port
    - but `kubectl create service` doesn't allow specifying a selector

5. `kubectl create deploy <deployment_name> --image=<image_name> --replicas=N`

6. `kubectl run <pod_name> --image=<image_name> --port=N`

7. `kubectl create ns <namespace_name>`

8. `kubectl create deploy <deployment_name> --image=<image_name> --replicas N -n <namespace_name>`

9. Two ways:
  - create the pod with `kubectl run <pod_name>`, then create the service with `kubectl expose`
  - combine together: `kubectl run <pod_name> --image=<image_name> --port N --expose`


# Core Concepts/Practice Test - Manual Scheduling SOLUTION

3. look at the pods running in `-n kube-system` and see that there is no scheduler pod

4. add `nodeName: <node_name>` to the manifest under the `spec:` section
- also run `kubectl replace --force -f <manifest_file.yml>` to delete and recreate the pod

5. same as #4


# Core Concepts/Practice Test - Labels and Selectors SOLUTION

1. `kubectl get po --selector env=dev` and count
  - alternatively, `kubectl get po --selector env=dev --no-headers | wc -l`
2. `kubectl get po --selector bu=finance`
3. `kubectl get all --selector env=prod`
4. `kubectl get po --selector env=prod,bu=finance,tier=frontend`
5. change the rs label to match the pod label


# Core Concepts/Practice Test - Taints and Tolerations SOLUTION

2. `kubectl describe no node01 | grep Taint`
3. `kubectl taint no node01 spray=mortein:NoSchedule`
6. run `kubectl describe po mosquito` and look at the Events at the bottom
7. `kubectl run bee --image=nginx -o yaml --dry-run=client > bee.yml` and then add
      tolerations:
        - key: "spray"
          operator: "Equal"
          value: "mortein"
          effect: "NoSchedule"
  then run `kubectl create -f bee.yml`; NOTE: instructor did not put values inside quotes and it still worked
8. `kubectl get po -o wide`  
10. `kubectl taint no controlplane node-role.kubernetes.io/control-plane:NoSchedule-`


# Core Concepts/Practice Test - Node Affinity SOLUTION

3. `kubectl label no node01 <label_key>:<label_value>` **CHECK THIS**
4. `kubectl create deploy <deployment_name> --image=<image_name> --replicas N`
6. `kubectl create deploy blue --image=nginx --replicas 3 -o yaml --dry-run=client > deploy.yml` then add the following to deploy.yml and run `kubectl replace -f deploy.yml`:
      affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
            - matchExpressions:
              - key: color
                operator: In
                values:
                - blue
  could also have done `kubectl edit deployment blue`, copy the affinity block from K8s documentation, in Vim `V` and select the lines to indent to the right and `shift + .` for `>`
8. `kubectl create deploy red --image=nginx --replicas 2 --dry-run=client -o yaml > red-dep.yml` and add
      affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
            - matchExpressions:
              - key: node-role.kubernetes.io/master
                operator: Exists
  then run `kubectl create deploy -f red-dep.yml`                


# Core Concepts/Practice Test - Resource Limits SOLUTION

3. `kubectl describe po elephant` and look for the Last State section; OOMKilled indicates a memory issue; pod was killed because it ran out of memory
5. `kubectl edit po elephant` and change the memory limit; but that won't save the pod because this is an attempt to edit a field in the pod that isn't editable; but the file is saved in a temp directory, so can then run `kubectl replace --force -f <path_to_temp_file>`, which deletes the old pod and recreates it


# Core Concepts/Practice Test - DaemonSets SOLUTION

6. `kubectl create deploy elasticsearch -n=kube-system --image=registry.k8s.io/fluentd-elasticsearch:1.20 -o yaml --dry-run=client > fluentd-ds.yml` and delete the `replicas`, `strategy`, `status` (RSS) fields



# Notes:

- *every manifest contains:*
  - `apiVersion`
  - `kind`
  - `metadata`
  - `spec`


- *Objects (kinds) I've used:*
  - Pod
  - ReplicaSet
  - Deployment
  - Services
  - Namespace
  - DaemonSet


- *Two ways to increase the number of replicas in a ReplicaSet:*
  1. replace `replicas: N` with a new value of N
  2. run `kubectl scale --replicas=N -f <replicaset_definition_manifest.yaml>`
    - won't change the original number of replicas in the manifest, however


- *Deployments automatically create ReplicaSets*
  - ReplicaSets automatically create Pods


- *Miscellaneous commands:*
  - `kubectl get all` to see all objects created
  - `kubectl get po --watch` so don't have to refresh to see pods' statuses


- *Service types:*
  - NodePort
  - ClusterIP
  - LoadBalancer


- *NodePort:*
  - service that makes an internal pod on a node accessible through a port on the node
    - the service is like a virtual server inside the node
    - has its own IP address inside the cluster, the cluster IP of the service
  - maps a port on the node to a port on the pod
  - important terms (from viewpoint of the service):
    - TargetPort: port of the pod
      - where the service forwards the request to
      - if omitted, will be assumed to be equal to Port
    - Port: port of the NodePort service
      - this is the only one that is mandatory
    - NodePort: port of the node
      - range: 30000-32767
      - if omitted, a free port in the range will be automatically chosen


- *ClusterIP:*
  - the default type, so it can be omitted under `spec.type`
  - copy the `metadata.labels` array (`app` and `type`) to the service definition manifest to tell the service exactly which pods to connect to


- *Imperative vs. Declarative:*
  - using `kubectl edit <object_type> <object_name>` means that the previous change will be lost
    - better version: edit manifest file with changes and then run `kubectl replace -f <manifest_file.yml>` to update the object
      - the changes are recorded and tracked

  - `kubectl apply -f <manifest_file.yml>` will apply changes if the manifest file already exists, or will create the file if it doesn't exist yet
    - declarative approach
      - otherwise, using imperative approach one must check if the object already exists before using `kubectl replace`, and must check that the object doesn't already exist before using `kubectl create`

  - `kubectl create` and `kubectl replace` commands don't store the last-applied-configuration as an annotation in the live object configuration; only `kubectl apply` does
    - `kubectl apply` compares the local manifest file to the live object configuration and the last-applied-configuration inside the live object configuration to decide what changes must be made to the live configuration


- *2 ways to limit pods to run on specific nodes:*
  1. node selectors
    - simpler, easier
    - add `nodeSelector: <node_label>` to the `spec:` section
    - remember: `kubectl label nodes <node_name> <label_key>:<label_value>` to label a node
    - limitation: single label doesn't work on complex limitations
  2. node affinity


- *Daemon Sets:*
  - ensures that one copy of a pod runs on each node in the cluster
  - whenever a new node is added to the cluster, a replica of the pod is added to the new node; same if a node is removed
  - use cases: 
    - great for monitoring agents for each node because any changes in the cluster will be handled automatically by the daemon set
    - Kube-proxy: a worker node component that is required on every node in the cluster
    - Weave-net: requires an agent to be deployed on each agent in the cluster 
  - manifest is almost identical to that of a ReplicaSet except the kind changes
    - must ensure that the labels in the `selector` match the labels in the pod template
  - `kubectl create -f <daemon_set_manifest.yml>`
  - uses NodeAffinity and default scheduler starting in v1.12



# Work-related notes:

- which of our applications need redundancy, i.e., ReplicaSets to ensure high availability?

- which of our containers must be running on the same pod (see K8s Up and Running book for considerations)?