Kubernetes : Configure Control Plane Node2022/11/02
 	
Install Kubeadm to Configure Multi Nodes Kubernetes Cluster.
This example is based on the environment like follows.
-----------+---------------------------+--------------------------+------------
           |                           |                          |
       eth0|10.0.0.25              eth0|10.0.0.71             eth0|10.0.0.72
+----------+-----------+   +-----------+-----------+   +-----------+-----------+
|  [ ctrl.srv.world ]  |   |  [snode01.srv.world]  |   |  [snode02.srv.world]  |
|     Control Plane    |   |      Worker Node      |   |      Worker Node      |
+----------------------+   +-----------------------+   +-----------------------+

[1]	
Configure pre-requirements on all Nodes, refer to here.
[2]	
Configure initial setup on Control Plane Node.
For [control-plane-endpoint], specify the Hostname or IP address that Etcd and Kubernetes API server are run.
For [--pod-network-cidr] option, specify network which Pod Network uses.
There are some plugins for Pod Network. (refer to details below)
  ⇒ https://kubernetes.io/docs/concepts/cluster-administration/networking/
On this example, it uses Calico.
root@ctrl:~# kubeadm init --control-plane-endpoint=10.0.0.25 --pod-network-cidr=192.168.0.0/16 --cri-socket=unix:///run/containerd/containerd.sock
[init] Using Kubernetes version: v1.25.3
[preflight] Running pre-flight checks
[preflight] Pulling images required for setting up a Kubernetes cluster
[preflight] This might take a minute or two, depending on the speed of your internet connection
[preflight] You can also perform this action in beforehand using 'kubeadm config images pull'
[certs] Using certificateDir folder "/etc/kubernetes/pki"
[certs] Generating "ca" certificate and key
[certs] Generating "apiserver" certificate and key
[certs] apiserver serving cert is signed for DNS names [ctrl.srv.world kubernetes kubernetes.default kubernetes.default.svc kubernetes.default.svc.cluster.local] and IPs [10.96.0.1 10.0.0.25]
[certs] Generating "apiserver-kubelet-client" certificate and key
[certs] Generating "front-proxy-ca" certificate and key
[certs] Generating "front-proxy-client" certificate and key
[certs] Generating "etcd/ca" certificate and key
[certs] Generating "etcd/server" certificate and key
[certs] etcd/server serving cert is signed for DNS names [ctrl.srv.world localhost] and IPs [10.0.0.25 127.0.0.1 ::1]
[certs] Generating "etcd/peer" certificate and key
[certs] etcd/peer serving cert is signed for DNS names [ctrl.srv.world localhost] and IPs [10.0.0.25 127.0.0.1 ::1]
[certs] Generating "etcd/healthcheck-client" certificate and key
[certs] Generating "apiserver-etcd-client" certificate and key
[certs] Generating "sa" key and public key
[kubeconfig] Using kubeconfig folder "/etc/kubernetes"
[kubeconfig] Writing "admin.conf" kubeconfig file
[kubeconfig] Writing "kubelet.conf" kubeconfig file
[kubeconfig] Writing "controller-manager.conf" kubeconfig file
[kubeconfig] Writing "scheduler.conf" kubeconfig file
[kubelet-start] Writing kubelet environment file with flags to file "/var/lib/kubelet/kubeadm-flags.env"
[kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
[kubelet-start] Starting the kubelet
[control-plane] Using manifest folder "/etc/kubernetes/manifests"
[control-plane] Creating static Pod manifest for "kube-apiserver"
[control-plane] Creating static Pod manifest for "kube-controller-manager"
[control-plane] Creating static Pod manifest for "kube-scheduler"
[etcd] Creating static Pod manifest for local etcd in "/etc/kubernetes/manifests"
[wait-control-plane] Waiting for the kubelet to boot up the control plane as static Pods from directory "/etc/kubernetes/manifests". This can take up to 4m0s
[apiclient] All control plane components are healthy after 14.502196 seconds
[upload-config] Storing the configuration used in ConfigMap "kubeadm-config" in the "kube-system" Namespace
[kubelet] Creating a ConfigMap "kubelet-config" in namespace kube-system with the configuration for the kubelets in the cluster
[upload-certs] Skipping phase. Please see --upload-certs
[mark-control-plane] Marking the node ctrl.srv.world as control-plane by adding the labels: [node-role.kubernetes.io/control-plane node.kubernetes.io/exclude-from-external-load-balancers]
[mark-control-plane] Marking the node ctrl.srv.world as control-plane by adding the taints [node-role.kubernetes.io/control-plane:NoSchedule]
[bootstrap-token] Using token: sx7691.vhnz9i9szyqecy6e
[bootstrap-token] Configuring bootstrap tokens, cluster-info ConfigMap, RBAC Roles
[bootstrap-token] Configured RBAC rules to allow Node Bootstrap tokens to get nodes
[bootstrap-token] Configured RBAC rules to allow Node Bootstrap tokens to post CSRs in order for nodes to get long term certificate credentials
[bootstrap-token] Configured RBAC rules to allow the csrapprover controller automatically approve CSRs from a Node Bootstrap Token
[bootstrap-token] Configured RBAC rules to allow certificate rotation for all node client certificates in the cluster
[bootstrap-token] Creating the "cluster-info" ConfigMap in the "kube-public" namespace
[kubelet-finalize] Updating "/etc/kubernetes/kubelet.conf" to point to a rotatable kubelet client certificate and key
[addons] Applied essential addon: CoreDNS
[addons] Applied essential addon: kube-proxy

Your Kubernetes control-plane has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

Alternatively, if you are the root user, you can run:

  export KUBECONFIG=/etc/kubernetes/admin.conf

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

You can now join any number of control-plane nodes by copying certificate authorities
and service account keys on each node and then running the following as root:

  kubeadm join 10.0.0.25:6443 --token sx7691.vhnz9i9szyqecy6e \
        --discovery-token-ca-cert-hash sha256:6166ac8d73379081c2ca2b9625480b3f2cc992f571659e776b68d676e84cc3d9 \
        --control-plane

Then you can join any number of worker nodes by running the following on each as root:

# the command below is necessary to run on Worker Node when it joins to the cluster, so remember it
kubeadm join 10.0.0.25:6443 --token sx7691.vhnz9i9szyqecy6e \
        --discovery-token-ca-cert-hash sha256:6166ac8d73379081c2ca2b9625480b3f2cc992f571659e776b68d676e84cc3d9

# set cluster admin user
# if you set common user as cluster admin, login with it and run [sudo cp/chown ***]
root@ctrl:~# mkdir -p $HOME/.kube
root@ctrl:~# cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
root@ctrl:~# chown $(id -u):$(id -g) $HOME/.kube/config
[3]	Configure Pod Network with Calico.
root@ctrl:~# wget https://raw.githubusercontent.com/projectcalico/calico/master/manifests/calico.yaml
root@ctrl:~# kubectl apply -f calico.yaml
poddisruptionbudget.policy/calico-kube-controllers created
serviceaccount/calico-kube-controllers created
serviceaccount/calico-node created
configmap/calico-config created
customresourcedefinition.apiextensions.k8s.io/bgpconfigurations.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/bgppeers.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/blockaffinities.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/caliconodestatuses.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/clusterinformations.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/felixconfigurations.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/globalnetworkpolicies.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/globalnetworksets.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/hostendpoints.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/ipamblocks.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/ipamconfigs.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/ipamhandles.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/ippools.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/ipreservations.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/kubecontrollersconfigurations.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/networkpolicies.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/networksets.crd.projectcalico.org created
clusterrole.rbac.authorization.k8s.io/calico-kube-controllers created
clusterrole.rbac.authorization.k8s.io/calico-node created
clusterrolebinding.rbac.authorization.k8s.io/calico-kube-controllers created
clusterrolebinding.rbac.authorization.k8s.io/calico-node created
daemonset.apps/calico-node created
deployment.apps/calico-kube-controllers created

# show state : OK if STATUS = Ready
root@ctrl:~# kubectl get nodes
NAME             STATUS   ROLES           AGE     VERSION
ctrl.srv.world   Ready    control-plane   9m35s   v1.25.3

# show state : OK if all pods are Running
root@ctrl:~# kubectl get pods -A
NAMESPACE     NAME                                       READY   STATUS    RESTARTS   AGE
kube-system   calico-kube-controllers-59697b644f-fgfqp   1/1     Running   0          2m32s
kube-system   calico-node-z8vgs                          1/1     Running   0          2m32s
kube-system   coredns-565d847f94-v8v2x                   1/1     Running   0          10m
kube-system   coredns-565d847f94-zh4rd                   1/1     Running   0          10m
kube-system   etcd-ctrl.srv.world                        1/1     Running   0          11m
kube-system   kube-apiserver-ctrl.srv.world              1/1     Running   0          11m
kube-system   kube-controller-manager-ctrl.srv.world     1/1     Running   0          11m
kube-system   kube-proxy-lnvhj                           1/1     Running   0          10m
kube-system   kube-scheduler-ctrl.srv.world              1/1     Running   0          11m
