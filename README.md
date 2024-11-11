# cis-ingress
Use F5 BIG-IP Container Ingress Services (CIS) as your Ingress Controller
then we deploy Hello-World App Using Ingress Resource. 
So in short, we are setting up CIS WITHOUT the intricacies of VXLan or BGP. 

It just communicates directly as NodePort, which is automatically added as pool members of BIG-IP. 
So, as long as it can communicate directly/has route to F5's Self-IP/Management IP 
So if you are using NodePort, you don't need container networking such as Flannel/BGP, as they are

Only to be used to access the pod directly (which involves routing/vxlan)

Additional Reference: 
https://loadbalancing.se/2021/03/28/installing-troubleshooting-and-running-bigip-ingress-controller/

Tips: 
Don't forget to Increase Restjavad memory on BIG-IP
  
- BIG-IP1: 10.201.10.156 (mgmt) / 172.16.100.156 on Version 16.1.0
- Kubernetes Master Node: 10.201.10.179 on Ubuntu 18.04 LTS and kubernetes version v1.25.4; containerd://1.5.9
- f5-appsvcs 	3.40.0

    
``` 
git clone https://github.com/ericausente/cis-ingress.git
cd cis_ingress
``` 
    
``` 
kubectl create secret generic bigip-login -n kube-system --from-literal=username=admin --from-literal=password="<BIG-IP Password>"
kubectl create serviceaccount bigip-ctlr -n kube-system
kubectl apply -f k8s_rbac.yaml
kubectl apply -f https://raw.githubusercontent.com/F5Networks/k8s-bigip-ctlr/master/docs/config_examples/rbac/clusterrole.yaml
### In Prep for VS CRD Later On
https://raw.githubusercontent.com/F5Networks/k8s-bigip-ctlr/refs/heads/master/docs/cis-20.x/config_examples/customResourceDefinitions/stable/customresourcedefinitions.yml
kubectl apply -f  cis-deploy.yaml
kubectl apply -f ingress-class.yaml
``` 

Although this example is copied from https://clouddocs.f5.com/training/community/containers/html/class1/module1/lab2.html
Take note that it was an old article and the Ingress Resource does not have ingressclassname configuration in it. 
We are now to specify the ingress controller that should handle the ingress resource by using the ingressClassName 
Source of truth in terms of parameters are now found in here: https://clouddocs.f5.com/containers/latest/userguide/ingress.html

Now deploy the hello-world app: 
``` 
kubectl create -f deployment-hello-world.yaml
kubectl create -f nodeport-service-hello-world.yaml
kubectl create -f ingress-hello-world.yaml
``` 
  
To Delete Hello-World
``` 
kubectl delete -f ingress-hello-world.yaml
kubectl delete -f nodeport-service-hello-world.yaml
kubectl delete -f deployment-hello-world.yaml
``` 


Upon applying the service: 
``` 
2023/01/31 02:43:44 [DEBUG] [CORE] Discovered members for service default/f5-hello-world-web is [{172.16.100.21 31301 8080 user-enabled} {172.16.100.22 31301 8080 user-enabled} {172.16.100.23 31301 8080 user-enabled} {10.201.10.153 31301 8080 user-enabled}]
2023/01/31 02:43:44 [DEBUG] [CORE] Updated 0 of 0 virtual server configs, deleted 0
2023/01/31 02:43:44 [DEBUG] [CORE] Finished syncing virtual servers f5-hello-world-web in namespace default (834.74µs), 9/9
2023/01/31 02:43:44 [DEBUG] [AS3] Preparing response message to response handler for arp and fdb config
2023/01/31 02:43:44 [DEBUG] [AS3] Sent response message to response handler for arp and fdb config
``` 

Upon creation of Ingress: 
``` 
2023/01/31 02:47:32 [DEBUG] [CORE] Configured rule: {ingress___ingress_default_f5-hello-world-web / 0 [0xc000307aa0] []}
2023/01/31 02:47:32 [DEBUG] [RESOURCE] Configured policy: {ingress_172-16-4-68_80 kubernetes  [forwarding]  true [http] [0xc000307b00] /Common/first-match}
2023/01/31 02:47:32 [DEBUG] Updating poolmembers for nodeport mode with service f5-hello-world-web/default
2023/01/31 02:47:32 [DEBUG] [CORE] Service backend matched {ServiceName:f5-hello-world-web ServicePort:8080 Namespace:default}: using node port 31466
2023/01/31 02:47:32 [DEBUG] [CORE] Updated 0 of 1 virtual server configs, deleted 0
2023/01/31 02:47:32 [DEBUG] [CORE] Finished syncing virtual servers f5-hello-world-web in namespace default (128.633µs), 9/9
2023/01/31 02:47:32 [DEBUG] [CORE] Configured rule: {ingress___ingress_default_f5-hello-world-web / 0 [0xc0004e4de0] []}
2023/01/31 02:47:32 [DEBUG] [RESOURCE] Configured policy: {ingress_172-16-4-68_80 kubernetes  [forwarding]  true [http] [0xc0004e4e40] /Common/first-match}
2023/01/31 02:47:32 [DEBUG] [CORE] Configured rule: {ingress___ingress_default_f5-hello-world-web / 0 [0xc0004e4f60] []}
2023/01/31 02:47:32 [DEBUG] [RESOURCE] Configured policy: {ingress_172-16-4-68_80 kubernetes  [forwarding]  true [http] [0xc0004e4fc0] /Common/first-match}
2023/01/31 02:47:32 [DEBUG] [CORE] Updated 0 of 1 virtual server configs, deleted 0
2023/01/31 02:47:32 [DEBUG] [CORE] Finished syncing virtual servers f5-hello-world-web in namespace default (259.902µs), 9/9
2023/01/31 02:47:38 [DEBUG] [AS3] Response from BIG-IP: code: 200 --- tenant:kubernetes --- message: success
2023/01/31 02:47:38 [DEBUG] [AS3] Preparing response message to response handler for arp and fdb config
2023/01/31 02:47:38 [DEBUG] [AS3] Sent response message to response handler for arp and fdb config
2023/01/31 02:47:38 [DEBUG] [AS3] Preparing response message to response handler for arp and fdb config
2023/01/31 02:47:38 [DEBUG] [AS3] Sent response message to response handler for arp and fdb config
``` 


