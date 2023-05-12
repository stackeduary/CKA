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

2. can use `kubectl get pods -n research` as shorthand to get pods in the research namespace

3. can use `kubectl run redis --image=redis -n finance` as shorthand to create a pod from the redis image in the finance namespace

4. can use `kubectl get pods -A` as shorthand to get all pods in all namespaces

6. an application can access another service in the same namespace just by using its name

7. in another namespace, an application must use the service's FQDN, which is of the form <service_name>.<namespace>.svc.cluster.local, e.g., `db-service.dev.svc.cluster.local`


# Core Concepts/Practice Test - Imperative Commands SOLUTION

2. `kubectl run <pod_name> --image=<image_name>`
   - image name is required!

3. `kubectl run <pod_name> --image=<image_name> --labels="<label_name>"`

4. `kubectl expose po <pod_name> --port <port_num> --name <service_name>`
- only use `kubectl create service` when you need to specify a particular `--node-port`
  - there is no way to specify the `--node-port` in the `kubectl expose pod` command
    - have to export the command to a YAML file and edit the file to add the node port
    - but `kubectl create service` doesn't allow specifying a selector

1. `kubectl create deploy <deployment_name> --image=<image_name> --replicas=N`

2. `kubectl run <pod_name> --image=<image_name> --port=N`

3. `kubectl create ns <namespace_name>`

4. `kubectl create deploy <deployment_name> --image=<image_name> --replicas N -n <namespace_name>`

5. Two ways:
  - create the pod with `kubectl run <pod_name>`, then create the service with `kubectl expose`
  - combine together: `kubectl run <pod_name> --image=<image_name> --port N --expose`


# Scheduling/Practice Test - Manual Scheduling SOLUTION

3. look at the pods running in `-n kube-system` and see that there is no scheduler pod

4. add `nodeName: <node_name>` to the manifest under the `spec:` section
- also run `kubectl replace --force -f <manifest_file.yml>` to delete and recreate the pod

5. same as #4


# Scheduling/Practice Test - Labels and Selectors SOLUTION

1. `kubectl get po --selector env=dev` and count
  - alternatively, `kubectl get po --selector env=dev --no-headers | wc -l`

2. `kubectl get po --selector bu=finance`

3. `kubectl get all --selector env=prod`

4. `kubectl get po --selector env=prod,bu=finance,tier=frontend`

5. change the rs label to match the pod label


# Scheduling/Practice Test - Taints and Tolerations SOLUTION

2. `kubectl describe no node01 | grep Taint`

3. `kubectl taint no node01 spray=mortein:NoSchedule`

4. run `kubectl describe po mosquito` and look at the Events at the bottom

7. `kubectl run bee --image=nginx -o yaml --dry-run=client > bee.yml` and then add
      tolerations:
        - key: "spray"
          operator: "Equal"
          value: "mortein"
          effect: "NoSchedule"
  then run `kubectl create -f bee.yml`; NOTE: instructor did not put values inside quotes and it still worked

8. `kubectl get po -o wide`  

10.  `kubectl taint no controlplane node-role.kubernetes.io/control-plane:NoSchedule-`


# Scheduling/Practice Test - Node Affinity SOLUTION

1. `k describe no node01`

3. `kubectl label no node01 <label_key>=<label_value>`

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


# Scheduling/Practice Test - Resource Limits SOLUTION

3. `kubectl describe po elephant` and look for the Last State section; OOMKilled indicates a memory issue; pod was killed because it ran out of memory

5. `kubectl edit po elephant` and change the memory limit; but that won't save the pod because this is an attempt to edit a field in the pod that isn't editable; but the file is saved in a temp directory, so can then run `kubectl replace --force -f <path_to_temp_file>`, which deletes the old pod and recreates it


# Scheduling/Practice Test - DaemonSets SOLUTION

6. `kubectl create deploy elasticsearch -n kube-system --image=registry.k8s.io/fluentd-elasticsearch:1.20 -o yaml --dry-run=client > fluentd-ds.yml` and delete the `replicas`, `strategy`, `status` (RSS) fields


# Scheduling/Practice Test - Static Pods SOLUTION

1. `kubectl get po -A | grep controlplane`
   - double check by running `kubectl get pod <pod_name> -n kube-system -o yaml` and check the `ownerReferences.kind = Node` section to be sure

5. `ps aux | grep kubelet | grep config` => `--config=/var/lib/kubelet/config.yaml` => `less /var/lib/kubelet/config.yml` => `staticPodPath: /etc/kubernetes/manifests`

8. `kubectl run static-busybox --image=busybox --dry-run=client -o yaml --command -- sleep 1000 > /etc/kubernetes/manifests/static-busybox.yml` then run `kubectl get pods` to ensure that it was created
   - SHOULD ALWAYS VERIFY, especially on the exam: `kubectl get pods --watch` 

10. `kubectl get pods` will reveal that the pod is named `static-greenbox-node01` and is on `node01`
    - `kubectl get nodes -o wide` and note the internal IP address
    - `ssh <IP_address>` or `ssh <node_name>`
    - `cat /var/lib/kubelet/config.yml` find the `staticPodPath` and then remove the manifest from that directory
    - exit that node and then run `kubectl get pods --watch` to ensure that that pod is terminated


