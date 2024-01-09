# DNS resolution
In hub-spoke architecture, both Public DNS and Private DNS Zone azure services are used for external and internal domain name resolution respectively. This document specifies different test case scenarios for the domain resolution. It also demonstrates via use of `az-cli` commands how to deploy the architecture in Azure cloud and test it.

## Test cases
1. An app hosted in hub network, should be accessible via internet using externally resolvable domain name.
2. An app hosted in hub network, should be accessible via machines in hub network using internally resolvable domain name.
3. An app hosted in hub network, should be accessible via machines in spoke network using internally resolvable domain name.
  
4. An app hosted in spoke network, should be accessible via hub network, using internally resolvable domain name.
5. An app hosted in spoke network, without having public ip address, having DNAT rule via firewall, can be connected from internet using externally resolvable domain name.
6. An app hosted in spoke network, should be accessible via another spoke network, using internally resolvable domain name.

### Local environment
Before running any of the test cases specified below, a developer would need to login to terminal/shell and set up correct subscription. Example commands are specified below. After running the test scenrios, destroy the resources created and use `az logout` command to log out of the shell.

```sh
az login --tenant cloudadminsystemsinmotion.onmicrosoft.com

az account set --subscription "sandbox.nexientcloud.com"
```
### Test case 1
An app hosted in hub network, should be accessible via internet using externally resolvable domain name.

**Set Up**
- VNET
- Subnet
- Public DNS Zone
- VM in subnet with public IP address

**az cli commands**
```sh
# Set subscription
az account set --subscription "sandbox.nexientcloud.com"

# Create hub resource group
az group create --location "eastus" --resource-group "hub-rg"

# Create hub vnet
az network vnet create --name "hub-vnet" \
                       --resource-group "hub-rg" \
                       --address-prefixes "10.0.0.0/16" \
                       --location "eastus"

# Create hub subnet
az network vnet subnet create --name "hub-subnt" \
                              --resource-group "hub-rg" \
                              --vnet-name "hub-vnet" \
                              --address-prefixes "10.0.1.0/24" 

# Create vm in hub vnet with public ip address
az vm create --name "jumpbox-vm" \
             --resource-group "hub-rg" \
             --accelerated-networking "false" \
             --admin-password "P@ssw0rd1234" \
             --admin-username "adminuser" \
             --authentication-type password \
             --image "Win2019Datacenter" \
             --location "eastus" \
             --priority "Spot" \
             --size "Standard_DS1_v2" \
             --specialized "false" \
             --subnet "hub-subnt"  \
             --subnet-address-prefix "10.0.1.0/24" \
             --vnet-address-prefix "10.0.0.0/16" \
             --vnet-name "hub-vnet" \
             --public-ip-address "default" \
             --public-ip-address-allocation "dynamic"

# Install IIS on hub vm
az vm extension set --publisher Microsoft.Compute \
                    --version 1.8 \
                    --name CustomScriptExtension \
                    --vm-name "jumpbox-vm" \
                    --resource-group "hub-rg" \
                    --settings '{"commandToExecute":"powershell.exe Install-WindowsFeature -Name Web-Server"}'             

# Create NSG rule to allo http traffic from internet to hub vm
az network nsg rule create --resource-group "hub-rg" \
                           --nsg-name "jumpbox-vmNSG" \
                           --name "allowhttp" \
                           --priority 100 \
                           --access "Allow" \
                           --destination-address-prefixes $(az vm nic show --vm-name jumpbox-vm --resource-group hub-rg --nic jumpbox-vmVMNic --query "ipConfigurations[0].privateIPAddress" -o tsv) \
                           --destination-port-ranges 80 443 \
                           --direction "Inbound" \
                           --protocol "Tcp" \
                           --source-address-prefixes "*" \
                           --source-port-ranges "*"

# Create A record in public DNS zone for the hub vm endpoint resolution
az network dns record-set a add-record --ipv4-address $(az network public-ip show --ids /subscriptions/e71eb3cd-83f2-46eb-8f47-3b779e27672f/resourceGroups/hub-rg/providers/Microsoft.Network/publicIPAddresses/default --query ipAddress -o tsv) \
                                       --record-set-name "testvm" \
                                       --resource-group "public-dns-eastus-rg-000" \
                                       --zone-name "sandbox.launch.dresdencraft.com" \
                                       --ttl 1
```
### Test case 2
An app hosted in hub network, should be accessible via machines in hub network using internally resolvable domain name.

