## Private AKS cluster with vnet integration

Control plane is exposed only on private ip and internal load balancer.
contrrol plane has public FQDN exposed to the internet. Public FQDN references a private IP could be disabled
Devops CD could only access through the Internal Load Balancer. 
Nodes access Control plane through internal load balancer.
No private endpoint is created although the cluster is private.

## Create the public cluster follow below steps.

1. Create resource group.
``` bash
az group create -n private-cluster-vnet-int -l eastus
```
3. vnet integration is still under preview. So we need register "EnableAPIServerVnetIntegrationPreview"
``` bash
az extension add --name aks-preview
az feature register --namespace "Microsoft.ContainerService" --name "EnableAPIServerVnetIntegrationPreview"
```
5. To verify, registration.
``` bash
az feature show --namespace "Microsoft.ContainerService" --name "EnableAPIServerVnetIntegrationPreview"
```
7. create private AKS cluster with vnet integration
``` bash
az aks create -n private-aks-cluster -g private-cluster-vnet-int --enable-apiserver-vnet-integration --enable-private-cluster
```
5. fetch public endpoint, You will get the public endpoint but it will be resolved to private ip of the cluster.
``` bash
az aks show -n private-aks-cluster -g private-cluster-vnet-int --query fqdn
# The behavior of this command has been altered by the following extension: aks-preview
# "private-aks-private-cluster-v-5c09ef-i82tbb2n.hcp.eastus.azmk8s.io"
```
6. nslookup the given fqdn
``` bash
nslookup public-aks-public-cluster-v-5c09ef-i82tbb2n.hcp.eastus.azmk8s.io
# Server:  reliance.reliance
# Address:  2405:201:d01d:3010::c0a8:1d01

# Non-authoritative answer:
# Name:    private-ak-private-cluster--5c09ef-03yzz3yi.hcp.eastus.azmk8s.io
# Address:  10.226.0.4
```
7. You can fetch the private fqdn, but it is only resolvable within the network.
``` bash
az aks show -n private-aks-cluster -g private-cluster-vnet-int --query privateFqdn
# The behavior of this command has been altered by the following extension: aks-preview
# "private-ak-private-cluster--5c09ef-5s5s5exh.e15fda61-605b-4afc-9543-a2e37a3509be.private.eastus.azmk8s.io"
This setup will add 2 components, 1. Internal Load Balancer 2. private DNS Zone.
```
## access the private cluster.
1. kubectl command invoke the command "kubectl get pods"
2. Jumpbox VM inside the AKS vnet or peered network
3. Use a Express route or VPN connection
4. User private endpoint connection

