---
title: "Kubernetes"
authors: ["gio"]
---

#### Cheat Sheet

```
kubectl config view
kubectl current-context
kubectl config set-context <context-name> --namespace=<namespace> --cluster=<cluster> --user=<user>
kubectl config use-context <context-name>

kubectl get namespaces
kubectl create namespace vote

kubectl get nodes
kubectl get services
kubectl get pods

kubectl create -f <yaml-file(s)>
kubectl apply -f <yaml-file(s)> # preferred

kubectl logs -f --tail=100 <pod-name>
kubectl exec -it <pod-name> -- /bin/sh
```

#### Kubernetes Networking

References

- https://sookocheff.com/post/kubernetes/understanding-kubernetes-networking-model/
- https://www.youtube.com/watch?v=y2bhV81MfKQ

#### Fetching Context from Kops-provisioned Kubernetes

```
curl "https://s3.amazonaws.com/aws-cli/awscli-bundle.zip" -o "awscli-bundle.zip"
unzip awscli-bundle.zip
sudo ./awscli-bundle/install -i /usr/local/aws -b /usr/local/bin/aws

# Configure AWS Credentials
aws configure

# Install Kops
brew install kops

# Fetch Context
export KOPS_STATE_STORE=<state-url>
kops get cluster
kops export kubecfg <cluster-name>
```