# Scheduling/Practice Test - Multiple Schedulers SOLUTION

4. `kubectl create cm <CM_name> --from-file=<path_to_CM_manifest> -n <namespace>`

5. `kubectl get pods -A` => `kubectl describe po -n kube-system kube-scheduler-controlplane | grep Image` => copy and paste into `vim my-scheduler.yml` in the image section => `kubectl create -f my-scheduler.yml`

6. add `schedularName: my-scheduler` to the `spec` section of `nginx.yml` 


# Logging & Monitoring/Practice Test - Managing Application Logs SOLUTION

2. `kubectl logs -f <pod_name>`

4. `kubectl logs <pod_name> <container_name>` if there are multiple containers running on a single pod


# Application Lifecycle Management/Practice Test - Rolling Updates and Rollbacks SOLUTION

11. `kubectl edit deploy frontend` and change image version
  - `kubectl set image deployment <deployment_name> <container_name>=<image_name>:<image_version>` would have also worked

12. `k edit deploy frontend`

13. `k set image deploy frontend simple-webapp=kodekloud/webapp-color:v3`


# Application Lifecycle Management/Practice Test - Commands and Arguments SOLUTION

5. `k edit po ubuntu-sleeper-3` won't work because the command property can't be changed; but can use `k replace --force -f <path_to_temp_file_created.yml>`  

10. 
  - harder way: `k run webapp-green --image=kodekloud/webapp-color --dry-run=client -o yaml > output-file.yml` and add the command line arguments
  - easier way: `k run webapp-green --image=kodekloud/webapp-color -- --color green`
    - equivalent to: `python app.py --color green`


# Application Lifecycle Management/Practice Test - Environment Variables SOLUTION

5. `k edit po`
  **USE TO EDIT A POD WHEN YOU DON'T KNOW WHERE THE MANIFEST FILE IS!!!!!!!!!!!**
  `k delete webapp-color`
  `k apply -f /tmp/kubectl-edit-2222681163.yaml`
  could have also done:
  `k replace --force -f /tmp/kubectl-edit-2222681163.yaml` in one step, not two

9. `k create cm webapp-config-map --from-literal=APP_COLOR=darkblue --from-literal=APP_OTHER=disregard`

10. `k edit po webapp-color`
    added to containers:
      - env:
        - name: APP_COLOR
          valueFrom:
            configMapKeyRef:
              name: webapp-config-map
              key: APP_COLOR
    `k delete po webapp-color`
    `k apply -f /tmp/kubectl-edit-3483122881.yaml`

    could have also added to containers:
    - envFrom
      - configMapRef:
        name: webapp-config-map


# Application Lifecycle Management/Practice Test - Secrets SOLUTION

6. `k create secret generic db-secret --from-literal=DB_Host=sql01 --from-literal=DB_User=root --from-literal=DB_Password=password123`

7. `k edit po webapp-pod`
   added to spec:
      spec:
        containers:
        - image: kodekloud/simple-webapp-mysql
          imagePullPolicy: Always
          name: webapp
          envFrom:
          - secretRef:
              name: db-secret
   `k replace --force -f /tmp/kubectl-edit-758332135.yaml`


# Application Lifecycle Management/Practice Test - Multi Container Pods SOLUTION

1. `k describe po <pod_name>` is one way to get the number of containers running in a pod, but `k get po` and look at the "READY" column is much faster

3. `k run yellow --image=busybox --dry-run=client -o yaml > yellow.yml`
    added:
      containers:
      - image: busybox
        name: lemon
        command: ["sleep", "1000"]
      - image: redis
        name: gold
      `k replace --force -f yellow.yml`

7. `k exec app -c app -it -- /bin/sh`
   could have also done `k logs app -n elastic-stack` to see the logs in order to answer the question
   or `k -n elastic-stack exec -it app -- cat /log/app.log` with fewer commands

8. `k edit po app`
   copied the previous image and changed to:
      - image: kodekloud/filebeat-configured
          imagePullPolicy: Always
          name: sidecar
          resources: {}
          terminationMessagePath: /dev/termination-log
          terminationMessagePolicy: File
          volumeMounts:
          - mountPath: /var/log/event-simulator/
            name: log-volume
          - mountPath: /var/run/secrets/kubernetes.io/serviceaccount
            name: kube-api-access-gjx69
            readOnly: true    
   `k replace --force -f /tmp/kubectl-edit-1199939433.yaml`


# Application Lifecycle Management/Practice Test - Init Containers SOLUTION

1. use `k describe po` to get a description of all the pods at once instead of one at a time
   - the READY column excludes initContainers; it only considers normal containers in that pod 

8. `k edit po red` and added:
   initContainers:
  - command:
    - sleep
    - "20"
    image: busybox
    name: bizzybox
   `k replace --force -f /tmp/kubectl-edit-2220993593.yaml`

9. `k logs orange` won't return anything meaningful because the pod is in an initializing state (waiting to start), so logs aren't really generated
   `k logs orange -c init-myservice` does return meaningful logs  


