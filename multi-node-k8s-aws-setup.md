# AWS Multi-node Kubernetes Cluster Provisioning

## Launching an instance on AWS EC2 to act as master node

Points to note:
1. Kubernetes requires the master node to have atleast 2 CPUs by default. So create a new t2.medium EC2 instance for the master node.

2. Kubernetes master API server communicates with nodes via port 6443. So your security group for the instance should have a custom custom TCP rule which allows communication via port 6443.

## Launching two worker nodes on AWS EC2

Points to note:
1. Use the same security group as that of the master node.

## Configuring Master node.

# Copy the bootstrap scripts to the master node.
```
scp -i your_aws_key.pem k8smaster.sh username@master_node_ip:/home/ubuntu/
scp -i your_aws_key.pem k8scommon.sh username@master_node_ip:/home/ubuntu/
```

# Copy the bootstrap script to the worker nodes.
```
scp -i your_aws_key.pem k8scommon.sh username@worker_node1_ip:/home/ubuntu/
scp -i your_aws_key.pem k8scommon.sh username@worker_node2_ip:/home/ubuntu/
```

# SSH into the master node
```
ssh -i your_aws_key.pem username@master_node_ip
```

# SSH into worker node1 in a different terminal window
```
ssh -i your_aws_key.pem username@worker_node1_ip
```

# SSH into worker node2 in a different terminal window
```
ssh -i your_aws_key.pem username@worker_node2_ip
```

# Making the bootstrap scripts executable and then executing them on the master node
```
[ON MASTER NODE]
sudo su
chmod +x k8scommon.sh
chmod +x k8smaster.sh
sh k8scommon.sh
sh k8smaster.sh
```
Note: These will setup tools like Docker, Kubernetes etc on the master node and also will create a new k8s multi-node cluster. After executing the k8smaster.sh bootstrap script, you will get the join command for workers. Copy this join command and we will use it on the worker nodes.

# Making the bootstrap script executable and then executing them on worker node1
```
[ON WORKER NODE1]
sudo su
chmod +x k8scommon.sh
sh k8scommon.sh
```
Note: These will setup tools like Docker, Kubernetes etc on worker node1.

# Making the bootstrap script executable and then executing them on worker node2
```
[ON WORKER NODE2]
sudo su
chmod +x k8scommon.sh
sh k8scommon.sh
```
Note: These will setup tools like Docker, Kubernetes etc on worker node2.

# Getting node details before workers join. Execute this on master node.
```
[ON MASTER NODE]
kubectl get nodes
```
Note: You will see only node in the cluster, which is the master node itself.

# Joining worker node1 to the cluster. Execute the 'kubeadm join' command you copied from your master node after bootstrapping on worker node1.
```
[ON WORKER NODE1]
kubeadm join master_node_ip:6443 --token <cluster_token> --discovery-token-ca-cert-hash <cluster_discovery_token_certificate_hash>
```
Note: This will make worker node1 join the k8s cluster.

# Joining worker node2 to the cluster. Execute the 'kubeadm join' command you copied from your master node after bootstrapping on worker node2.
```
[ON WORKER NODE2]
kubeadm join master_node_ip:6443 --token <cluster_token> --discovery-token-ca-cert-hash <cluster_discovery_token_certificate_hash>
```
Note: This will make worker node2 join the k8s cluster.

# Getting node details after the workers join. Execute this on master node.
```
[ON MASTER NODE]
kubectl get nodes
```
Note: You will see three nodes in the cluster now: one master node and two worker nodes.

# Creating an nginx deployment on the cluster. Execute this on master node.
```
[ON MASTER NODE]
kubectl run nginx --image=nginx --port 80
```

# Creating the nginx service on the cluster. Execute this on master node.
```
[ON MASTER NODE]
kubectl expose deploy nginx --port 80 --target-port 80 --type NodePort
```

# Get the service details. Execute this on master node.
```
[ON MASTER NODE]
kubectl get services
```
Note: Ensure that the service mapped port is open on the security group attached to the cluster nodes.

# Testing the nginx service.
```
[ON A BROWSER]
http://worker_node1_ip:<mapped_port>
```
Note: Here you will see the nginx service default page being served.

# Getting the service details. Execute this on master node.
```
[ON MASTER NODE]
kubectl describe svc nginx
```

# Getting the pod details. Execute this on master node.
```
[ON MASTER NODE]
kubectl get pods
kubectl get pods --output=wide
```
Note: Here you will see the worker node on which the service currently runs.

# Scaling the deployment. Execute this on master node.
```
[ON MASTER NODE]
kubectl scale deployment nginx --replicas=3
```
Note: This will scale the nginx deployment to be available on total 3 pods from the current 1 pod.

# Getting the pod details after scaling. Execute this on master node.
```
[ON MASTER NODE]
kubectl get pods
kubectl get pods --output=wide
```
Note: Here you will see that the deployment is scaled to 3 pods, spread across the two worker nodes within the cluster. By default, the master node is not assigned any pods.

# Enabling the master node for pod scheduling. Execute this on master node.
```
[ON MASTER NODE]
kubectl taint nodes --all node-role.kubernetes.io/master-
```
Note: Here you will see that the deployment is scaled to 3 pods, spread across the two worker nodes within the cluster. By default, the master node is not assigned any pods.

# Deleting the deployment and service. Execute this on master node.
```
[ON MASTER NODE]
kubectl delete svc nginx
kubectl delete deployment nginx
```

## Terminate the EC2 instances.
