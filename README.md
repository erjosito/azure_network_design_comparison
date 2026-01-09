# Azure network architecture comparison

This project is a simple spreadsheet that can be used to compare the cost of different Azure architectures in Azure.

The spreadsheet includes prices updated in January 2026 for the West Europe region. Make sure that you update the prices to reflect current ones.

## Adjustable parameters

There are four parameters that you can change in the simulation:

- Total number of spoke VNets over all regions
- Number of regions
- Traffic volume between every pair of spoke VNets
- Traffic volume from every spoke VNet to on-premises
- Traffic volume from on-premises to every spoke VNet: this will impact the sizing for ExpressRoute gateways.

## Designs compared

Four designs are compared in the cost calculation

### Option 1: customer-managed hub and spoke

Hub VNet with Azure Firewall and ExpressRoute gateways per region. The maximum number of spokes per region is set to 390 (the actual number is 400, the maximum number of UDRs in the GatewaySubnet route table). If there are more spoke VNets than 390 in a single region, additional spoke blocks are created.

### Option 2: Virtual WAN with routing intent

Similar to option 1, but with Virtual WAN hubs instead of customer-managed hub VNets. The maximum number of spokes per hub is set to 600, as per the official documentation [here](https://learn.microsoft.com/azure/virtual-wan/how-to-routing-policies#address-limits).

### Option 3: indirect spokes with VWAN as core

Spoke VNets are peered to transit VNets with Azure Firewall. These transit VNets are connected to Virtual WAN, which is used as backbone. The advantage of this model is that you don't need to configure the peerings between the hubs, as you need to do in option 1.

### Option 4: Azure Virtual Network Manager (AVNM)

Similar to option 1, but using AVNM to manage the peerings. Additionally, a mesh network is created between the spokes of each spoke block. As per the [AVNM limits doc](https://learn.microsoft.com/azure/virtual-network-manager/concept-limitations), the spoke block limit is 1,000. In the simulation it is set to 990, to leave a bit of space.

## Conclusions

The following conclusions can be derived from simulations:

- Costs for traffic processing and VNet peering shouldn't be neglected, since they can make up a significant portion of the overall costs.
- AVNM is more expensive in most situations. The additional price might be justified by the automation brought by AVNM. AVNM can be cheaper than other options with many spoke VNets and large VNet-to-VNet (V2V) traffic volumes.
- The indirect spoke design (option 3) is almost always more expensive, due to the additional VNet peerings and the VWAN hub data processing costs.
- The Virtual WAN option and the customer-managed hub-and-spoke options are similarly priced, with one being cheaper than the other depending on traffic patterns.
- With large V2V traffic volumes and high amount of spoke VNets, Virtual WAN is usually cheaper than hub-and-spoke, due to the lower cost for inter-region and intra-region communication.
