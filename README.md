This is a guide to deploy a Private IP VPN between AWS and Azure using the Megaport Cloud Router as the private transport layer.

The following resources are deployed:

* Megaport Cloud Router (MCR)
* Megaport Virtual Cross Connect (VXC) to AWS Direct Connect
* Megaport Virtual Cross Connect (VXC) to Azure ExpressRoute
* AWS Direct Connect Hosted Connection
* AWS Direct Connect Gateway
* AWS Transit Gateway
* AWS Direct Connect Gateway to Transit Gateway association
* AWS VPC
* AWS VPC Subnet
* AWS Customer Gateway
* AWS Site-to-Site VPN
* AWS Transit Gateway attachment to VPC
* AWS Transit Gateway attachment to Direct Connect Gateway
* AWS Transit Gateway attachment to VPN
* Azure Resource Group
* Azure ExpressRoute Circuit
* Azure ExpressRoute Virtual Gateway
* Azure ExpressRoute to ExpressRoute Virtual Gateway Connection
* Azure VNet
* Azure VNet Subnet
* Azure VNet Gateway Subnet
* Azure Local Gateway
* Azure VPN Virtual Gateway Connections

## Prerequisites

* Megaport acount
* AWS account
* Azure account

## Deployment Instructions

### Step 1

From the Megaport portal select "Create MCR" to deploy a Megaport Cloud Router in the desired metro: [Link](https://docs.megaport.com/mcr/creating-mcr/)

