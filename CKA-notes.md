# Notes:

- *kube-apiserver:*
  - all user access is managed by kube-apiserver
    - both types of requests go through kube-apiserver:
      - kubectl
      - access the API directly: `curl https://<kube_server_IP>:<port>`


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
  - CertificateSigningRequest


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


- *Two ways to limit pods to run on specific nodes:*
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


- *K8s Security Primitives:*
  - controlling access to the API server is the first line of defense
  - authentication: who can access the API server
  - authorization: what authenticated users can do
  - by default, all pods can access all other pods in the cluster
    - can restrict pods' access using network policies


- *Authentication:*
  - K8s doesn't manage user accounts natively
    - cannot create users
      - relies on external source to manage users:
        - file with user details
        - certificates
        - third party identity service, e.g., LDAP
    - but K8s can manage service accounts:
      - `k create sa <sa_name>`
  - kubeapi-server authenticates a request (either through `kubectl` or by accessing the API directly) before processing it
  - 4 authentication mechanisms:
    - i. static password file
      - RESUME HERE: AUTHENTICATION lesson
    - ii. static token file
    - iii. certificates
    - iv. third party auth protocols, e.g., LDAP, Kerberos


- *Certificates & TLS Basics:* 
  - admin uses its key pair to secure SSH connectivity to a server
  - the server uses its key pair to secure HTTPS traffic
    - 1. server sends a certificate signing request to a certificate authority
    - 2. the CA uses its private key to sign the CSR
      - all users have a copy of the CA's public key
    - 3. the signed certificate (root certificate) is sent back to the server
    - 4. the server configures the web application with the signed certificate (serving certificate)
    - 5. whenever a user accesses the web application, the server sends the certificate with its public key
    - 6. the user's browser reads the certificate and uses the validate and retrieve the server's public key
    - 7. the user's browser generates a symmetric key that it uses to encrypt all future communication 
    - 8. the user's symmetric key is encrypted using the server's public key and sent back to the server 
    - 9. the server uses its private key to decrypt the message and retrieve the symmetric key, which is used to encrypt all future traffic
    - 10. user uses its username and password to authenticate to the server
    - 11. to establish trust with the client, the server requests a certificate from the client (client certificate)
    - 12. the client generates a pair of keys and a signed certificate from a valid CA
    - 13. the client sends the certificate to the server to prove the client is who the client says it is
      - TLS client certificates are not generally implemented on web servers
        - and if they are, they are generated behind the scenes so a client doesn't need to generate or manage certificates manually


- *Viewing Certificate Details:*
  - "the hard way"
    - deploy all components as native services
    - look at logs using `journalctl -u etcd.service -l`
  - kubeadm
    - deploys all components as pods
    - find the certificates used in the kube-apiserver definition file at `/etc/kubernetes/manifests/kube-apiserver.yaml`
    - run `openssl x509 -in <certificate_file_path> -text -noout` to decode the certificate and view its details
    - view logs using `k logs <pod_name>`
      - if core components, such as kube-apiserver or etcd-server, are down, then going down a level to fetch logs as `kubectl` won't work
        - `docker ps -a`
        - `docker logs <container_ID>`
  - `tls-cert-file` and `tls-private-key-file` are for the kube-apiserver itself
  - for the kube-apiserver to connect to ETCD as a client: `etcd-cafile`, `etcd-certfile` and `etcd-keyfile` in `/etc/kubernetes/manifests/kube-apiserver.yaml`
  - if `kubectl` commands don't work, then the `kubectl` utility isn't able to communicate with `kube-apiserver`
  - ETCD port is 2379
  - run `crictl` commands in place of `docker` for environments running CRI-O instead of Docker


- *Certificates API:*
  - Kubernetes has a built-in API that manages certificates and signing requests, and rotate certificates when they expire in an automated way
    - send a certificate signing request directly to Kubernetes via an API call
    - the administrator of the cluster creates a Kubernetes API object called CertificateSigningRequest 
      - cluster administrators can see all certificate signing requests
      - use `kubectl` commands to review and approve certificate signing requests
      - can share certs with users
  - process:
    - 1. user creates a key: `openssl genrsa -out <key_name>.key <num_of_bits>`
    - 2. user sends request to administrator: `openssl req -new -key <key_name>.key -subj "/CN=<common_name>" -out <csr_name>.csr`
    - 3. administrator creates a certificate signing request object using the key
      - created from a manifest file: `kind: CertificateSigningRequest`
      - `spec:` section:
        - groups: specify groups user should be a part of
        - usages: list usages for the account
        - request: the user's certificate signing request in base64
  - commands:
    - `k get csr` so an administrator can see all certificate signing requests
    - `k certificate approve <CSR_name>` to approve a request
    - `k get csr jane -o yaml` to view the certificate
  - K8s signs the certificate using the CA key pairs and generates a certificate for the user, which can then be shared with the user
  - the Controller Manager handles all certificate-related operations
  - the CA's root certificate and private key are needed to sign any certificates
    - the Controller Manager service configuration has two options where the CA's root certificate and private key can be listed
      - `/etc/kubernetes/manifests/kube-controller-manager.yaml`
      - `--cluster-signing-cert-file`
      - `--cluster-signing-key-file`
  - sample CSR manifest:
    apiVersion: certificates.k8s.io/v1
    kind: CertificateSigningRequest
    metadata:
      name: <name>
    spec:
      signerName: kubernetes.io/kube-apiserver-client
      groups:
      - <groups>
      usages:
      - digital signature
      - key encipherment
      - server auth
      request:
        run `cat <CSR_filename>.csr | base64 | tr -d "\n"` and copy output here


- *Authorization:*

- *Role-Based Access Controls:*

- *Cluster Roles:*

- *Service Accounts:*

- *Image Security:*

- *Security in Docker:*

- *Security Contexts:*

- *Network Policies:*

- *Developing Network Policies:*

- *Storage in Docker:*

- *Container Storage Interface:*

- *Volumes:*

- *Persistent Volumes:*

- *Persistent Volume Claims:*

- *Storage Class:*









# Outstanding Questions:

- how/where to configure `etcd.service`?
  - `vim /etc/systemd/system/etcd.service`?