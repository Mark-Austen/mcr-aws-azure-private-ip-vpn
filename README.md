
This is a guide to deploy a Private IP VPN between AWS and Azure using the Megaport Cloud Router (MCR) as the private transport layer.

AWS and Azure Private IP VPNs enable the use of IPSec VPNs across AWS Direct Connect and Azure ExpressRoute. Intergrated with the Megaport Cloud Router, traffic is encrypted across Megaports global private network. This solution provides secure, private, high performance multicloud network connectivity.

In this guide the following resources are deployed:

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

<img width="1377" alt="image" src="https://github.com/Mark-Austen/mcr-aws-azure-private-ip-vpn/assets/117334224/1a4bf430-1e7d-45a5-b367-79f6bd8145ab">

* Megaport Cloud Router (MCR) provides the virtual routing connection between AWS Direct Connect and Azure ExpressRoute Circuit cloud on-ramps.
* Virtual Cross Connects (VXCs) are a point to point connection from the MCR to AWS Direct Connect and Azure ExpressRoute Circuit.
* BGP Peering is used between the MCR and the AWS Direct Connect Gateway/Transit VIF and the Azure ExpressRoute MSEE routers.
* Azure ExpressRoute Circuit provides a primary and secondary connection, in this example only the primary is used.
* AWS Direct Connect Gateway is associated with the Transit Gateway. BGP ASNs for the DGW and TGW have been configured as shown on the diagram.
* Transit Gateway provides the IP VPN functionality across Direct Connect by use of outside private IP addressing on the IPSec tunnels. It also provides the routing between the VPCs, Direct Connect, and IP VPN.
* Azure ExpressRoute Circuit has a connection to the ExpressRoute Virtual Network Gateway. BGP ASNs for the MSEE routers is 12076 and is not configurable, the ExpressRoute Virtual Network Gateway does not have an ASN defined.
* Azure VPN Virtual Network Gateway provides the IP VPN functionality across ExpressRoute by use of outside private IP addressing on the IPSec tunnels.

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

<img width="1287" alt="image" src="https://github.com/Mark-Austen/mcr-aws-azure-private-ip-vpn/assets/117334224/5f05bbf7-c8c6-455a-9882-c4d2391ccebf">

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

<img width="1295" alt="image" src="https://github.com/Mark-Austen/mcr-aws-azure-private-ip-vpn/assets/117334224/cbb76964-2ce1-4c53-bb39-8131ffd8d0e7">

<img width="1322" alt="image" src="https://github.com/Mark-Austen/mcr-aws-azure-private-ip-vpn/assets/117334224/6a544437-b9cd-40ff-b9d2-8d9ccd5c046b">

<img width="1317" alt="image" src="https://github.com/Mark-Austen/mcr-aws-azure-private-ip-vpn/assets/117334224/4d8b5488-48ab-4b27-a0ca-29db74767036">

