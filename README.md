This is a guide to deploy a Private IP VPN between AWS and Azure using the Megaport Cloud Router as the private transport layer.

The following resources are deployed:

* Megaport Cloud Router (MCR)
* Megaport Virtual Cross Connect (VXC) to AWS Direct Connect Hosted Connection
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
* Megaport Virtual Cross Connect (VXC) to Azure ExpressRoute
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

### Step 1 - Deploy Megaport Cloud Router

* From the Megaport portal select "Create MCR" to deploy a Megaport Cloud Router in the desired metro: [Link](https://docs.megaport.com/mcr/creating-mcr/)

### Step 2 - Deploy AWS Direct Connect Hosted Connection VXC

* From the Megaport portal select **+Connection** on the newly created MCR. 
* Follow the workflow to deploy the AWS Direct Connect Hosted Connection VXC: [Link](https://docs.megaport.com/cloud/mcr/aws/#creating-a-hosted-connection)

### Step 3 - Deploy AWS Direct Connect Gateway

* From the AWS console search for **Direct Connect** in the search bar.
* Select **Direct Connect**.
* Select **Direct Connect gateway**.
* Select **Create Direct Connect gateway**.
* Enter a Name and Amazon-side ASN for the Direct Connect gateway.
* Select **Create Direct Connect gateway**.

### Step 4 - Deploy AWS Transit Gateway

* From the AWS console search for **Transit gateways** in the search bar.
* Select **Transit gateways**.
* Select **Create transit gateways**.
* Enter a Name tag and Transit gateway CIDR block, e.g. 10.255.1.0/24.
* Select **Create transit gateway**.



