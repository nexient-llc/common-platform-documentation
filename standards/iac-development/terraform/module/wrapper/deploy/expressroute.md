## High level components required for architecture
    1. Express Route Circuit
    2. VPN Gateteway of type "-GatewayType ExpressRoute"
    3. Virtual WAN
    4. Virtual Hub
    5. Virtual Networks(in one or more subscriptions)
   
## Terraform modules:
1. Create expressroute circuit:
   1. [azurerm_express_route_circuit](https://registry.terraform.io/providers/hashicorp/azurerm/latest/docs/resources/express_route_circuit)
      1. [Peering location](https://learn.microsoft.com/en-us/azure/expressroute/expressroute-locations-providers#expressroute-locations)
      2. [service_provider_name](https://learn.microsoft.com/en-us/azure/expressroute/expressroute-locations-providers)
   

2. Create ExpressRoute virtual network gateways - To connect your Azure virtual network and your on-premises network using ExpressRoute, you must first create a virtual network gateway. A virtual network gateway serves two purposes: exchange IP routes between the networks and route network traffic. 
   1. [azurerm_virtual_wan](https://registry.terraform.io/providers/hashicorp/azurerm/latest/docs/resources/virtual_wan)
   2. [azurerm_virtual_hub](https://registry.terraform.io/providers/hashicorp/azurerm/latest/docs/resources/virtual_hub)
   3. [azurerm_express_route_gateway](https://registry.terraform.io/providers/hashicorp/azurerm/latest/docs/resources/express_route_gateway)

3. Link a virtual network to an ExpressRoute Circuit - 
   1. Using Terraform - [azurerm_express_route_connection](https://registry.terraform.io/providers/hashicorp/azurerm/latest/docs/resources/express_route_connection) 
   2. Using Azure Cli - https://learn.microsoft.com/en-us/azure/expressroute/expressroute-howto-linkvnet-cli
    
    Note: Azure Vnets are connected to Azure Virtual Hub via Azure Virtual WAN. Azure Express Route Gateway is connected to same Azure Virtual Hub. ExpressRoute connection is created between Azure Express Route Gateway and Azure Express Route circuit peering. This provides a link to connect the Azure Virtual Network/s to an ExpressRoute Circuit(in Theory)


4. Create and modify peering configurations - These instructions only apply to circuits created with service providers offering Layer 2 connectivity services. If you're using a service provider that offers managed Layer 3 services (typically an IPVPN, like MPLS), your connectivity provider will configure and manage routing for you.
   1. [azurerm_express_route_circuit_peering](https://registry.terraform.io/providers/hashicorp/azurerm/latest/docs/resources/express_route_circuit_peering)
   Notes: `Azure private peering` is required for establishing connectivity from on-premise vnets to azure vnets using expressroute

5. Managing express route circuit for usage in different subscriptions - You can share an ExpressRoute circuit across multiple subscriptions. Each of the departments within the organization uses their own subscription for deploying their services,but they can share a single ExpressRoute circuit to connect back to your on-premises network. 
   1. Circuit Owner - The 'Circuit Owner' is an authorized Power User of the ExpressRoute circuit resource. The Circuit Owner can create authorizations that can be redeemed by 'Circuit Users'. A "User defined role" would need to be created using permissions specified [here](https://learn.microsoft.com/en-us/azure/expressroute/expressroute-howto-linkvnet-cli#administration---circuit-owners-and-circuit-users)
      1. [azurerm_role_definition](https://registry.terraform.io/providers/hashicorp/azurerm/latest/docs/resources/role_definition)
   2. Circuit Users - Circuit Users are owners of virtual network gateways that aren't within the same subscription as the ExpressRoute circuit. Circuit Users can redeem authorizations (one authorization per virtual network).
   3. The circuit owner creates an authorization, which creates an authorization key to be used by a circuit user to connect their virtual network gateways to the ExpressRoute circuit. An authorization is valid for only one connection.
      1. [azurerm_express_route_circuit_authorization](https://registry.terraform.io/providers/hashicorp/azurerm/latest/docs/resources/express_route_circuit_authorization)

  
## Useful Microsoft documentation
1. [Link a virtual network to an expressroute circuit](https://learn.microsoft.com/en-us/azure/expressroute/expressroute-howto-linkvnet-cli)
2. [Create expressroute peering connections](https://learn.microsoft.com/en-us/azure/expressroute/howto-routing-cli)
3. [Express route introduction](https://learn.microsoft.com/en-us/azure/expressroute/expressroute-introduction)
4. [Express route locations by providers](https://learn.microsoft.com/en-us/azure/expressroute/expressroute-locations-providers)
5. [Create express route virtual network gateway](https://learn.microsoft.com/en-us/azure/expressroute/expressroute-howto-add-gateway-portal-resource-manager)