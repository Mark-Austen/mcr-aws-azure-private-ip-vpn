This is a guide to deploy a Private IP VPN between AWS and Azure using the Megaport Cloud Router as the private transport layer.

The following resources are deployed:

* Megaport Cloud Router (MCR)
* Megaport Virtual Cross Connect (VXC) to AWS Direct Connect Hosted Connection
* Megaport Virtual Cross Connect (VXC) to Azure ExpressRoute
* AWS Direct Connect Gateway
* AWS Direct Connect Transit VIF
* AWS Transit Gateway
* AWS Direct Connect Gateway to Transit Gateway association
* AWS Transit Gateway attachment to VPC
* AWS Transit Gateway attachment to Direct Connect Gateway
* AWS Customer Gateway
* AWS Site-to-Site VPN
* AWS Transit Gateway attachment to VPN
* Azure ExpressRoute Circuit
* Azure ExpressRoute Virtual Gateway
* Azure ExpressRoute to ExpressRoute Virtual Gateway Connection
* Azure VNet Gateway Subnet
* Azure Local Gateway
* Azure VPN Virtual Gateway Connections

## Prerequisites

* Existing AWS VPC and subnet: [Link](https://docs.aws.amazon.com/vpc/latest/userguide/create-vpc.html#create-vpc-and-other-resources)
* Existing Azure VNet and subnet: [Link](https://learn.microsoft.com/en-us/azure/virtual-network/quick-create-portal)

## Deployment Instructions

### Step 1 - AWS Direct Connect Gateway

* From the AWS console search for **Direct Connect** in the search bar.
* Select **Direct Connect**.
* Select **Direct Connect gateway**.
* Select **Create Direct Connect gateway**.
* Enter a Name and Amazon-side ASN for the Direct Connect gateway.
* Select **Create Direct Connect gateway**.

### Step 2 - AWS Transit Gateway

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

### Step 4 - AWS Transit Gateway attachment to VPC

* From the AWS console search for **VPC** in the search bar.
* Select **VPC**.
* In the left hand column select **Transit gateway attachements**.
* Select **Create transit gateway attachement**.
* Select Transit gateway ID and VPC ID from the drop boxes.
* Select **Create transit gateway attachement**.

### Step 5 - Megaport Cloud Router

* From the Megaport portal select **Create MCR** to deploy a Megaport Cloud Router in the desired metro: [Link](https://docs.megaport.com/mcr/creating-mcr/)

### Step 6 - Megaport Virtual Cross Connect (VXC) to AWS Direct Connect Hosted Connection

* From the Megaport portal select **+Connection** on the newly created MCR. 
* Follow the workflow to deploy the AWS Direct Connect Hosted Connection VXC: [Link](https://docs.megaport.com/cloud/mcr/aws/#creating-a-hosted-connection)

### Step 7 - AWS Direct Connect Transit VIF

* From the AWS console search for **Direct Connect** in the search bar.
* Select **Direct Connect**.
* Select **Connections**.
* Select Virtual Cross Connect (VXC) created in step 6.
* Select **Accept** > **Confirm**.
* After state changes to available select **Create virtual interface**.
* Select Virtual interface type **Transit**.
* Enter **Virtual interface name**.
* Select accepted **Connection**.
* Select **Direct Connect gateway** deployed in step 1.
* Enter **BGP ASN** of the MCR, in this example ASN 133937.
* Select **Additional settings**.
* Enter **Your router peer ip**, in this example the IP address of the MCR - 192.168.100.1/30.
* Enter **Amazon router peer ip**, in this example - 192.168.100.2/30.
* Enter the **BGP Authentication key** created in step 6.
* Select **Create virtual interface**.
* Select the Virtual interface from the interface list, the state will transition to **available** and BGP status to **up**.

### Step 8 - Confirm MCR has recevied AWS Prefixes

* From the Megaport portal select select the MCR Looking Glass.
* The route table should contain the AWS VPC CIDR, in this example 10.1.0.0/16, and the Transit gateway CIDR, in this example 10.254.1.0/24.