# Cluster Maintenance/Practice Test - OS Upgrades SOLUTION

4. `k drain node01 --ignore-daemonsets`

6. `k uncordon node01`

7. `k get po -o wide`

9. pods are not usually scheduled on the controlplane node because it usually has some sort of taint on it; but controlplane has no taints on it, so that's why pods are placed on the controlplane node
  - usually in a multi-node cluster, the controlplane node has taints placed on it so that no application pods are scheduled on it

12. answer: there is a pod on node01 that isn't part of a replica set
  - when doing a drain, the node is first uncordoned and marked unschedulable, then tries to evict the pods
  - it's easy to delete pods managed by ReplicaSets, Jobs or ReplicationControllers because when a pod is manually deleted, these objects take care of recreating that pod on another node
    - less risky
  - but this pod is not created by another object; it's a lone pod, so if that pod is deleted as part of the draining process, that pod and any data stored on it will be lost forever
    - so that's why the `hr-app` pod was not killed

13. `k get po -o wide`

14. `k drain node01 --ignore-daemonsets --force`

16. `k cordon node01`
    `k get po -o wide`


# Cluster Maintenance/Practice Test - Cluster Upgrade Process SOLUTION

1. `k get no`

3. `k describe node | grep Taints`
   - neither node has taints on it, so both can host applications; so 2

4. `k get deploy`
   1

5. `k get po -o wide`
   controlplane, node01

7. `kubeadm upgrade plan`

9. `k drain controlplane --ignore-daemonset`

