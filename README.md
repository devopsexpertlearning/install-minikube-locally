# Kubernetes & Minikube Setup Guide

This guide covers the installation of Docker, Conntrack, `kubectl`, `minikube`, and necessary configurations for deploying applications locally or on AWS EC2.

## Step 1: **Docker Installation**

Run the following commands to install Docker and set up user permissions:

```bash
sudo yum update -y && sudo amazon-linux-extras install docker -y
sudo systemctl start docker
sudo systemctl enable docker

```

Add the current user to the Docker group

```
sudo usermod -aG docker $USER
newgrp docker # this allows running Docker commands without "ec2-user"

```
## Step 2: Install Conntrack

Conntrack is required for Kubernetes networking.

```
sudo yum install conntrack -y
```

## Step 3: How to Install kubectl on Linux

Use the following commands to install kubectl, which is the Kubernetes command-line tool:

Get the latest stable version of kubectl
```
curl -LO https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl
```
Make kubectl executable
```
chmod +x ./kubectl
```
Move it to the system PATH
```
sudo mv ./kubectl /usr/bin/kubectl
```
Verify installation

```
kubectl version --client
```

## Step 4: Minikube Installation

Follow these commands to install Minikube, a local Kubernetes cluster:

Download the latest Minikube release

```
wget https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
```

Make it executable
```
sudo chmod +x minikube-linux-amd64
```

Move it to the system PATH
```
sudo mv minikube-linux-amd64 /usr/bin/minikube
```
Verify the installation
```
minikube version
```

## Step 5: Start Your Cluster

To start Minikube and verify that everything is working:

Start Minikube
```
minikube start
```

Verify the cluster is running
```
kubectl get po -A
```

## Step 6: If Installing on Ubuntu, Install cri-dockerd

If you're using Ubuntu, you may need to install cri-dockerd for Docker compatibility with Kubernetes.

Install Go

```
wget https://storage.googleapis.com/golang/getgo/installer_linux
chmod +x ./installer_linux
./installer_linux
source ~/.bash_profile

# Clone cri-dockerd repository and build the binary
cd cri-dockerd
mkdir bin
go build -o bin/cri-dockerd
install -o root -g root -m 0755 bin/cri-dockerd /usr/bin/cri-dockerd

# Install systemd service files
cp -a packaging/systemd/* /etc/systemd/system
sed -i -e 's,/usr/bin/cri-dockerd,/usr/bin/cri-dockerd,' /etc/systemd/system/cri-docker.service
systemctl daemon-reload
systemctl enable cri-docker.service
systemctl enable --now cri-docker.socket
```

## Step 7: Install crictl

To install crictl, which is used for interacting with container runtimes:

```
# Download crictl
VERSION="v1.24.1"
wget https://github.com/kubernetes-sigs/cri-tools/releases/download/$VERSION/crictl-$VERSION-linux-amd64.tar.gz

# Extract and install it
sudo tar zxvf crictl-$VERSION-linux-amd64.tar.gz -C /usr/bin
rm -f crictl-$VERSION-linux-amd64.tar.gz
```

## Step 8: Deploy Applications

To deploy a simple application using Minikube:
```
# Create a deployment for a sample application
kubectl create deployment hello-minikube --image=k8s.gcr.io/echoserver:1.4

# Expose the deployment on a NodePort
kubectl expose deployment hello-minikube --type=NodePort --port=8080

# Get the service details
kubectl get services hello-minikube

```

## Step 9: Use kubectl to Forward the Port

You can forward the port to access the service locally:
```
# Forward the port to your local machine
kubectl port-forward service/hello-minikube [ForwardPort]:[TargetPort]

# Example: Forwarding the service on port 8080 to port 7081
kubectl port-forward service/hello-minikube 7081:8080

```
Step 10: Create Local SSH Tunnel

To access your Kubernetes cluster externally, create a local SSH tunnel:

```
ssh -i /c/Users/jaibhole/Desktop/aws-key -L 8081:localhost:7080 ec2-user@52.66.221.70

# Access the service in the browser
URL - http://127.0.0.1:8081

```
## Conclusion

You should now have a fully functional Minikube Kubernetes cluster set up with Docker, kubectl, and cri-dockerd. You can deploy applications, forward ports, and even create local SSH tunnels to access services.


Let me know if you want any changes to the formatting or content!
