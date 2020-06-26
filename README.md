## Resources

* https://docs.microsoft.com/en-us/azure/aks/
* https://docs.microsoft.com/en-us/azure/aks/tutorial-kubernetes-prepare-app
* https://azure.microsoft.com/en-us/resources/get-started-with-kubernetes-on-azure
* https://www.linkedin.com/learning/topics/kubernetes

## Azure Cloud Shell

Start Azure Cloud Shell

## Clone this repo

```
git clone https://github.com/shurikk/quick-aks.git
cd quick-aks
``` 

## Create resource group

```
az group create -l eastus -n rg-handsonaks
```

## Create AKS cluster

```
az aks create -n handsonaks -g rg-handsonaks -a http_application_routing --load-balancer-sku Standard -c 2 -l eastus --generate-ssh-keys
```

NOTE: this step might take a few minutes

## Get cluster credentials

```
az aks get-credentials -n handsonaks -g rg-handsonaks
```

## Explore AKS cluster

List of cluster nodes

```
kubectl get nodes -o wide
```

List of namespaces

```
kubectl get namespaces -o wide
```

List of pods running in kube-system namespace

```
kubectl get pods -n kube-system
```

List of different things in all namespaces

```
kubectl  get svc,deploy,ds,rs,pods --all-namespaces -o wide
```

## Basic echo service

Deploy

```
0-deployment.yml
kubectl apply -f 0-deployment.yml
```

Inspect

```
kubectl get deploy -o wide
kubectl get deploy -o yaml
kubectl get pods -o wide
```

Create service

```
cat 1-service.yml
kubectl apply -f 1-service.yml
```

Inspect

```
kubectl get svc -o wide
```

Create ingress, expose service publically

```
cat 2-ingress.yml
kubectl apply -f 2-ingress.yml
watch kubectl get ingress -o wide
```

Use the IP address and access the service using your web browser or `curl`, e.g.

```
export IP=$(kubectl get ingress/echo -o jsonpath="{.status.loadBalancer.ingress[*].ip}")
export URL=http://$IP/echo
echo $IP
echo $URL
curl $URL/i/am/echoed
```

NOTE: also try opening URL using your browser

Scale up and down

```
kubectl scale --replicas 5 deploy/echo
for i in $(seq 10); do curl -s $URL/hi/there/$i | grep -a2 "Request served"; done
kubectl scale --replicas 1 deploy/echo
```

Delete echo application

```
kubectl delete -f 0-deployment.yml,1-service.yml,2-ingress.yml
kubectl get deploy,svc,ing
```

## Demo vote application

```
kubectl apply -f 4-azure-vote-all-in-one.yml
watch kubectl get pods
kubectl get svc -o wide
```

Get azure-vote-front service external IP address and open in your web browser, amd watch pods logs

```
kubectl logs -l app=azure-vote-front -f
```

Enter the pod

```
kubectl exec -it $(kubectl get pod -o name -l app=azure-vote-front) -- bash
ps auxw
```

Delete vote application

```
kubectl delete -f 4-azure-vote-all-in-one.yml
```

## Cleanup

Delete resource group, everything will be gone

```
az group delete -n rg-handsonaks --no-wait --yes
```