![image](https://github.com/Mark-Austen/mcr-aws-azure-private-ip-vpn/assets/117334224/f5cd2f05-6988-4098-9098-9738ade10ab7)

### Step 18 - Megaport Cloud Router Static Routes

* From the Megaport portal select the AWS Virtual Cross Connect **VXC Details** button.
* Select **A-End**.
* Enter a **Static Route** for the AWS Tunnel 1 outside IP address, in this example `10.254.1.18/32`, use the **Peer IP** as the **Next Hop**.
* Enter a **Static Route** for the AWS Tunnel 2 outside IP address, in this example `10.254.1.80/32`, use the **Peer IP** as the **Next Hop**.
* Select **Save** > **Close**.
* Select the Azure Virtual Cross Connect **VXC Details** button.
* Select **A-End**.
* Enter a **Static Route** for the Azure Tunnel outside IP address, in this example `10.2.0.6/32`, use the **Peer IP** as the **Next Hop**.
* Select **Save** > **Close**.

![image](https://github.com/Mark-Austen/mcr-aws-azure-private-ip-vpn/assets/117334224/2b723e14-4f6f-4866-900f-f5c325ca0dfa)

![image](https://github.com/Mark-Austen/mcr-aws-azure-private-ip-vpn/assets/117334224/b2f7ad3c-8580-48c9-ba18-794c85efb322)

### Step 19 - Megaport Cloud Router Prefix Filter Lists

* From the Megaport portal select the MCR **MCR Details** button.
* Select **Prefix Filter Lists**.
* Select **New List**.
* Enter name **Azure-Export**.
* Enter an exact match entry for `10.254.1.18/32`, and `10.254.1.80/32`.
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
* Select **Update** > **Save** > **Close**.
* Select the Azure Virtual Cross Connect **VXC Details** button.
* Select **A-End**.
* Select **Edit** for the BGP session.
* Select **Filters**
* Under **Export Prefix Filter** select **Permit List**, in the drop down box select **Azure-Export**.
* Select **Update** > **Save** > **Close**.
* Select **Tools** in the top menu bar > **MCR Looking Glass**.
* Check **Neighbour Routes** for each BGP session to ensure only the tunnel outside prefixes are advertsied.

![image](https://github.com/Mark-Austen/mcr-aws-azure-private-ip-vpn/assets/117334224/a18cfb41-0fdc-4e7e-9f00-3bb36f87a2b0)

![image](https://github.com/Mark-Austen/mcr-aws-azure-private-ip-vpn/assets/117334224/1d89068b-cefe-452c-b7bb-9c0b84121136)

<img width="1270" alt="image" src="https://github.com/Mark-Austen/mcr-aws-azure-private-ip-vpn/assets/117334224/6c3825e1-9469-479c-b1d5-c469aa95a3d1">

<img width="1270" alt="image" src="https://github.com/Mark-Austen/mcr-aws-azure-private-ip-vpn/assets/117334224/0ce771f1-2f2d-4520-965c-8dbd99315186">

<img width="1282" alt="image" src="https://github.com/Mark-Austen/mcr-aws-azure-private-ip-vpn/assets/117334224/7e1115ff-6594-4c1c-962e-cc280d5708de">

<img width="1296" alt="image" src="https://github.com/Mark-Austen/mcr-aws-azure-private-ip-vpn/assets/117334224/6b794d18-75a0-429a-952d-3903c6744fb2">

### Step 20  - Azure Local Network Gateway (1 of 2)

* From the Azure portal search for **Local network gateways** in the search bar.
* Select **Local network gateways**.
* Select **Create local network gateway**.
* Select **Subscription**.
* Select **Resource Group**.
* Select a **Region**, in this example Australia East.
* Enter a **Name**.
* Enter the AWS Tunnel 1 outside IP address, in this example `10.254.1.18`.
* Select **Next: Advanced >**
* Select **Yes** for **Configure BGP settings**.
* Enter the **Autonomous system number (ASN)** of the AWS Transit Gateway, in this example `65534`.
* Enter the **BGP peer IP address** which is the inside IP address of Tunnel 1, in this example `169.254.21.1`.
* Select **Next: Review + create** > **Create**.

![image](https://github.com/Mark-Austen/mcr-aws-azure-private-ip-vpn/assets/117334224/de24c5e9-d142-4076-bdf7-5c1f9274545c)

![image](https://github.com/Mark-Austen/mcr-aws-azure-private-ip-vpn/assets/117334224/753ff4b2-d2d1-43d1-9dbf-049a0e350865)

### Step 21  - Azure Local Network Gateway (2 of 2)

* From the Azure portal search for **Local network gateways** in the search bar.
* Select **Local network gateways**.
* Select **Create local network gateway**.
* Select **Subscription**.
* Select **Resource Group**.
* Select a **Region**, in this example Australia East.
* Enter a **Name**.
* Enter the AWS Tunnel 2 outside IP address, in this example `10.254.1.80`.
* Select **Next: Advanced >**
* Select **Yes** for **Configure BGP settings**.
* Enter the **Autonomous system number (ASN)** of the AWS Transit Gateway, in this example `65534`.
* Enter the **BGP peer IP address** which is the inside IP address of Tunnel 1, in this example `169.254.22.1`.
* Select **Next: Review + create** > **Create**.

<img width="1224" alt="image" src="https://github.com/Mark-Austen/mcr-aws-azure-private-ip-vpn/assets/117334224/8af73ebe-8601-4004-a5aa-37aef1c8c434">

<img width="1225" alt="image" src="https://github.com/Mark-Austen/mcr-aws-azure-private-ip-vpn/assets/117334224/0271f1b1-6f59-4c92-bf3b-4d956bd9d622">

### Step 22  - Azure VPN Virtual Network Gateway - VPN Connection (1 of 2)

* From the Azure portal search for **Virtual network gateways** in the search bar.
* Select the VPN Virtual network gateway.
* Select **Connections** > **Add**.
* Select **Connection type** as Site-to-site VPN (IPsec).
* Select **Next : Settings >**
* Select the VPN Virtual network gateway in the drop down box.
* Select the first Local network gatewat in the drop down box.
* Enter the **Shared Key(PSK)**
* Tick **Use Azure Private IP Address**
* Tick **Enable BGP**
* Tick **Enable Custom BGP Addresses**
* In the **Primary Custom BGP Address** box select `169.254.21.2`.
* Select **Review + create** > **Create**.

<img width="1265" alt="image" src="https://github.com/Mark-Austen/mcr-aws-azure-private-ip-vpn/assets/117334224/aab3145e-e43b-4b26-a96e-60b02b1aa242">

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

<img width="1269" alt="image" src="https://github.com/Mark-Austen/mcr-aws-azure-private-ip-vpn/assets/117334224/2044ff11-d0c5-445d-8279-6819c737533f">

### Step 24  - Verification

* From the Azure portal in **Virtual network gateways** > *gateway name* > **Connections**, status for both VPN connections should be **Connected**.

![image](https://github.com/Mark-Austen/mcr-aws-azure-private-ip-vpn/assets/117334224/30894433-4948-42a3-9272-73ee43247a05)

* From the AWS console in **VPC** > **VPN connections** > *VPN ID*, the tunnel state status for both tunnels should be **Up**.

![image](https://github.com/Mark-Austen/mcr-aws-azure-private-ip-vpn/assets/117334224/fb655873-a60b-4914-b28e-ccee7b6fcd48)

* From the AWS console in **VPC** > **Transit gateway route tables** > *Transit gateway route table ID*, in the **Routes** tab, there are two routes of significance, in this example the Azure VNet CIDR `10.2.0.0/16`reachable via the VPN attachment, and the Azure VPN tunnel outside IP address `10.2.0.6` reachable via the Direct Connect Gateway attachment.

<img width="1305" alt="image" src="https://github.com/Mark-Austen/mcr-aws-azure-private-ip-vpn/assets/117334224/86c71031-9a1d-4b44-94c7-4dfb1ed12467">

* From the Azure portal in  **Virtual network gateways** > *gateway name* > **BGP Peers**, the BGP statis of the Transit Gateway BGP sessions should be **Connected**, and the AWS VPC CIDR `10.1.0.0/16` reachable via BGP using the two tunnels.

![image](https://github.com/Mark-Austen/mcr-aws-azure-private-ip-vpn/assets/117334224/23df29a6-7395-4293-bb0c-f1f0bbd8ae46)