10. process is upgrade kubeadm, upgrade master, upgrade kubelet:
   `apt update`
   `apt-cache madison kubeadm` (question already told us to upgrade to 1.26.0-00, so this isn't necessary)
   looking at the documentation TO UPGRADE kubeadm:
   `apt-mark unhold kubeadm && apt-get update && apt-get install -y kubeadm=1.26.0-00 && apt-mark hold kubeadm`
   `kubeadm version` to verify that kubeadm is at version v1.26.0
   `kubeadm upgrade plan`
   `kubeadm upgrade apply v1.26.0`
   skip upgrading CNI for now; also, no other controlplane node
   (reacting to documentation step) node already drained, so don't have to do that
   upgrade kubelet and kubectl:
   `apt-mark unhold kubelet kubectl && apt-get update && apt-get install -y kubelet=1.26.0-00 kubectl=1.26.0-00 && apt-mark hold kubelet kubectl`
   `systemctl daemon-reload`
   `systemctl restart kubelet`
   `k get no` to verify that the controlplane node was updated to v1.26.0

11. `k uncordon controlplane`
    `k get no` to verify that it is schedulable now

12. `k drain node01 --ignore-daemonsets`
    `k get no` to ensure that node01 is not schedulable
    `k get po -o wide` to ensure that all pods are on controlplane (none on node01)

13. upgrade kubeadm:
    MUST BE ON WORKER NODE; MUST SSH
    `ssh node01` (or `k get no -o wide` and look for the internal IP and SSH using that)
    `apt-mark unhold kubeadm && apt-get update && apt-get install -y kubeadm=1.26.0-00 && apt-mark hold kubeadm`
    `kubeadm upgrade node`
    (reacting to documentation step) node already drained, so don't have to do that
    `apt-mark unhold kubelet kubectl && apt-get update && apt-get install -y kubelet=1.26.0-00 kubectl=1.26.0-00 && apt-mark hold kubelet kubectl`
    `systemctl daemon-reload`
    `systemctl restart kubelet`

14. `k uncordon node01`
    `k get no` to verify that node01 is schedulable and has been upgraded


# Cluster Maintenance/Practice Test - Backup and Restore Methods SOLUTION

2. `k describe po -n kube-system etcd-controlplane` and find the `Image: k8s.gcr.io/etcd:3.5.1-0`. 3.5.1 is the answer

3. most details that this question asks about are in the `Command: etcd` section: 
   `--listen-client-urls=https://127.0.0.1:2379` is what we want

4. `--cert-file=/etc/kubernetes/pki/etcd/server.crt`

5. `--trusted-ca-file=/etc/kubernetes/pki/etcd/ca.crt`

6. It's a static pod because it is named `etcd-controlplane` and static pods are created from manifests in `/etc/kubernetes/manifests`, so he opened `etcd.yaml` in that directory and highlighted the `hostPath` section, which is the path on the node that the current pod is running on (controlplane in this case); the volume called `etcd-certs` (see `volumes.hostPath.name`) is created from the `/etc/kubernetes/pki/etcd` path on the host and is used as a volumeMount; that is, `volumes.hostPath.path` is `/etc/kubernetes/pki/etcd` on the host and is mounted to `volumeMounts.mountPath`, which is `/etc/kubernetes/pki/etcd`, in the container; similarly, the data path called `etcd-data` at volumes.hostPath.names `/var/lib/etcd` on the host is mounted to `/var/lib/etcd` on the container
next, look at options passed in `spec.containers.command` section of the `etcd-controlplane` pod: `--data-dir=/var/lib/etcd`; `ls /var/lib/etcd` shows a `member` directory, which is where ETCD stores information about the cluster; `ls /etc/kubernetes/pki/etcd` shows the certificate files used by ETCD

if I run `etcdctl snapshot` (or any other command) and it responds with "No help topic for 'snapshot'", then that means that the `ETCDCTL_API=3` environment variable needs to be set first

   `export ETCDCTL_API=3`

   `ls -la /etc/kubernetes/pki/etcd` to verify that all the authentication files are present
   `etcdctl snapshot save /opt/snapshot-pre-boot.db --endpoints=127.0.0.1:2379 --cacert=/etc/kubernetes/pki/etcd/ca.crt --cert=/etc/kubernetes/pki/etcd/server.crt --key=/etc/kubernetes/pki/etcd/server.key`

8. `k get po -A`
   `k get svc -A`
   `k get deploy -A`
   those three were enough; `k get all --all-namespaces` could have worked as well
    - returned pods, services, daemonsets, deployments and replicasets

9. plan is to use the `etcdctl restore` command to restore the data to a directory on the localhost and will reconfigure the manifest file `/etc/kubernetes/manifests/etcd.yaml` and the `volumes.hostPath.path` value to use the path we restored the data to, not the current `/var/lib/etcd` directory; remember that taking a snapshot means contacting the ETCD server directly at the endpoint, so must pass all the certificates along with the request to the endpoint, but in this case, the `etcdctl snapshot restore` operation of restoring data from an ETCD backup is not done by communicating with the ETCD server, it's just a local operation, so we don't need to specify the endpoint or the certificate details
   `etcdctl snapshot restore /opt/snapshot-pre-boot.db --data-dir /var/lib/etcd-from-backup`
   `vim /etc/kubernetes/manifests/etcd.yaml` and change `volumes.hostPath.path` where name is `etcd-data` to `/var/lib/etcd-from-backup`; any change in this manifest file will recreate the `etcd-controlplane` pod and will mount `/var/lib/etcd-from-backup` on the host to `/var/lib/etcd` on the pod/container; so `spec.containers.command` `--data-dir=/var/lib/etcd` should always match `volumeMounts.mountPath` for `etcd-data`; but `volumes.hostPath.path` and `volumeMounts.mountPath` don't have to match because their names do the matching `etcd-data` in this case
   fun fact: when `etcd-controlplane` restarts, it also restarts the `kube-controller-manager-controlplane` and `kube-scheduler-controlplane` pods
   he had to run `k delete po -n kube-system etcd-controlplane` to delete the pod, which will automatically restart it


# Cluster Maintenance/Practice Test - Backup and Restore Methods 2 SOLUTION

2. `k config view` shows all of the currently configured clusters

3. `k config get-clusters`
   `k config view` also works

4. `k config use-context cluster1`
   `k get no -A`

5. `k config use-context cluster2`
   `k get no -A`

7. `k config use-context cluster1`
   `k describe po -n kube-system etcd-cluster1-controlplane`
    This means that ETCD is set up as a Stacked ETCD Topology where the distributed data storage cluster provided by etcd is stacked on top of the cluster formed by the nodes managed by kubeadm that run control plane components. 
    from solution video: after running `k get po -n kube-system` "if we see a pod for ETCD that means we're using Stacked ETCD--the ETCD server is running on the controlplane node."
    Can also check by running `k describe po -n kube-system kube-apiserver-cluster1-controlplane` and check the URL that the kube-apiserver uses to communicate with the ETCD server. If it points to `localhost` or the IP address of the controlplane node, then that means it uses Stackded ETCD 

8. `k config set-context cluster2`
   `k get po -n kube-system` reveals no ETCD pod
   `ssh cluster2-controlplane` (use `k get po` to find the node names to SSH into)
   `ls -la /etc/kubernetes/manifests` reveals no ETCD manifest
   `ps -ef | grep etcd` shows that the process for the kube-apiserver is referencing an external etcd datastore
   `k describe po -n kube-system kube-apiserver-cluster2-controlplane` also shows similar information--see the `--etcd-servers` config; here it is `etcd-servers=https://192.26.148.23:2379`, which shows that it points to a separate IP address

9. `k describe po -n kube-system kube-apiserver-cluster2-controlplane | grep etcd` shows an external server: `--etcd-servers=https://192.26.148.23:2379`

10. `k describe po -n kube-system etcd-cluster1-controlplane | grep data` returns `--data-dir=/var/lib/etcd  /var/lib/etcd from etcd-data (rw) etcd-data:`

11. `ssh cluster2-controlplane ps -ef | grep etcd-servers` and look for `--etcd-servers=https://192.26.148.23`

12. `ssh etcd-server`
    `ps -ef | grep etcd`
    look for the `--data-dir` config option. Anwer is: `/var/lib/etcd-data`

13. To answer how many nodes are in the ETCD cluster, connection details to the ETCD server are required (found from running `ps -ef | grep etcd` from the `etcd-server` node): 
   `export ETCDCTL_API=3`
   `ls -la /etc/etcd/pki` to verify that all the authentication files are present
   `etcdctl member list --endpoints=127.0.0.1:2379 --cacert=/etc/etcd/pki/ca.pem --cert=/etc/etcd/pki/etcd.pem --key=/etc/etcd/pki/etcd-key.pem`
   this shows that there's only one member in this cluster because there's only one entry returned

14. from the `student-node`: 
    `k config use-context cluster1`
    `k describe po -n kube-system etcd-cluster1-controlplane | grep pki`
    need the `advertise-client-urls` config option, the URL to reach the ETCD server
    `ssh cluster1-controlplane`
    `export ETCDCTL_API=3`
    `etcdctl snapshot save /opt/cluster1.db --endpoints=127.0.0.1:2379 --cacert=/etc/kubernetes/pki/etcd/ca.crt --cert=/etc/kubernetes/pki/etcd/server.crt --key=/etc/kubernetes/pki/etcd/server.key`
    `scp cluster1-controlplane:/opt/cluster1.db /opt` because instructions say to save on `student-node`
    general pattern: `scp [host from dir] [host to dir]`, i.e., `scp hostname_from:dir hostname_to:dir`

15. `scp /opt/cluster2.db etcd-server:/root`
    `ssh etcd-server`
    `export ETCDCTL_API=3`
    `etcdctl snapshot restore /root/cluster2.db --data-dir /var/lib/etcd-data-new --endpoints=127.0.0.1:2379 --cacert=/etc/etcd/pki/ca.pem --cert=/etc/etcd/pki/etcd.pem --key=/etc/etcd/pki/etcd-key.pem`
    instructor said that connection details above aren't needed because we're just doing a restore, so don't even need to connect to the ETCD server
    `vim /etc/systemd/system/etcd.service` and add `--data-dir=/var/lib/etcd-data-new`
    `chown -R etcd:etcd /var/lib/etcd-data-new` since permissions are set to the root user by default; want to set to the `etcd` user
    `systemctl daemon-reload`
    `systemctl restart etcd`
    should reset all the controlplane components to ensure they're not using any stale data:
    `k config use-context cluster2`
    `k delete po -n kube-system kube-controller-manager-cluster2-controlplane kube-scheduler-cluster2-controlplane`
    `ssh cluster2-controlplane`
    `systemctl restart kubelet`


# Security/Practice Test - View Certificate Details SOLUTION


# Security/Practice Test - Certificates API SOLUTION


# Security/Practice Test - KubeConfig SOLUTION


# Security/Practice Test - Role Based Access Controls SOLUTION


# Security/Practice Test - Cluster Roles SOLUTION


# Security/Practice Test - Service Accounts SOLUTION


# Security/Practice Test - Image Security SOLUTION


# Security/Practice Test - Security Contexts SOLUTION


# Security/Practice Test - Network Policies SOLUTION


# Storage/Practice Test - Persistent Volume Claims SOLUTION


# Storage/Practice Test - Storage Class SOLUTION


# Networking/Practice Test - Explore Environment SOLUTION


# Networking/Practice Test - CNI SOLUTION


# Networking/Practice Test - Deploy Network Solution SOLUTION


# Networking/Practice Test - Networking Weave SOLUTION


# Networking/Practice Test - Service Networking SOLUTION


# Networking/Practice Test - CoreDNS in Kubernetes SOLUTION


# Networking/Practice Test - CKA Ingress Networking 1 SOLUTION


# Networking/Practice Test - CKA Ingress Networking 2 SOLUTION


# Kubernetes the kubeadm Way/Practice Test - Deploy a Kubernetes Cluster Using kubeadm SOLUTION


# Troubleshooting/Practice Test - Application Failure SOLUTION


# Troubleshooting/Practice Test - Control Plane Failure SOLUTION


# Troubleshooting/Practice Test - Worker Node Failure SOLUTION


# Troubleshooting/Practice Test - Troubleshoot Network SOLUTION


# Other Topics/Practice Test - Advanced kubectl Commands SOLUTION






















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
  - ConfigMap
  - Secret


- *Two ways to increase the number of replicas in a ReplicaSet:*
  1. replace `replicas: N` with a new value of N
  2. run `kubectl scale --replicas=N -f <replicaset_definition_manifest.yaml>`
    - won't change the original number of replicas in the manifest, however


- *Deployments automatically create ReplicaSets*
  - ReplicaSets automatically create Pods
  - a deployment manages a replica set and a replica set manages pods


- *Namespaces:*
  - `k config set-context --current --namespace=<new_namespace_name>` to set the default namespace to `<new_namespace_name>`


- *Miscellaneous commands:*
  - `kubectl get all` to see all objects created
  - `kubectl get po --watch` so don't have to refresh to see pods' statuses
  - `k exec <pod_name> -c <container_name> -it -- /bin/sh (or bash)` to exec into a running container
  - `alias` to set `k=kubectl`
  - `k api-resources` to see short names for API resources (objects)


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


- *Static Pods:*
  - have the node name appended to the pod name
    - can double check by running `kubectl get pod -n kube-system <pod_name> -o yaml` and check the `ownerReferences.kind = Node` section to be sure
    - non-static pods are not owned by the node, but by some other object
  - it isn't possible to delete a static pod with the `kubectl delete pod <pod_name>` command
    - instead, have to delete the manifest file for that static pod


- *Multiple Schedulers:*
  - you can extend Kubernetes and write your own scheduling algorithm to place pods on nodes, package and deploy it as the default scheduler or as an additional scheduler
  - the default scheduler is called `default-scheduler` and is configured in `scheduler-config.yml`
  - instead of creating a pod with the scheduler defined in it directly, a deployment can be used that will manage a replica set that will in turn manage the scheduler pods, thereby making the scheduler resilient to failures


- *Scheduling:*
  - 4 phases: 
    - scheduling
      - pods to be scheduled go in a scheduling queue
      - to set a priority, first must set a PriorityClass object
        - requires a priority name and a priority value
      - PrioritySort plugin
      - queueSort extension point plus others
    - filtering
      - when nodes that can't run the pod are filtered out
      - NodeResourcesFit plugin
      - NodeName plugin
      - NodeUnschedulable plugin
      - filter extension point plus others
    - scoring
      - nodes are scored with different weights
      - weights = amount of free space left after a pod is scheduled on it (more space left => higher weight)
      - NodeResourcesFit plugin
      - ImageLocality plugin
        - gives nodes that already have the container image a higher score
        - score extension point plus others
    - binding
      - when pod is finally bound to a node with the highest score
      - DefaultBinder plugin
      - bind extension point plus others


- *Kubelet:*
  - resides on each node and runs instructions from KubeAPI-server


- *Rollouts and Versioning:*
  - creating a deployment triggers a rollout
    - a new rollout creates a new deployment revision
  - when container version is updated, a new rollout is triggered and a new deployment revision is created
    - allows tracking changes made to the deployment
    - enables rolling back to a previous version of the deployment
  - `kubectl rollout status deployment/<name_of_deployment>`
  - `kubectl rollout history deployment/<deployment_name>` to see the revisions and rollout history
  - 2 types of deployment strategies
    - Recreate strategy
      - delete all replicas (instances) of the application and recreate them with a newer version
        - application has a down period and is inaccessible to users
    - Rolling Update strategy
      - take each instance down individually and replace with a newer version one-by-one
        - application never goes down and upgrade is seamless
      - default deployment strategy
  - `kubectl rollout undo deployment/<name_of_deployment>` to rollback to a previous revision


- *Docker and Kubernetes commands:*
  - `ENTRYPOINT[<shell_program>]` command in Docker corresponds to `command: ["<shell_program>"]` in `spec.containers` in the pod manifest
    - entrypoint is what is run when the container is started up
  - `CMD[<argument>]` command in Docker corresponds to `args: ["<argument>"]` in `spec.containers` in the pod manifest
  - `command:` is always an array, so it must use the syntax `command: ["sleep", "5000"]` 
    or the syntax
      command:
      - "sleep"
      - "5000"
    or the syntax
      command: ["sleep"]
      args: ["5000"]
  - if the Dockerfile contains
      ENTRYPOINT ["python", "app.py"]
      CMD ["--color", "red"]
    then the command that is run at startup is `python app.py --color red`
  - if there's a Dockerfile and a pod manifest, the image is created from the Dockerfile and the pod is created from the manifest using the image created from the Dockerfile
  - manifest command overrides ENTRYPOINT in Dockerfile
  - manifest args override CMD in Dockerfile
  - the `--` in the command separates the commands for the `kubectl` utility from commands for the application running inside the container
    - before the `--` is for `kubectl`; anything after `--` is for the application
    - `kubectl ... -- <arg1> <arg2> ... <argN>`
    - `kubectl ... --command -- <cmd> <arg1> <arg2> ... <argN>`
  - how to set an environment variable in K8s:
    - set the `env:` property inside `spec:`, which is an array
      - every item under `env:` starts with a dash
      - each item has a `name:` and a `value:` property
    - can also run `docker run -e APP_COLOR=pink simple-webapp-color`


- *ConfigMaps:*
  - if there are lots of pod manifests, it becomes difficult to manage environment variables in the manifests themselves
    - so extract the environment variables and put into a ConfigMap
  - ConfigMaps pass environment variables in the form of key-value pairs
    - when a pod is created, inject the ConfigMap into the pod so the key-value pairs are available to the pod as environment variables for the application hosted inside the container in the pod
  - Two phases:
    - 1. create the ConfigMap
      - Two ways to create a ConfigMap:
        - 1. imperative: 
          - on the CLI: `kubectl create cm <configmap_name> --from-literal=<key>=<value>`
            - use the `--from-literal` syntax for each key-value pair added to the ConfigMap
          - from a file:
            - `kubectl create cm <configmap_name> --from-file=<file_path>`
        - 2. declarative
          - manifest contains 4 items:
            - `apiVersion: v1`, `kind: ConfigMap`, `metadata:`, `data:`
              apiVersion: v1
              kind: ConfigMap
              metadata:
                name: <CM_name> e.g., `app-config`
              data:
                key1: val1
                keyN: valN
    - 2. inject ConfigMap into the pod
      - pass a ConfigMap to a pod using the `envFrom:` property under `spec.containers`
        spec:
          envFrom:
            - configMapRef:
              name: <CM_name_from_configmap.yml> e.g., `app-config`
        `envFrom` is a list
      - single env variable and a volume are also possible ways to inject env vars into a pod
  - `kubectl get cm` will show the key-value env var pairs


- *Secrets:*
  - Two steps to use secrets
    - 1. create secret
      - i. imperative
        - `k create secret generic <secret_name> --from-literal=<key>=<value>`
        - `k create secret generic <secret_name> --from-file=<path_to_secrets_file>`
      - ii. declarative
    - 2. inject secret into pod

- *Encryption at rest:*
  - to enable encryption at rest:
    - create a configuration file
        apiVersion: apiserver.config.k8s.io/v1
        kind: EncryptionConfiguration
        resources:
    - pass it in to `/etc/kubernetes/manifests/kube-apiserver.yml` as an option
      - see https://kubernetes.io/docs/tasks/administer-cluster/encrypt-data/ for more details


- *Multi-container pods:*
  - containers can refer to each other as localhost since they share the same network space
  - can access the same storage volumes
    - don't have to enable volume sharing or services between pods to allow communication between the pods
  - the `containers:` section in the pod manifest is an array, which allows for declaring multiple containers in a single pod


- *OS Upgrades:*
  - if a node is offline for more than five minutes, the pods on that node are considered dead and terminated
    - but if the pods are part of a replica set, then those pods on the offline node are recreated on other nodes
  - pod eviction timeout: the time the master node waits for a pod to come back online
    - default value is 5 mins: `kube-controller-manager --pod-eviction=5m0s`
  - pod comes back online blank without any nodes scheduled on it
  - if a node is offline for less than five minutes, the node can be rebooted and pods will be back on that node
    - so quick maintenance is easy but there's no guarantee that the node will be back up in that time frame
    - safer way:
      - intentionally drain the node of its workloads: `kubectl drain <node_name>`
        - pods are gracefully terminated on one node and recreated on another
      - the node is cordoned, i.e., marked unschedulable until it is uncordoned `kubectl uncordon <node_name>`
        - the pods that were moved to other nodes aren't automatically returned to their original node
          - only newly created pods can be scheduled to go on this restarted node
      - can also cordon a node `k cordon <node_name>`: doesn't drain the node, but marks it unschedulable for all future pods


- *K8s Software Versions:*
  - `k get no` shows the version of K8s that's running on each node
  - K8s uses semantic versioning: Major.Minor.Patch
    - minor releases have new features and functionalities
    - patches have bug fixes
    - bug fixes and improvements go into an alpha release tagged "alpha"
      - features are disabled by default and could be buggy
    - then moved to a beta release where code is well-tested and features are enabled by default
    - then moved to main stable release
  - all controlplane components must have the same version
    - not true for ETCD cluster and CoreDNS
      - can have versions different from controlplane components
      - ETCD and CoreDNS are separate projects


- *Cluster Upgrade:*
  - kube-apiserver is primary component in controlplane
    - the component that all other components communicate with
    - thus, no components should be at a version higher than that of the kube-apiserver
  - controller-manager and kube-scheduler can be up to one version lower than kube-apiserver
  - kubelet and kube-proxy can be up to two versions lower than kube-apiserver
  - Kubernetes only supports the most current minor version (whatever that currently is) and two minor versions below it
  - recommended upgrade approach is to upgrade minor versions one by one
    - the actual process will depend on whether `kubeadm` was used to deploy the cluster or if it was done the "hard way" or if one is using a managed K8s cluster
      - `kubeadm upgrade plan` will show current version for components as well as available versions
        - must manually update kubelet versions on each worker node since kubeadm doesn't install or upgrade kubelets
        - must update kubeadm itself before upgrading cluster
          - follows the same version as K8s
  - 2 steps to upgrade a cluster:
    - i. upgrade master node
      - controlplane components go down briefly, but it doesn't affect worker nodes and the applications on them
      - can't use kubectl or controller-manager, can't deploy new pods or modify existing ones, can't use kube-apiserver during upgrade
    - ii. upgrade worker nodes
      - worker node upgrade strategies:
        - a) upgrade all nodes simultaneously
          - users can't access applications during simultaneous upgrade; requires downtime
        - b) upgrade nodes one at a time
          - pods are migrated to nodes that are not upgrade 
        - c) add temp nodes to the cluster with the newer software version, add workloads to the new nodes and remove the old nodes
  - commands:
    - `cat /etc/*release*` command to see what OS is running on each node
    - `kubeadm upgrade plan`
    - `apt upgrade -y kubeadm=<version>` upgrade kubeadm tool itself
    - `kubeadm upgrade apply <version>`
      - should see message: `[upgrade/successful] SUCCESS! Your cluster was upgraded to "<version>". Enjoy!"`
      - rerunning `kubeadm upgrade plan` should show no new upgrades available
    - `kubect get no` to see what version the nodes are running
    - `kubectl drain controlplane --ignore-daemonsets` if master node is running pods
    - upgrade kubelet on the master node if it has kubelet running: `apt upgrade -y kubelet=<version>`
    - `k get no` should show that controlplane is unschedulable
    - `systemctl daemon-reload`
    - `systemctl restart kubelet`
    - `k get no` should show that if kubelet is running on master, it has been upgraded and is ready; but workers are still on the old version
    - `k uncordon controlplane`
    - SUCCESSFULLY UPGRADED MASTER NODE; NOW MOVE TO WORKER NODES:
    - `apt upgrade -y kubeadm=<version>` on each worker node
    - `apt upgrade -y kubelet=<version>` on each worker node
    - `kubectl drain node <node_name> --ignore-daemonsets` MUST RUN ON MASTER NODE (same for `uncordon`)
    - `k drain <node_name>` if using the one-at-a-time worker node upgrade strategy
    - upgrade kubelet and kubectl on worker nodes, e.g.,
        `\# replace x in 1.27.x-00 with the latest patch version`
        `apt-mark unhold kubelet kubectl && apt-get update && apt-get install -y kubelet=1.27.x-00 kubectl=1.27.x-00 && apt-mark hold kubelet kubectl`
    - `kubeadm upgrade node config --kubelet-version <version>`
    - `systemctl daemon-reload`
    - `systemctl restart kubelet`
    - `kubectl uncordon <node_name>` DONE ON CONTROLPLANE NODE
  - high-level:
    - upgrade CP nodes
      - upgrade kubelet and kubectl on CP nodes 
    - upgrade worker nodes
      - since kubelet is responsible for running pods, must drain node before updating kubelet
      - kubectl upgrades are done on CP node(s)
  - https://kubernetes.io/docs/tasks/administer-cluster/cluster-upgrade/
  - https://kubernetes.io/docs/tasks/administer-cluster/kubeadm/kubeadm-upgrade/
    - change patch version in commands manually (will be listed as `x` in instructions)
  - **THESE COMMANDS ARE NOT FULLY CORRECT; SEE # Cluster Maintenance/Practice Test - Cluster Upgrade Process SOLUTION FOR FULL LIST OF COMMANDS**


