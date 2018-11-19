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
kubectl delete -f <yaml-file(s)>

kubectl logs -f --tail=100 <pod-name>
kubectl exec -it <pod-name> -- /bin/sh
```

#### Helm

Setup helm

```
brew install kubernetes-helm
kubectl config use-context <CONTEXT-NAME>
helm init
```

Search charts

```
# show all charts
helm search

# show all charts in stable repository
helm search stable/
```

Install charts

```
# When user supplies custom values, these values will override the values in the chart's values.yaml file.
helm install --values=myvals.yaml <CHART-NAME>

# If you only want to test generated configuration, you can add --dry-run and the chart won't be installed
helm install --debug --dry-run --values=myvals.yaml <PATH-OR-CHART-NAME>
```

References
- https://docs.helm.sh/chart_template_guide
- https://github.com/helm/helm/blob/master/docs/charts_tips_and_tricks.md

#### Minikube

If you encounter this error when deploying in minikube cluster:

```
rpc error: code = Unknown desc = Error response from daemon
```

Try restarting minikube.

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