**Set Up**
- VNET
- Subnet
- Private DNS Zone
- VM in subnet with private IP address
- Private Link in private DNS zone to register VNET
  
**az cli commands**
Few commands used for Test case 1 + below set of additional commands

```sh
# Create private dns zone in hub resource group
az network private-dns zone create --resource-group "hub-rg" \
                                   --name "sandbox.launch.dresdencraft.com"
                                  

# Create virtual network link in private dns zone for resolution of ip addresses in hub vnet
az network private-dns link vnet create --resource-group "hub-rg" \
                                        --virtual-network "hub-vnet" \
                                        --zone-name "sandbox.launch.dresdencraft.com" \
                                        --name  "hub-vnet-link" \
                                        --registration-enabled true
``` 

### Test case 3
An app hosted in hub network, should be accessible via machines in spoke network using internally resolvable domain name.

**Set Up**
- VNET(spoke+hub)
- Subnets in each VNET
- APP hosted on VM in hub
- VM in spoke with private IP address
- Bastion to access spoke
- Virtual network link in private dns zone of hub to register spoke-vnet

**az cli commands**
```sh
# Create spoke resource group
az group create --location "eastus" --resource-group "spoke-rg"

# Create spoke vnet
az network vnet create --name "spoke-vnet" \
                       --resource-group "spoke-rg" \
                       --address-prefixes "192.168.0.0/16" \
                       --location "eastus"

# Create spoke subnet
az network vnet subnet create --name "spoke-subnt" \
                              --resource-group "spoke-rg" \
                              --vnet-name "spoke-vnet" \
                              --address-prefixes "192.168.0.0/24"

# Create vm in spoke vnet with private ip address
az vm create --name "spoke-vm" \
             --resource-group "spoke-rg" \
             --accelerated-networking "false" \
             --admin-password "P@ssw0rd1234" \
             --admin-username "adminuser" \
             --authentication-type password \
             --image "Win2019Datacenter" \
             --location "eastus" \
             --priority "Spot" \
             --size "Standard_DS1_v2" \
             --specialized "false" \
             --subnet "spoke-subnt"  \
             --subnet-address-prefix "192.168.0.0/24" \
             --vnet-address-prefix "192.168.0.0/16" \
             --vnet-name "spoke-vnet" \
             --public-ip-address ""                           

# Establish vnet peering between spoke and hub vnet
az network vnet peering create --name "spoke-vnet-to-hub-vnet" \
                               --remote-vnet "/subscriptions/e71eb3cd-83f2-46eb-8f47-3b779e27672f/resourceGroups/hub-rg/providers/Microsoft.Network/virtualNetworks/hub-vnet" \
                               --resource-group "spoke-rg" \
                               --vnet-name "spoke-vnet" \
                               --allow-vnet-access true \
                               --allow-forwarded-traffic true


# Establish vnet peering between hub and spoke vnet
az network vnet peering create --name "hub-vnet-to-spoke-vnet" \
                               --remote-vnet "/subscriptions/e71eb3cd-83f2-46eb-8f47-3b779e27672f/resourceGroups/spoke-rg/providers/Microsoft.Network/virtualNetworks/spoke-vnet" \
                               --resource-group "hub-rg" \
                               --vnet-name "hub-vnet" \
                               --allow-vnet-access true \
                               --allow-forwarded-traffic true             

# Create AzureBastionSubnet subnet in spoke vnet
az network vnet subnet create --name "AzureBastionSubnet" \
                              --resource-group "spoke-rg" \
                              --vnet-name "spoke-vnet" \
                              --address-prefixes "192.168.1.0/24"                               

# Create public-ip for AzureBastionSubnet subnet
az network public-ip create --name "spoke-bastion-pip" \
                            --resource-group "spoke-rg" \
                            --allocation-method "Static" \
                            --location "eastus" \
                            --sku "Standard" \
                            --tier "Regional" \
                            --version "IPv4"         

# Create AzureBastion for spoke vnet
az network bastion create --name "spoke-bastion" \
                          --public-ip-address "spoke-bastion-pip" \
                          --resource-group "spoke-rg" \
                          --vnet-name "spoke-vnet" \
                          --location "eastus" \
                          --sku "Basic"                                                  

# Register spoke network with private dns zone of hub, so from spoke machines, hub apps are resolvable 
az network private-dns link vnet create --name "spoke-vnet-link" \
                                        --registration-enabled no \
                                        --resource-group "hub-rg" \
                                        --virtual-network "/subscriptions/e71eb3cd-83f2-46eb-8f47-3b779e27672f/resourceGroups/spoke-rg/providers/Microsoft.Network/virtualNetworks/spoke-vnet" \
                                        --zone-name "sandbox.launch.dresdencraft.com"        
```

