This is a guide to deploy a Private IP IPSec VPN between AWS and Azure using the Megaport Cloud Router (MCR) as the private transport layer.

The following resources are deployed:

* Megaport Cloud Router (MCR)
* Megaport Virtual Cross Connect (VXC) to AWS Direct Connect Hosted Connection
* Megaport Virtual Cross Connect (VXC) to Azure ExpressRoute
* AWS Direct Connect Gateway
* AWS Direct Connect Transit VIF
* AWS Transit Gateway
* AWS Customer Gateway
* AWS Site-to-Site VPN
* AWS Direct Connect Gateway to Transit Gateway association
* AWS Transit Gateway attachment to VPC
* AWS Transit Gateway attachment to Direct Connect Gateway
* AWS Transit Gateway attachment to VPN
* Azure ExpressRoute Circuit
* Azure ExpressRoute Virtual Network Gateway
* Azure ExpressRoute to ExpressRoute Virtual Gateway Connection
* Azure VPN Virtual Network Gateway
* Azure VNet Gateway Subnet
* Azure Local Gateway
* Azure VPN Virtual Network Gateway Connections

## Prerequisites

* Megaport account
* AWS account
* Azure subscription
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

### Step 9 - Azure ExpressRoute Circuit

* From the Azure portal search for **ExpressRoute** in the search bar.
* Select **ExpressRoute circuits**.
* Select **Create ExpressRoute**.
* Select **Subscription**.
* Select **Resource Group**.
* Select **Region**, in this example - Australia East.
* Enter **Circuit name**.
* Select Megaport in the **Provider** box.
* Select **Peering location**, in this example - Sydney.
* Select **Bandwidth**, in this example - 50Mbps.
* Leave SKU as **Standard**, and billing model as **Metered**.
* Select **Review + create** > **Create**.
* When deployment is complete select **Go to resource**.
* From the ExpressRoute details copy the service key for use in the next step.

### Step 10 - Megaport Virtual Cross Connect (VXC) to Azure ExpressRoute

* From the Megaport portal select **+Connection** on the newly created MCR.
* Follow the workflow to deploy the Azure ExpressRoute VXC: [Link](https://docs.megaport.com/cloud/megaport/microsoft/#creating-an-expressroute-connection)

### Step 11 - Azure ExpressRoute Gateway

* From the Azure portal search for **Virtual network gateways** in the search bar.
* Select **Virtual network gateways** > **Create virtual network gateway**.
* Enter a **Name**.
* Select a **Region**, in this example - Australia East.
* Select **Gateway type** > **ExpressRoute**.
* Select **SKU**, in this example - Standard. SKU types: [Link](https://learn.microsoft.com/en-us/azure/expressroute/expressroute-about-virtual-network-gateways#gwsku)
* Select **Virtual network**.
* Enter **Gateway subnet address range**, in this example - 10.2.254.0/24.
* Select **Create new** Public IP address.
* Enter Public IP address name.
* Select **Review + create** > **Create**.

### Step 12 - Azure ExpressRoute to ExpressRoute Virtual Gateway Connection

* From the Azure portal search for **ExpressRoute** in the search bar.
* Select **ExpressRoute circuits**.
* Select the ExpressRoute Circuit created in step 9.
* In the left hand column select **Connections**.
* Select **Add**.
* Select **Connection type** ExpressRoute.
* Enter a name for the connection.
* Select **Next**.
* Select **Virtual network gateway** created in step 11.
* Select **ExpressRoute circuit** created in step 9.
* Select **Review + create** > **Create**.

### Step 13 - Confirm MCR has recevied Azure Prefixes

* From the Megaport portal select select the MCR Looking Glass.
* The route table should contain the Azure VNet CIDR, in this example 10.2.0.0/16.