- *Backup and Restore Methods:*
  - problem: if some objects were created the imperative way without any documentation, while others were created the declarative way; how would those resources be backed up?
    - `k get all --all-namespaces -o yaml > <all_deploy_services_objects_file.yml>` to query the kube-apiserver and save all resource configurations for all objects created on the cluster as a copy
      - there are solutions such as Velero (ARK/HeptIO)
  - ETCD cluster stores info about the state of the cluster (nodes, all resources in the cluster)
  - instead of backing up resources, can back up ETCD server itself
    - the data directory is specified when configuring ETCD where all data are stored
      - usually `/var/lib/etcd`
      - this directory is what will be backed up by backup tool
    - ETCD also has a built-in snapshot tool: `ETCDCTL_API=3 etcdctl snapshot save <snapshot_name>.db` 
    - view status of the backup: `ETCDCTL_API=3 etcdctl snapshot status <snapshot_name>.db`
    - to restore cluster from the backup: 
      - stop the kube-apiserver service `service kube-apiserver stop`
        - will have to restart ETCD cluster in the process, and the kube-apiserver depends on the ETCD cluster
      - `ETCDCTL_API=3 etcdctl snapshot restore <snapshot_name>.db --data-dir <path_of_backup_file>` to restore the backup
        - when ETCD restores from a backup, it initializes a new cluster configuration and configures the members of ETCD as new members to a new cluster
          - prevents a new member from accidentally joining an existing cluster
          - a new data directory is created
        - configure ETCD configuration file to use the new data directory
      - `systemctl daemon-reload`
      - `service etcd restart`
      - `service kube-apiserver start` and then cluster should be back in the original state
        - for all ETCD commands, you must specify: the certificate files for authentication, the endpoint to the ETCD cluster, the CA certificate, the ETCD server certificate and the key; for example:
        `ETCDCTL_API=3 etcdctl snapshot save snapshot.db --endpoints=https://127.0.0.1:2379 --cacert=/etc/etcd/ca.crt --cert=/etc/etcd/etcd-server.crt --key=/etc/etcd/etcd-server.key`
  - summary: Two options to back up:
    - 1. back up using ETCD
    - 2. back up by querying the kube-apiserver
  - may not have access to the ETCD cluster in a managed K8s service, so backing up by querying the kube-apiserver is likely the better option


- *Backup and Restore Methods:*
  - controlling access to the API server is the first line of defense
  - authentication: who can access the API server
  - authorization: what authenticated users can do
  - by default, all pods can access all other pods in the cluster
    - can restrict pods' access using network policies



# Work-related notes:

- which of our applications need redundancy, i.e., ReplicaSets to ensure high availability?

- which of our containers must be running on the same pod (see K8s Up and Running book for considerations)?



# Outstanding Questions:

- how/where to configure `etcd.service`?
  - `vim /etc/systemd/system/etcd.service`?