### Test case 4
An app hosted in spoke network, should be accessible via hub network, using internally resolvable domain name.

**Set Up**
- VNET(spoke+hub)
- Subnets in each VNET
- APP hosted on VM in hub
- VM in spoke with private IP address
- Bastion to access spoke
- Private DNS zone in spoke
- Virtual network link for spoke vnet to automatically register spoke vms in the private dns zone of spoke
- Virtual network link for hub vnet in the private dns zone of spoke so hub machines can resolve ip addresses of spoke machines.
  
**az cli commands**
```sh
# Install IIS on spoke VM
az vm extension set --publisher Microsoft.Compute \
                    --version 1.8 \
                    --name CustomScriptExtension \
                    --vm-name "spoke-vm" \
                    --resource-group "spoke-rg" \
                    --settings '{"commandToExecute":"powershell.exe Install-WindowsFeature -Name Web-Server"}'   

# Create DNS zone in spoke, the name should be different than that in hub
az network private-dns zone create --resource-group "spoke-rg" \
                                   --name "spoke.sandbox.launch.dresdencraft.com"

# Create private link in spoke private dns zone for spoke vnet so that spoke machines are automatically resolvable
az network private-dns link vnet create --resource-group "spoke-rg" \
                                        --virtual-network "spoke-vnet" \
                                        --zone-name "spoke.sandbox.launch.dresdencraft.com" \
                                        --name  "spoke-vnet-link" \
                                        --registration-enabled true        

# Register hub network with private dns zone of spoke, so from hub machines, spoke apps are resolvable
az network private-dns link vnet create --name "hub-vnet-link" \
                                        --registration-enabled no \
                                        --resource-group "spoke-rg" \
                                        --virtual-network "/subscriptions/e71eb3cd-83f2-46eb-8f47-3b779e27672f/resourceGroups/hub-rg/providers/Microsoft.Network/virtualNetworks/hub-vnet" \
                                        --zone-name "spoke.sandbox.launch.dresdencraft.com"                                                                      
```

### Test case 5
An app hosted in spoke network, without having public ip address, having DNAT rule via firewall, can be connected from internet using externally resolvable domain name.  

**Set Up**
- VNET(spoke+hub)
- Subnets in each VNET
- VM in spoke with private IP address
- APP hosted on VM in spoke
- Firewall with DNAT rule to connect to spoke vm's private ip
- A record in public DNS zone for firewall public IP

