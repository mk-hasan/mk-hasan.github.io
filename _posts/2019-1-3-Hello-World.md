---
layout: post
title: Deploying YOLO Object Detection Model With DeepLearning 4 Java & Apache Flink in Kubernets Cluster Using Raspberry PI!
---

> I was trying to test how object detection model working on kuberenets cluster with apache flink. As i love to do everything in java rather than python, i wanted to give a shot for DeepLearning4j API. This API has been built for running various deep learning model on top of Java. it works pretty good. I set up a kubernets cluster to run the Apache Flink and deploy the YOLO model with flink configuration to see how it works. Let's move on to step by step process. 

## Kubernets Cluster

For Kuberenets cluster i have used severel Raspberry Pi to make the cluster works. First of all, i have used HypriotOS for firing up the RasPi's. It has build in docker support. I have kept all the RasPI's in same network, so that all the device can communicate with each other easily. After setting up the netwrok i moved for initializing the kubernets cluster. To initialize the kuberenets cluster you have to go through number of commands. You can get these commands from Kubernets official website or you can use from here.

>Note: For master node your RasPI has to have at least 2 GB of RAM and 2 CPU Core. Otherwise it will not work from Kubernets version 1.10.

As i have used normal RasPI with 1GB of RAM and 1 CPU, i have initialiazed master node with kuberenets version 1.9.0. This is the last kuberenets version which works with 1GB RAM Master Node.

Before initialization you need to install necessary dependencies like docker, kubelet, kubeadm, kubectl. 

> curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
echo "deb http://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list
sudo apt-get update -q
>sudo apt-get install -qy --force-yes kubeadm=1.9.6-00 kubectl=1.9.6-00 kubelet=1.9.6-00

>sudo aptitude install docker-ce=17.12.1~ce-0~raspbian

Make sure to do the swapoff "sudo swapoff -a" if you want to make cluster using ubuntu machine.

### Initialize the Master Node:
>kubeadm init --pod-network-cidr=172.16.0.0/16 --apiserver-advertise-address=192.168.1.2

In above command , you have to give your master node ip address , though it is not necessary. As i ahve used flannel network , i have mentioned this parameters. After entering this command you will get the kuberenets initialization message with join command for slave nodes. With this join command you can join any number of slave nodes you want. We will talk about this later.

### Configure the master node:

You will get some commands after initializing the master node, using thiscommand you can configure the master node.

> mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

For communicating between the pods in cluster we need to configure the network. Later I have used weave net for pod networking rather than flannel for few reasons. Though you can use flannel also.
>Kubernet pod networking activation: 
kubectl apply -f "https://cloud.weave.works/k8s/net?k8s-version=$(kubectl version | base64 | tr -d '\n')"


### Now we can join other slave nodes to the cluster with master nodes.
For that we need to again install all the necessary dependencies like docker, kubeadm as i mentioned the command earlier. Run these command again to all of your slave nodes. Then run the join command to you every slave nodes...

>kubeadm join --token fd1017.8591d160fa6cec7c 192.168.1.3:6443 --discovery-token-ca-cert-hash sha256:875cc0af9878ef860f2d36c916b31f862b458f5455d1a73488856fb6b108cb7c

You can recreate your join command any time by using this command.
>kubeadm token create --print-join-command

Now you will see the message that node has joined to master node in the cluster after running the join command in every slave node. 

Now if run the follwing command in kubernets master node, you will be able to see all the nodes in the cluster.
>kubectl get nodes

You can algo get all the pods using following command..
>kubectl get pods

If you want to see the log or want to see the details of pods,
>kubectl log "pod-name"
>kubectl describe pod "pod-name"

If you want to bring the kuberents dashboard then you can run following deployment command to bring the dashboard pod online. 
>kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v1.10.1/src/deploy/recommended/kubernetes-dashboard.yaml

To access Dashboard from your local workstation you must create a secure channel to your Kubernetes cluster. Run the following command:
>kubectl proxy 

Now access Dashboard at:

>http://localhost:8001/api/v1/namespaces/kube-system/services/https:kubernetes-dashboard:/proxy/

To login to the dashboard you need to create some token or user id. The follwoing you can generate the token to login to the dashbaord.

 >kubectl create serviceaccount cluster-admin-dashboard-sa
 >kubectl create clusterrolebinding cluster-admin-dashboard-sa \
  --clusterrole=cluster-admin \
  --serviceaccount=default:cluster-admin-dashboard-sa
>kubectl get secret | grep cluster-admin-dashboard-sa
>kubectl describe secret “cluster-admin-dashboard-sa-token-6xm8l(output from previous command)”




