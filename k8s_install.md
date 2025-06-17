# создадим папку, если наша ubuntu ниже 22
sudo mkdir -p -m 755 /etc/apt/keyrings
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.29/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.29/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list

sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl containerd
sudo apt-mark hold kubelet kubeadm kubectl containerd

kubeadm init
# ждём, пока скачаются нужные контейнеры ... и ...
.......
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

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join 161.35.85.51:6443 --token d6doyb.khhen11z33sa01uf \
        --discovery-token-ca-cert-hash sha256:b61fc4a7f0f09a2df827051c0b10d05e7eec74c7ade02a26e1eff698a4ad74f1

# проверим состояние контейнерного движка
sudo crictl ps

CONTAINER           IMAGE               CREATED              STATE               NAME                      ATTEMPT             POD ID              POD
a929766bc2156       43c6c10396b89       About a minute ago   Running             kube-proxy                0                   1a3121b357447       kube-proxy-8sdqv
61e767ea34e75       a0eed15eed449       2 minutes ago        Running             etcd                      0                   0e382564dd834       etcd-node1
8a6a5d2124b75       53b148a9d1963       2 minutes ago        Running             kube-apiserver            0                   06672d6d9d3a1       kube-apiserver-node1
7403d3a8adec4       406945b511542       2 minutes ago        Running             kube-scheduler            0                   f682921e01168       kube-scheduler-node1
de827554f7919       79d451ca186a6       2 minutes ago        Running             kube-controller-manager   0                   072747505e91d       kube-controller-manager-node1

# скопируем себе в директорию kubeconfig (в отличие от minikube, тут у нас самооблуживание)

mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config

# проверим состояние компонентов kubernetes

kubectl get pods -n kube-system
NAMESPACE     NAME                            READY   STATUS    RESTARTS   AGE
kube-system   coredns-76f75df574-lbbq4        0/1     Pending   0          15m
kube-system   coredns-76f75df574-pbrkz        0/1     Pending   0          15m
kube-system   etcd-node1                      1/1     Running   0          15m
kube-system   kube-apiserver-node1            1/1     Running   0          15m
kube-system   kube-controller-manager-node1   1/1     Running   0          15m
kube-system   kube-proxy-8sdqv                1/1     Running   0          15m
kube-system   kube-scheduler-node1            1/1     Running   0          15m

# вот тут ошибочка вышла. Заранее скажем, что проблема в том, что после выполнения kubeadm init нужно выполнить ещё одно действие
# кстати, это написано в результатах работы команды kubeadm init. Кто заметил, тому 5 за внимательность.
# проверим статус самой ноды
kubectl get nodes
NAME    STATUS     ROLES           AGE   VERSION
node1   NotReady   control-plane   17m   v1.29.1
# нода не готова, т.к. мы не применили никакой сетевой плагин. В качестве примера установим calico

kubectl apply -f https://raw.githubusercontent.com/projectcalico/calico/v3.26.3/manifests/calico.yaml

poddisruptionbudget.policy/calico-kube-controllers created
serviceaccount/calico-kube-controllers created
serviceaccount/calico-node created
serviceaccount/calico-cni-plugin created
configmap/calico-config created
customresourcedefinition.apiextensions.k8s.io/bgpconfigurations.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/bgpfilters.crd.projectcalico.org created
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
clusterrole.rbac.authorization.k8s.io/calico-cni-plugin created
clusterrolebinding.rbac.authorization.k8s.io/calico-kube-controllers created
clusterrolebinding.rbac.authorization.k8s.io/calico-node created
clusterrolebinding.rbac.authorization.k8s.io/calico-cni-plugin created
daemonset.apps/calico-node created
deployment.apps/calico-kube-controllers created

# подождём около минуты
sudo kubectl --kubeconfig /etc/kubernetes/admin.conf get nodes
NAME    STATUS   ROLES           AGE   VERSION
node1   Ready    control-plane   19m   v1.29.1

# проверим состояние компонентов kubernetes
kubectl get pods -n kube-system
NAMESPACE     NAME                                       READY   STATUS    RESTARTS   AGE
kube-system   calico-kube-controllers-558d465845-mzsln   1/1     Running   0          5m4s
kube-system   calico-node-rpg6s                          1/1     Running   0          5m4s
kube-system   coredns-76f75df574-lbbq4                   1/1     Running   0          22m
kube-system   coredns-76f75df574-pbrkz                   1/1     Running   0          22m
kube-system   etcd-node1                                 1/1     Running   0          22m
kube-system   kube-apiserver-node1                       1/1     Running   0          22m
kube-system   kube-controller-manager-node1              1/1     Running   0          22m
kube-system   kube-proxy-8sdqv                           1/1     Running   0          22m
kube-system   kube-scheduler-node1                       1/1     Running   0          22m