**az cli commands**
```sh
# Create AzureFirewallSubnet subnet
az network vnet subnet create --name "AzureFirewallSubnet" \
                              --resource-group "hub-rg" \
                              --vnet-name "hub-vnet" \
                              --address-prefixes "10.0.0.0/24"

# Create Azure Firewall Policy
az network firewall policy create --name "hub-firewall-policy" \
                                  --resource-group "hub-rg" \
                                  --location "eastus" \
                                  --sku "Standard" 

# Create public ip for the azure firewall
az network public-ip create --name "firewall-pip" \
                            --resource-group "hub-rg" \
                            --allocation-method "Static" \
                            --location "eastus" \
                            --sku "Standard" \
                            --tier "Regional" \
                            --version "IPv4"

# Create the azure firewall
az network firewall create --name "hub-firewall" \
                           --resource-group "hub-rg" \
                           --firewall-policy "hub-firewall-policy" \
                           --location "eastus" \
                           --public-ip "firewall-pip" \
                           --sku "AZFW_VNet" \
                           --tier "Standard" \
                           --vnet-name "hub-vnet" \
                           --private-ranges IANAPrivateRanges  

# Create the ip configuration for the azure firewall
az network firewall ip-config create --firewall-name "hub-firewall" \
                                     --name "hub-firewall-ip-config" \
                                     --public-ip-address "firewall-pip" \
                                     --resource-group "hub-rg" \
                                     --vnet-name "hub-vnet"

#  Create DNAT rule collection group
az network firewall policy rule-collection-group create --name "hub-firewall-dnat-rcg" \
                                                        --policy-name "hub-firewall-policy" \
                                                        --priority 100 \
                                                        --resource-group "hub-rg"

#  Create DNAT rule to forward the traffic directed to azure firewall public ip address to spoke vm
az network firewall policy rule-collection-group collection add-nat-collection --collection-priority 100 \
                                                                               --name "dnat-collection" \
                                                                               --policy-name "hub-firewall-policy" \
                                                                               --rcg-name "hub-firewall-dnat-rcg" \
                                                                               --resource-group "hub-rg" \
                                                                               --action "DNAT" \
                                                                               --destination-addresses $(az network public-ip show --ids $(az network firewall ip-config show --firewall-name "hub-firewall" --name "hub-firewall-ip-config" --resource-group "hub-rg" --subscription "sandbox.nexientcloud.com" --query "publicIpAddress.id" -o tsv) --query "ipAddress" -o tsv \
                                                                               --destination-ports 80 \
                                                                               --rule-name "allowHttpFromNet" \
                                                                               --source-addresses "*" \
                                                                               --translated-address $(az vm nic show --vm-name spoke-vm --resource-group spoke-rg --nic spoke-vmVMNic --query "ipConfigurations[0].privateIPAddress" -o tsv) \
                                                                               --translated-port 80 \
                                                                               --ip-protocols TCP

# Create A record in public dns zone for public ip address of firewall
az network dns record-set a add-record --ipv4-address $(az network public-ip show --ids $(az network firewall ip-config show --firewall-name "hub-firewall" --name "hub-firewall-ip-config" --resource-group "hub-rg" --subscription "sandbox.nexientcloud.com" --query "publicIpAddress.id" -o tsv ) --query "ipAddress" -o tsv) \
                                       --record-set-name "spokevm" \
                                       --resource-group "public-dns-eastus-rg-000" \
                                       --zone-name "sandbox.launch.dresdencraft.com" \
                                       --ttl 1          
```

### Test case 6
An app hosted in spoke network, should be accessible via another spoke network, using internally resolvable domain name.  

**Set Up**
- Network(2 spokes + hub)
- VM in spoke1 with private IP address
- VM in spoke2 with private IP address
- APPS hosted on VM in spoke1 and spoke2
- Azure bastion to connect to spoke vms
- Route tables on both spoke vnets to forward traffic to each other via firewall

