# Введение в k8s

* Запустить minikube

  ```bash
  minikube start
  ```

* Установить dashboard addon, создать и настроить сервисный аккаунт, и получить для него токен

  ```bash
  kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.0.0-beta1/aio/deploy/recommended.yaml
  kubectl apply -f bootstrap/k8s/admin-user.yaml
  kubectl apply -f bootstrap/k8s/cluster-role-binding.yaml
  kubectl -n kube-system describe secret $(kubectl -n kube-system get secret | grep admin-user | awk '{print $1}')
  ```

* Удалить все контейнеры

  ```bash
  minikube ssh
  docker ps
  docker rm -f $(docker ps -a -q)
  ```

* Удалить все pod из kube-system namespace

  ```bash
  kubectl delete pod --all -n kube-system
  ```

* apiserver и т. д. не будут удалены, т.к. это **static pod**, который загружается из **/etc/kubernetes/manifests** и он управляется напрямую **kubelet**
  (это легко проверить, если в этот каталог подложить манифест какого-нибудь произвольного pod - в данном примере, это adapter.yaml и
  перезапустить **kubelet**, после чего этот pod тоже перестанет удаляться)

  ```bash
  systemctl show kubelet | grep ExecStart
  ExecStart={ path=/usr/bin/kubelet ; argv[]=/usr/bin/kubelet --authorization-mode=Webhook --bootstrap-kubeconfig=/etc/kubernetes/bootstrap-kubelet.conf --cgroup-driver=cgroupfs --client-ca-file=/var/lib/minikube/certs/ca.crt   --cluster-dns=10.96.0.10 --cluster-domain=cluster.local --container-runtime=docker --fail-swap-on=false --hostname-override=minikube --kubeconfig=/etc/kubernetes/kubelet.conf --pod-manifest-path=/etc/kubernetes/manifests ; ignore_errors=no ;   start_time=[Tue 2019-07-09 15:29:06 UTC] ; stop_time=[n/a] ; pid=3584 ; code=(null) ; status=0/0 }

  kubelet --help | grep pod-manifest-path
        --pod-manifest-path string                                                                                  Path to the directory containing static pod files to run, or the path to a single static pod file. Files starting with dots will   be ignored. (DEPRECATED: This parameter should be set via the config file specified by the Kubelet's --config flag. See https://kubernetes.io/docs/tasks/administer-cluster/kubelet-config-file/ for more information.)
  ```

  ```bash
  # ls -1 /etc/kubernetes/manifests/
  adapter.yaml
  addon-manager.yaml.tmpl
  etcd.yaml
  kube-apiserver.yaml
  kube-controller-manager.yaml
  kube-scheduler.yaml
  ```

* coredns не будет удален, т.к. управляется через Deployment, который создает соответствующий ReplicaSet

  ```bash
  kubectl get deployment  -n kube-system
  NAME      READY   UP-TO-DATE   AVAILABLE   AGE
  coredns   2/2     2            2           7d5h

  kubectl get rs  -n kube-system
  NAME                 DESIRED   CURRENT   READY   AGE
  coredns-5c98db65d4   2         2         2       7d5h
  ```

* kube-proxy управляется DaemonSet controller, поэтому тоже не будет удален

  ```bash
  kubectl get ds -n kube-system
  NAME         DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR                 AGE
  kube-proxy   1         1         1       1            1           beta.kubernetes.io/os=linux   7d5h
  ```

* Создать docker образ web и запушить его в dockerhub

  ```bash
  docker build -t loktionovam/web:1.17.1-alpine kubernetes-intro/web
  docker push loktionovam/web:1.17.1-alpine
  ```

* Запустить pod для докер образа web

  ```bash
  kubectl apply -f kubernetes-intro/web/web-pod.yaml
  kubectl get pods
  kubectl get pod web -o yaml
  kubectl describe pod web
  ```
