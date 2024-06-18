This is a guide to deploy a Private IP VPN between AWS and Azure using the Megaport Cloud Router as the private transport layer.

The following resources are deployed:

* Megaport Cloud Router (MCR)
* Megaport Virtual Cross Connect (VXC) to AWS Direct Connect Hosted Connection
* AWS Direct Connect Gateway
* AWS Transit Gateway
* AWS Direct Connect Gateway to Transit Gateway association
* AWS Customer Gateway
* AWS Site-to-Site VPN
* AWS Transit Gateway attachment to VPC
* AWS Transit Gateway attachment to Direct Connect Gateway
* AWS Transit Gateway attachment to VPN
* Azure ExpressRoute Circuit
* Megaport Virtual Cross Connect (VXC) to Azure ExpressRoute
* Azure ExpressRoute Virtual Gateway
* Azure ExpressRoute to ExpressRoute Virtual Gateway Connection
* Azure VNet Gateway Subnet
* Azure Local Gateway
* Azure VPN Virtual Gateway Connections

## Prerequisites

* AWS VPC: [Link](https://docs.aws.amazon.com/vpc/latest/userguide/create-vpc.html#create-vpc-and-other-resources)
* Azure VNet: [Link](https://learn.microsoft.com/en-us/azure/virtual-network/quick-create-portal)

## Deployment Instructions

### Step 1 - Deploy AWS Direct Connect Gateway

* From the AWS console search for **Direct Connect** in the search bar.
* Select **Direct Connect**.
* Select **Direct Connect gateway**.
* Select **Create Direct Connect gateway**.
* Enter a Name and Amazon-side ASN for the Direct Connect gateway.
* Select **Create Direct Connect gateway**.

### Step 2 - Deploy AWS Transit Gateway

* From the AWS console search for **Transit gateways** in the search bar.
* Select **Transit gateways**.
* Select **Create transit gateways**.
* Ensure the correct region is selected.
* Enter a Name tag and Transit gateway CIDR block, in this guide lets use 10.254.1.0/24.
* Select **Create transit gateway**.

### Step 3 - AWS Direct Connect Gateway to Transit Gateway association

* From the AWS console search for **Direct Connect** in the search bar.
* Select **Direct Connect**.
* Select **Direct Connect gateway**.
* Select Direct Connect gateway created in step 1.
* Select Gateway associations > Associated gateway.
* Select the Transit gateway created in step 2.
* Enter the AWS VPC CIDR and the Transit gateway CIDR in **Allowed Prefixes**.



### Step X - Deploy Megaport Virtual Cross Connect (VXC) to AWS Direct Connect Hosted Connection

* From the Megaport portal select **+Connection** on the newly created MCR. 
* Follow the workflow to deploy the AWS Direct Connect Hosted Connection VXC: [Link](https://docs.megaport.com/cloud/mcr/aws/#creating-a-hosted-connection)

### Step x - Deploy Megaport Cloud Router

* From the Megaport portal select "Create MCR" to deploy a Megaport Cloud Router in the desired metro: [Link](https://docs.megaport.com/mcr/creating-mcr/)