**az cli commands**
```sh
# Create spoke 2 resource group
az group create --location "eastus" --resource-group "spoke-2-rg"

# Create spoke 2 vnet
az network vnet create --name "spoke-2-vnet" \
                       --resource-group "spoke-2-rg" \
                       --address-prefixes "172.16.0.0/16" \
                       --location "eastus"

# Create spoke 2 subnet
az network vnet subnet create --name "spoke-2-subnt" \
                              --resource-group "spoke-2-rg" \
                              --vnet-name "spoke-2-vnet" \
                              --address-prefixes "172.16.0.0/24"

# Create spoke 2 vm
az vm create --name "spoke-2-vm" \
             --resource-group "spoke-2-rg" \
             --accelerated-networking "false" \
             --admin-password "P@ssw0rd1234" \
             --admin-username "adminuser" \
             --authentication-type password \
             --image "Win2019Datacenter" \
             --location "eastus" \
             --priority "Spot" \
             --size "Standard_DS1_v2" \
             --specialized "false" \
             --subnet "spoke-2-subnt"  \
             --subnet-address-prefix "172.16.0.0/24" \
             --vnet-address-prefix "172.16.0.0/16" \
             --vnet-name "spoke-2-vnet" \
             --public-ip-address ""    

# Install IIS on spoke 2 vm
az vm extension set --publisher Microsoft.Compute \
                    --version 1.8 \
                    --name CustomScriptExtension \
                    --vm-name "spoke-2-vm" \
                    --resource-group "spoke-2-rg" \
                    --settings '{"commandToExecute":"powershell.exe Install-WindowsFeature -Name Web-Server"}'              

# Create public ip for Azure bastion in spoke 2
az network public-ip create --name "spoke-2-bastion-pip" \
                            --resource-group "spoke-2-rg" \
                            --allocation-method "Static" \
                            --location "eastus" \
                            --sku "Standard" \
                            --tier "Regional" \
                            --version "IPv4"         

# Create AzureBastionSubnet for spoke2 vnet
az network vnet subnet create --name "AzureBastionSubnet" \
                              --resource-group "spoke-2-rg" \
                              --vnet-name "spoke-2-vnet" \
                              --address-prefixes "172.16.1.0/24"   

# Create public ip for AzureBastion in spoke2 vnet
az network bastion create --name "spoke-2-bastion" \
                          --public-ip-address "spoke-2-bastion-pip" \
                          --resource-group "spoke-2-rg" \
                          --vnet-name "spoke-2-vnet" \
                          --location "eastus" \
                          --sku "Basic"                      

# Create route table for spoke2 subnet
az network route-table create --name "spoke-2-sbnt-rt" \
                              --resource-group "spoke-2-rg" \
                              --location "eastus" 

# Associate route table with spoke2 subnet
az network vnet subnet update --resource-group "spoke-2-rg" \
                              --vnet-name "spoke-2-vnet" \
                              --name "spoke-2-subnt" \
                              --route-table "spoke-2-sbnt-rt"

# Create route to forward traffic from spoke subnet 2 to firewall
az network route-table route create --name "spoke2tospoke1route" \
                                    --resource-group "spoke-2-rg" \
                                    --route-table-name "spoke-2-sbnt-rt" \
                                    --address-prefix "192.168.0.0/24" \
                                    --next-hop-ip-address "10.0.0.4" \
                                    --next-hop-type VirtualAppliance

# Create route table for spoke subnet
az network route-table create --name "spoke-sbnt-rt" \
                              --resource-group "spoke-rg" \
                              --location "eastus" 

# Associate route table with spoke1 subnet
az network vnet subnet update --resource-group "spoke-rg" \
                              --vnet-name "spoke-vnet" \
                              --name "spoke-subnt" \
                              --route-table "spoke-sbnt-rt"

# Create route to forward traffic from spoke subnet to firewall
az network route-table route create --name "spoke1tospoke2route" \
                                    --resource-group "spoke-rg" \
                                    --route-table-name "spoke-sbnt-rt" \
                                    --address-prefix "172.16.0.0/24" \
                                    --next-hop-ip-address "10.0.0.4" \
                                    --next-hop-type VirtualAppliance                                    

# Create DNS zone in spoke 2, the name should be unique
az network private-dns zone create --resource-group "spoke-2-rg" \
                                   --name "spoke2.sandbox.launch.dresdencraft.com"

# Create private link in spoke private dns zone for spoke 2 vnet so that spoke 2 machines are automatically resolvable
az network private-dns link vnet create --resource-group "spoke-2-rg" \
                                        --virtual-network "spoke-2-vnet" \
                                        --zone-name "spoke2.sandbox.launch.dresdencraft.com" \
                                        --name  "spoke-2-vnet-link" \
                                        --registration-enabled true

# Create virtual network peering connection between spoke2 and hub vnet
az network vnet peering create --name "spoke-vnet-2-to-hub-vnet" \
                               --remote-vnet "/subscriptions/e71eb3cd-83f2-46eb-8f47-3b779e27672f/resourceGroups/hub-rg/providers/Microsoft.Network/virtualNetworks/hub-vnet" \
                               --resource-group "spoke-2-rg" \
                               --vnet-name "spoke-2-vnet" \
                               --allow-vnet-access true \
                               --allow-forwarded-traffic true 

# Create virtual network peering connection between hub and spoke2 vnet
az network vnet peering create --name "hub-vnet-to-spoke-2-vnet" \
                               --remote-vnet "/subscriptions/e71eb3cd-83f2-46eb-8f47-3b779e27672f/resourceGroups/spoke-2-rg/providers/Microsoft.Network/virtualNetworks/spoke-2-vnet" \
                               --resource-group "hub-rg" \
                               --vnet-name "hub-vnet" \
                               --allow-vnet-access true \
                               --allow-forwarded-traffic true 

# Create rule collection group for network policies
az network firewall policy rule-collection-group create --name "hub-firewall-network-rcg" \
                                                        --policy-name "hub-firewall-policy" \
                                                        --priority 200 \
                                                        --resource-group "hub-rg"

# Create network rule to forward spoke2 traffic to spoke1 via firewall
az network firewall policy rule-collection-group collection add-filter-collection --collection-priority 200 \
                                                                                  --name "network-filter-collection" \
                                                                                  --policy-name "hub-firewall-policy" \
                                                                                  --rcg-name "hub-firewall-network-rcg" \
                                                                                  --resource-group "hub-rg" \
                                                                                  --rule-type "NetworkRule" \
                                                                                  --destination-addresses "192.168.0.0/24" \
                                                                                  --destination-ports 80 \
                                                                                  --ip-protocols TCP \
                                                                                  --source-addresses "172.16.0.0/24" \
                                                                                  --description "spoke 2 to spoke 1 routing" \
                                                                                  --rule-name "forwardspoke2tospoke1"

# Create rule collection group for network policies
az network firewall policy rule-collection-group create --name "hub-firewall-network-rcg-2" \
                                                        --policy-name "hub-firewall-policy" \
                                                        --priority 300 \
                                                        --resource-group "hub-rg"


# Create network rule to forward spoke1 traffic to spoke2 via firewall
az network firewall policy rule-collection-group collection add-filter-collection --collection-priority 300 \
                                                                                  --name "network-filter-collection" \
                                                                                  --policy-name "hub-firewall-policy" \
                                                                                  --rcg-name "hub-firewall-network-rcg-2" \
                                                                                  --resource-group "hub-rg" \
                                                                                  --rule-type "NetworkRule" \
                                                                                  --destination-addresses "172.16.0.0/24" \
                                                                                  --destination-ports 80 \
                                                                                  --ip-protocols TCP \
                                                                                  --source-addresses "192.168.0.0/24" \
                                                                                  --description "spoke 1 to spoke 2 routing" \
                                                                                  --rule-name "forwardspoke1tospoke2"

# Register spoke2 network with private dns zone of hub, so from spoke 2 vnet machines, hub apps are resolvable
az network private-dns link vnet create --name "spoke2-vnet-link" \
                                        --registration-enabled no \
                                        --resource-group "hub-rg" \
                                        --virtual-network "/subscriptions/e71eb3cd-83f2-46eb-8f47-3b779e27672f/resourceGroups/spoke-2-rg/providers/Microsoft.Network/virtualNetworks/spoke-2-vnet" \
                                        --zone-name "sandbox.launch.dresdencraft.com"    

# Register spoke2 network with private dns zone of spoke 1, so from spoke 2 vnet machines, spoke apps are resolvable
az network private-dns link vnet create --name "spoke2-vnet-link" \
                                        --registration-enabled no \
                                        --resource-group "spoke-rg" \
                                        --virtual-network "/subscriptions/e71eb3cd-83f2-46eb-8f47-3b779e27672f/resourceGroups/spoke-2-rg/providers/Microsoft.Network/virtualNetworks/spoke-2-vnet" \
                                        --zone-name "spoke.sandbox.launch.dresdencraft.com"                                           

# Register spoke 1 network with private dns zone of spoke 2, so from spoke 1 vnet machines, spoke 2 apps are resolvable
az network private-dns link vnet create --name "spoke-vnet-link" \
                                        --registration-enabled no \
                                        --resource-group "spoke-2-rg" \
                                        --virtual-network "/subscriptions/e71eb3cd-83f2-46eb-8f47-3b779e27672f/resourceGroups/spoke-rg/providers/Microsoft.Network/virtualNetworks/spoke-vnet" \
                                        --zone-name "spoke2.sandbox.launch.dresdencraft.com"  

# Register hub network with private dns zone of spoke 2, so from spoke 2 vnet machines, hub apps are resolvable
az network private-dns link vnet create --name "hub-vnet-link" \
                                        --registration-enabled no \
                                        --resource-group "spoke-2-rg" \
                                        --virtual-network "/subscriptions/e71eb3cd-83f2-46eb-8f47-3b779e27672f/resourceGroups/hub-rg/providers/Microsoft.Network/virtualNetworks/hub-vnet" \
                                        --zone-name "spoke2.sandbox.launch.dresdencraft.com"                                                                                      
```                                     