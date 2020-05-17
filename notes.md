# Certified Kubernetes Administrator Notes

### Application Lifecycle Management (8%)

* Understand Deployments, ReplicaSets, StatefulSets, 
* Commands to know, rollout. apply, expose, replace, set
* spec.minReadySeconds - how long should pod be ready before update of deployment. Combine with readiness probe to failed before minReadySeconds
* Pass environment variables as ConfigMaps or Secrets
* StatefulSets must use headless service.
  

### Installation, Configuration & Validation (12%)

* Not required to build cluster
* Everything but kubelet can be run as a Pod
* Certificate generation
  - CA
    - `$ openssl genrsa -out ca.key 2048`
    - `$ openssl req -new -key ca.key -subj "/CN=KUBERNETES-CA" -out ca.csr`
    - `$ openssl x509 -req -in ca.csr -key ca.key -CAcreateserial -out ca.crt -days 1000`
  - Admin 
    - `$ openssl genrsa -out admin.key 2048`
    - `$ openssl req -new -key admin.key -subj="/CN=Admin/O=system:masters" -out admin.csr`
    - `$ openssl x509 -req -in admin.csr -CA ca.crt -CAkey ca.key -CAcreateserial -out admin.crt -days 1000`
  - kube-controller-manager
    - `$ openssl genrsa -out kube-controller-manager.key 2048`
    - `$ openssl req -new -key kube-controller-manager.key -subj="/CN=system:kube-controller-manager" -out kube-controller-manager.csr`
    - `$ openssl x509 -req -in kube-controller-manager.csr -CA ca.crt -CAkey ca.key -CAcreateserial -out kube-controller-manager.crt -days 1000`
  - kube-proxy
    - `$ openssl genrsa -out kube-proxy.key 2048`
    - `$ openssl req -new -key kube-proxy.key -subj="/CN=system:kube-proxy" -out kube-proxy.csr`
    - `$ openssl x509 -req -in kube-proxy.csr -CA ca.crt -CAkey ca.key -CAcreateserial -out kube-proxy.crt -days 1000`
  - kube-scheduler
    - `$ openssl genrsa -out kube-scheduler.key 2048`
    - `$ openssl req -new -key kube-scheduler.key -subj="/CN=system:kube-scheduler" -out kube-scheduler.csr`
    - `$ openssl x509 -req -in kube-scheduler.csr -CA ca.crt -CAkey ca.key -CAcreateserial -out kube-scheduler.crt -days 1000`
  -  
* systemd: manages system services, a single service is typically known as a unit and is initialized from a `.service` file.
   - status of systemd service 
     - `$ systemctl status nginx.service`
   - start a systemd service
     - `$ sudo systemctl start nginx.service`
   - stop a systemd service
     - `$ sudo systemctl stop nginx.service  `
   - reload a systemd service
     - `$ sudo systemctl reload nginx.service`
   - enable a systemd service 
     - `$ sudo systemctl enable nginx.service`  
   - show a unit file for systemd service
     - `$ systemctl show nginx.service`
   - edit a unit file of existing systemd service
     - `$ systemctl edit --full nginx.service`
* journald: log information daemon that collects systemd services logs
   - logs of a service
     - `$ journalctl -u nginx.service` 
* kubeadm: kubenetes administrator cli tool
   - initialize control node
     - `$ kubeadm init --pod-network-cidr=1234/12`
* Kubeadm stores kubernetes components in etc/kubernetes/manifest/ and non kubeadm is stored etc/systemd/system/
* Kubeadm does not automatically install kubelet


### Core Concepts (19%)
* ResourceQuotas Pods Deployments, .etc 
* Not many notes for me as much of it is rollover from CKAD
### Networking (11%)

* Ports for Components
  - api-server:6443
  - kubelet:10250
  - kube-scheduler:10251
  - kube-controller-manager:10252 
  - etcd:2379
* commands for debugging
  - `$ ip link`, `$ ip link show dev em1`
    - display and change the state of network interfaces.
    - identify networking interface for cluster connectivity (ens3)
    - show MAC address on the ens3 network interface (ip link show dev ens3)
  - `$ ip addr`
    - display IP Addresses and property information assigned to network interfaces.
    - check network range of cluster nodes
  - `$ ip route` or `route`
    - view routing table on the host.
    - check default route for dns resolution (ip route show default)
  - `$ arp`
    - mapping of IP addresses to MAC addresses
    - check MAC accress of a node 
  - `$ nslookup`
    - query DNS server to resolve domain name
    - check if k8s DNS server (like Core DNS) is working correctly
  -  `$ dig`
     - query DNS server to resolve domain name, similar to nslookup but returns more details
     - check if k9s DNS server (like Core DNS) is working correctly
  - `$ ps -aux | grep <service-name>`
    - find paths to certificates and other settings for CNI etc `ps -aux | grep kubelet`
  - `$ netstat -plnt`
    - displays network connections (both incoming and outgoing), routing tables, and a number of network interface (network interface controller or software-defined network interface) and network protocol statistics
  - /etc/cni/net.d
    - cni settings
* kube-proxy
  - service cluster ip range and pod cluster ip range shouldnt over lap 
  - creates ip tables to forward service ips to pod ips
* DNS in K8S
  - pods /etc/resolv.conf should point to dns server ip
  - config is /etc/coredns/Corefile passed in as configmap
  - sets cluster.local as top level domain name
  - kubelet config has clusterDNS ip (coredns or kube-dns)
