
This is a guide to deploy a Private IP VPN between AWS and Azure using the Megaport Cloud Router (MCR) as the private transport layer.

The following resources are deployed:

* Megaport Cloud Router (MCR)
* Megaport Virtual Cross Connect (VXC) to AWS Direct Connect Hosted Connection
* Megaport Virtual Cross Connect (VXC) to Azure ExpressRoute
* AWS Direct Connect Gateway
* AWS Direct Connect Transit VIF
* AWS Transit Gateway
* AWS VPN - Customer Gateway
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

## Architecture Overview

![image](https://github.com/Mark-Austen/mcr-aws-azure-private-ip-vpn/assets/117334224/a3fd5488-e34a-446b-bd54-bda03e82432b)

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
* Enter a Name tag and Transit gateway CIDR block, in this example `10.254.1.0/24`.
* Select **Create transit gateway**.

### Step 3 - AWS Direct Connect Gateway to Transit Gateway association

* From the AWS console search for **Direct Connect** in the search bar.
* Select **Direct Connect**.
* Select **Direct Connect gateway**.
* Select Direct Connect gateway created in step 1.
* Select **Gateway associations** > **Associate gateway**.
* Select the Transit gateway created in step 2.
* Enter the AWS VPC CIDR and the Transit gateway CIDR in **Allowed Prefixes**.

### Step 4 - AWS Transit Gateway attachment to VPC

* From the AWS console search for **VPC** in the search bar.
* Select **VPC**.
* In the left hand column select **Transit gateway attachements**.
* Select **Create transit gateway attachement**.
* Select Transit gateway ID created in step 2, and VPC ID from the drop boxes.
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
* Select the Virtual Cross Connect (VXC) created in step 6.
* Select **Accept** > **Confirm**.
* After the state changes to available select **Create virtual interface**.
* Select Virtual interface type **Transit**.
* Enter **Virtual interface name**.
* Select the **Connection**.
* Select **Direct Connect gateway** deployed in step 1.
* Enter **BGP ASN** of the MCR, in this example ASN `133937`.
* Select **Additional settings**.
* Enter **Your router peer ip**, in this example the IP address of the MCR `192.168.100.1/30`.
* Enter **Amazon router peer ip**, in this example `192.168.100.2/30`.
* Enter the **BGP Authentication key** created in step 6.
* Select **Create virtual interface**.
* Select the Virtual interface from the interface list, the state will transition to **available** and BGP status to **up**.

### Step 8 - Confirm MCR has recevied AWS Prefixes

* From the Megaport portal select select the MCR Looking Glass.
* The route table should contain the AWS VPC CIDR, in this example `10.1.0.0/16`, and the Transit gateway CIDR, in this example `10.254.1.0/24`.

### Step 9 - Azure ExpressRoute Circuit

* From the Azure portal search for **ExpressRoute** in the search bar.
* Select **ExpressRoute circuits**.
* Select **Create ExpressRoute**.
* Select **Subscription**.
* Select **Resource Group**.
* Select **Region**, in this example `Australia East`.
* Enter **Circuit name**.
* Select Megaport in the **Provider** box.
* Select **Peering location**, in this example `Sydney`.
* Select **Bandwidth**, in this example `50Mbps`.
* Leave SKU as **Standard**, and billing model as **Metered**.
* Select **Review + create** > **Create**.
* When deployment is complete select **Go to resource**.
* From the ExpressRoute details page copy the service key for use in the next step.

### Step 10 - Megaport Virtual Cross Connect (VXC) to Azure ExpressRoute

* From the Megaport portal select **+Connection** on the newly created MCR.
* Follow the workflow to deploy the Azure ExpressRoute VXC: [Link](https://docs.megaport.com/cloud/megaport/microsoft/#creating-an-expressroute-connection)

### Step 11 - Azure ExpressRoute Virtual Network Gateway

* From the Azure portal search for **Virtual network gateways** in the search bar.
* Select **Virtual network gateways** > **Create virtual network gateway**.
* Enter a **Name**.
* Select a **Region**, in this example Australia East.
* Select **Gateway type** > **ExpressRoute**.
* Select **SKU**, in this example Standard. SKU types: [Link](https://learn.microsoft.com/en-us/azure/expressroute/expressroute-about-virtual-network-gateways#gwsku)
* Select **Virtual network**.
* Enter **Gateway subnet address range**, in this example `10.2.0.0/24`.
* Select **Create new** Public IP address.
* Enter a **Public IP address** name.
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
* The route table should contain the Azure VNet CIDR, in this example `10.2.0.0/16`.

### Step 14 - Azure VPN Virtual Network Gateway

* From the Azure portal search for **Virtual network gateways** in the search bar.
* Select **Virtual network gateways** > **+Create**.
* Enter a **Name**.
* Select a **Region**, in this example Australia East.
* Select **Gateway type** > **VPN**.
* Select **SKU**, in this example `VpnGw1`. SKU types: [Link](https://learn.microsoft.com/en-us/azure/expressroute/expressroute-about-virtual-network-gateways#gwsku)
* Select **Generation**, in this example `Generation 1`.
* Select **Virtual network**.
* Enter a **Public IP Address** name.
* Select **Enable active-active mode** Disabled.
* Select **Configure BGP** Enabled.
* In this example use default BGP ASN `65515`.
* Enter two **Custom Azure APIPA BGP IP address**, in this example `169.254.21.2`, and `169.254.22.2`.
* Select **Review + create** > **Create**. (Deployment time is ~20min)

![image](https://github.com/Mark-Austen/mcr-aws-azure-private-ip-vpn/assets/117334224/6819ea35-de3d-48bf-a08f-151e4057c8bd) 

### Step 15 - Azure VPN Virtual Network Gateway - Enable Gateway Private IPs

* From the Azure portal search for **Virtual network gateways** in the search bar.
* Select the VPN Virtual network gateway created in step 14.
* In the left hand column select **Configuration**.
* Enable **Gateway Private IPs**.
* Select **Save**. (Virtual network gateway redeploys ~20min).

<img width="1223" alt="image" src="https://github.com/Mark-Austen/mcr-aws-azure-private-ip-vpn/assets/117334224/69579631-6a89-457d-ace4-800e01808b42">

* From the Virtual network gateway overview page select **See more** copy the **First Private IP address** for the next step, in this example `10.2.0.6`.

![image](https://github.com/Mark-Austen/mcr-aws-azure-private-ip-vpn/assets/117334224/77fbd7af-970a-4d22-b8d9-31e29fcd0f8d)

### Step 16 - AWS VPN Customer Gateway

* From the AWS console search for **VPC** in the search bar.
* Select **VPC**.
* In the left hand column select Customer Gateways.
* Select **Create customer gateway**.
* Enter the **BGP ASN** of the Azure VPN Virtual Network Gateway.
* Enter the **IP address** of the Azure VPN Virtual Network Gateway.
* Select **Create customer gateway**.

### Step 17 - AWS Site-to-Site VPN

* From the AWS console search for **VPC** in the search bar.
* Select **VPC**.
* In the left hand column select Virtual private gateways.
* Select **Site-to-Site VPN connections** > **Create VPN connection**.
* Select **Target gateway type** Transit gateway.
* Select the Transit gateway created in step 2.
* Select the **Customer gateway ID** created in step 16.
* Select **Outside IP address type** PrivateIpv4.
* Select the **Transport transit gateway attachment ID** created in step step 3.
* Expand **Tunnel 1 options** > enter **Inside IPv4 CIDR for tunnel 1**, in this example `169.254.21.0/30` (AWS takes lower IP).
* Enter **Pre-shared key for tunnel 1**.
* Expand **Tunnel 2 options** > enter **Inside IPv4 CIDR for tunnel 2**, in this example `169.254.22.0/30` (AWS takes lower IP).
* Enter **Pre-shared key for tunnel 2**.
* Select **Create VPN connection**.
* From the main VPN page select the VPN, from the **Tunnel state** details copy the Tunnel 1 and Tunnel 2 outside IP address for later use.

### Step 18 - Megaport Cloud Router Static Routes

* From the Megaport portal select the AWS Virtual Cross Connect **VXC Details** button.
* Select **A-End**.
* Enter a **Static Route** for the AWS Tunnel 1 outside IP address, in this example `10.254.1.53/32`, use the **Peer IP** as the **Next Hop**.
* Enter a **Static Route** for the AWS Tunnel 2 outside IP address, in this example `10.254.1.57/32`, use the **Peer IP** as the **Next Hop**.
* Select **Save** > **Close**.
* Select the Azure Virtual Cross Connect **VXC Details** button.
* Select **A-End**.
* Enter a **Static Route** for the Azure Tunnel outside IP address, in this example `10.2.0.6/32`, use the **Peer IP** as the **Next Hop**.
* Select **Save** > **Close**.

### Step 19 - Megaport Cloud Router Prefix Filter Lists

* From the Megaport portal select the MCR **MCR Details** button.
* Select **Prefix Filter Lists**.
* Select **New List**.
* Enter name **Azure-Export**.
* Enter an exact match entry for `10.254.1.50/32`, and `10.254.1.57/32`.
* Select **Save**
* Select **New List**.
* Enter name **AWS-Export**.
* Enter an exact match entry for `10.2.0.6/32`.
* Select **Save** > **Close**.
* Select the AWS Virtual Cross Connect **VXC Details** button.
* Select **A-End**.
* Select **Edit** for the BGP session.
* Select **Filters**
* Under **Export Prefix Filter** select **Permit List**, in the drop down box select **AWS-Export**.
* Select **Update** > **Close**.
* Select the Azure Virtual Cross Connect **VXC Details** button.
* Select **A-End**.
* Select **Edit** for the BGP session.
* Select **Filters**
* Under **Export Prefix Filter** select **Permit List**, in the drop down box select **Azure-Export**.
* Select **Update** > **Close**.
* Select **Tools** in the top menu bar > **MCR Looking Glass**.
* Check **Neighbour Routes** for each BGP session to ensure only the tunnel outside prefixes are advertsied.

### Step 20  - Azure Local Network Gateway (1 of 2)

* From the Azure portal search for **Local network gateways** in the search bar.
* Select **Local network gateways**.
* Select **Create local network gateway**.
* Select **Subscription**.
* Select **Resource Group**.
* Select a **Region**, in this example Australia East.
* Enter a **Name**.
* Enter the AWS Tunnel 1 outside IP address, in this example `10.254.1.53`.
* Select **Next: Advanced >**
* Select **Yes** for **Configure BGP settings**.
* Enter the **Autonomous system number (ASN)** of the AWS Transit Gateway, in this example `65534`.
* Enter the **BGP peer IP address** which is the inside IP address of Tunnel 1, in this example `169.254.21.1`.
* Select **Next: Review + create** > **Create**.

### Step 21  - Azure Local Network Gateway (2 of 2)

* From the Azure portal search for **Local network gateways** in the search bar.
* Select **Local network gateways**.
* Select **Create local network gateway**.
* Select **Subscription**.
* Select **Resource Group**.
* Select a **Region**, in this example Australia East.
* Enter a **Name**.
* Enter the AWS Tunnel 2 outside IP address, in this example `10.254.1.57`.
* Select **Next: Advanced >**
* Select **Yes** for **Configure BGP settings**.
* Enter the **Autonomous system number (ASN)** of the AWS Transit Gateway, in this example `65534`.
* Enter the **BGP peer IP address** which is the inside IP address of Tunnel 1, in this example `169.254.22.1`.
* Select **Next: Review + create** > **Create**.

### Step 22  - Azure VPN Virtual Network Gateway - VPN Connection (1 of 2)

* From the Azure portal search for **Virtual network gateways** in the search bar.
* Select the VPN Virtual network gateway.
* Select **Connections** > **Add**.
* Select **Connection type** as **Site-to-site VPN (IPsec).
* Select **Next : Settings >**
* Select the VPN Virtual network gateway in the drop down box.
* Select the first Local network gatewat in the drop down box.
* Enter the **Shared Key(PSK)**
* Tick **Use Azure Private IP Address**
* Tick **Enable BGP**
* Tick **Enable Custom BGP Addresses**
* In the **Primary Custom BGP Address** box select `169.254.21.2`.
* Select **Review + create** > **Create**.

### Step 23  - Azure VPN Virtual Network Gateway - VPN Connection (2 of 2)

* From the Azure portal search for **Virtual network gateways** in the search bar.
* Select the VPN Virtual network gateway.
* Select **Connections** > **Add**.
* Select **Connection type** as **Site-to-site VPN (IPsec).
* Select **Next : Settings >**
* Select the VPN Virtual network gateway in the drop down box.
* Select the second Local network gatewat in the drop down box.
* Enter the **Shared Key(PSK)**
* Tick **Use Azure Private IP Address**
* Tick **Enable BGP**
* Tick **Enable Custom BGP Addresses**
* In the **Primary Custom BGP Address** box select `169.254.22.2`.
* Select **Review + create** > **Create**.