### Scheduling (5%)
* Manually schedule using nodeName in pod spec.
* Taints and tolerations prevent pods from going to some nodes that don't tolerate there taint.
* Node Selector schedule pods to nodes with selected labels.
* Node Affinities can be required or preferred and they can kick off pods during execution.
* To guarantees only certain pods go to certain nodes must use taints/tolerations and node affinities 
* Namespaces LimitRange resources
* Static Pods are pods that are created by Kubelet without help from control plane components. Just place manifests in /etc/kubernetes/manifests(with default kubelet manifest path) if not there check in config yaml passed into kubelet.service file, config yaml will have staticPodPath parameter in config.yaml. To check if pod is up you must use docker ps assuming control plane components are down. If api-server is up it will have metadata on pod. This is how kubeadm deploys control plane components.
* A second scheduler can be deployed by deploying another schedule service using a .service file or kubeadm deploy another scheduler. Specify schedulerName to use second scheduler in pod spec. Specify different name when instantiating sheduler(in pod and command field for kubeadm in systemd just use different name). Also in kubeadm tool mark leader-elect false but container name same, different pod name. View events to determine which scheduler was used. https://kubernetes.io/docs/tasks/administer-cluster/configure-multiple-schedulers/
  
  - kubeadm
    - in new scheduler pod spec rename pod name and for commands in container rename --scheduler-name and --leader-elect false unless multi master set to true and --lock-object-name=scheduler-name
  - systemd
    - just create another .service file for your scheduler with new scheduler name and config 
### Security (12%)
* Authentication Authorization Certificates and Network Policies
* Authentication 
  - static password file: --basic-auth-file in csv when creating api-server, specify user and password on request
  - static token file: --token-auth-file in csv, specify token in request
  - certificates 
  - third party tools
* Certificates 
  - Components in k8s that require certificates
    - Client certs 
      - scheduler
      - controller-manager
      - kube-proxy
      - apiserver-kubelet
      - apiserver-etcd 
      - kubelet-client
    - Server certs
      - etcd
      - api-server
      - kubelet
  - "The Hard Way" cluster
    - find existing certs
      - `$ cat /etc/systemd/system/kube-apiserver.service` 
    - look at service logs using journalctl to determine if there is a service bug 
      - `$ journalctl -u etcd.service -l` 
  - kubeadm cluster
    - find existing certs  
      - `$ cat /etc/kubernetes/manifests/kube-apiserver.yaml` 
    - look at pod logs to determine bugs 
  - command to describe certs
    -  `$ openssl x509 -in /etc/kubernetes/pki/apiserver.crt -text -noout`
  - command to sign csr
    - `$ openssl x509 -req -in /t.csr -CA /ca.crt -CAkey /ca.key -CAcreateserial -out /apiserver-etcd-client.crt` 
  - Fields to check for bugs 
    - Issuer
    - CN (certificate Name)
    - Expiration 
    - SAN (subject alternative names)
      - IPs
      - kubernetes DNS names
* Kubeconfigs 
  - defining what user should access what cluster
  - sections 
    - Clusters
      - provide full ca cert path
    - Contexts: what use for what cluster
    - Users
    - Current-context
* Authorization
  - know RBAC both cluster and namespace specific, service accounts and users
### Cluster Maintenance (11%)
* For upgrades make sure to drain a node then following upgrade make sure to uncordon it. Cordon makes node unschedulable.
* Make sure if kube-apiversion is at version x everything else(other components) is a version lower than the api-server (disregard kubectl)
* Only upgrade minor version at a time not skip versions to another version
* Steps to upgrade master nodes with kubeadm:
  - `$ kubeadm upgrade plan` 
  - `$ apt-get upgrade -y kubeadm=1.12.0-00`
  - `$ kubeadm upgrade apply v1.12.0`
  - `$ apt-get upgrade -y kubelet=1.12.00-00` (if kubelet on master)
  - `$ kubectl get nodes`
* Steps to upgrade worker nodes with kubeadm: 
  - `$ kubectl drain node-1`
  - `$ apt-get upgrade -y kubeadm=1.12.0-00`
  - `$ kubeadm upgrade node config --kubelet-version v1.12.0`
  - `$ systemctl restart kubelet` 
  - `$ kubectl uncordon node-1`
* Backing up ETCD
  - `$ export ETCDCTL_API=3`  
  - `$ etcdctl member list --endpoints=https://127.0.0.1:2379 --cacert=/etc/etcd/ca.crt --cert=etc/etcd/etcd-server.crt --key=/etc/etcd/etcd-server.key` (check if arguments are correct)
  - `$ etcdctl snapshot save snapshot.db --endpoints=https://127.0.0.1:2379 --cacert=/etc/etcd/ca.crt --cert=etc/etcd/etcd-server.crt --key=/etc/etcd/etcd-server.key`
  - `$ etcdctl snapshot status snapshot.db` (check snapshot for successful save)
  - `$ service kube-apiserver stop`
  - `$ etcdctl snapshot restore --endpoints=https://127.0.0.1:2379 --cacert=/etc/etcd/ca.crt --cert=etc/etcd/etcd-server.crt --key=/etc/etcd/etcd-server.key --data-dir --initial-cluster --name --initial-advertise-peer-urls --initial cluster-token snapshot.db `
  - then fix static pod for etcd data dir and initial cluster token(make sure to switch mount path of data-dir initial cluster token)
  - new cluster token and data dir but all configs same as original
  - `$ systemctl daemon-reload`
  - `$ service etcd restart`
  - `$ service kube-apiserver start`
### Logging/Monitoring (5%)
* Not much to this section just know logs and logs -c for container specific.
### Storage (7%)

### Troubleshooting (10%)

