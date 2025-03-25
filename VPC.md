# Virtual Private Cloud (VPC)

## DHCP 

AWS uses DHCP (Dynamic Host Configuration Protocol) to automatically provide instances in a VPC with critical network details (IP addresses, DNS servers, default gateways (VPC router address), etc.).
This process relies on an L2 broadcast model, where an instance broadcasts a request and a DHCP server responds with configuration parameters.
In AWS, DHCP Option Sets define the specific DNS server addresses, domain names, NTP servers, and other network settings.
A single DHCP option set can be attached to multiple VPCs, but each VPC can only use one option set at a time.
Option sets themselves are immutable: if you need changes, you create a new set and associate it with the VPC.
Although the association is immediate, instances typically need a DHCP lease renewal to pick up the updated settings.
By default, AWS supplies both public and private DNS resolution (VPC’s built-in DNS resolver at the `.2` address (e.g., `10.0.0.2` in a `10.0.0.0/16` VPC)); you only need to configure a custom DNS if you want custom domain names or specific DNS servers.

## VPC Router

A VPC Router is an internal AWS component that routes traffic within a VPC and to external destinations.
Every subnet automatically gets a router interface with an address one higher than the subnet’s base (for a `10.16.32.0/24` subnet, the router is `10.16.32.1`).
Route Tables define where traffic should go.
Each VPC starts with one Main Route Table used by all subnets unless you explicitly override it by assigning a custom table to a subnet.
A subnet can only reference one route table at a time.
Routes follow the most specific match principle, so the detailed (longest-prefix) route is picked first.
You can configure routes for internal subnet traffic, on-premises connections (via VPN or Direct Connect), and public internet access (through an Internet Gateway).


## Network Access Control Lists

Network Access Control Lists (NACLs) are stateless, subnet-level security layers in Amazon VPC that control inbound and outbound traffic. 
Each NACL contains a set of rules defined by rule number, protocol, port range, source/destination, and action (allow or deny). 
Because they are stateless, return traffic must be explicitly allowed by separate rules in the opposite direction. 
NACLs are evaluated in order of rule number from lowest to highest, and the first matching rule is applied. 
If no rule matches, the default action is deny.

Each subnet must be associated with a NACL, and by default, it uses the default NACL which allows all traffic in and out. 
Custom NACLs start with all traffic denied by default until rules are added. 
NACLs apply to all traffic entering and leaving the subnet, including that between instances in different subnets.


## Security Groups

Security groups are stateful, instance-level virtual firewalls in Amazon VPC that control inbound and outbound traffic to Elastic Network Interfaces (ENIs). 
Each rule specifies protocol, port range, and source or destination, which can be an IP range or another security group. 
Being stateful means return traffic is automatically allowed, so only the initiating direction needs to be explicitly permitted.

Security groups operate at the instance level and are evaluated as a permissive set; they implicitly deny all traffic unless an allow rule exists.


## Local Zones

AWS Local Zones are an extension of AWS Regions that place compute and storage closer to large population centers, enabling low-latency access to applications that require single-digit millisecond response times. 
Each Local Zone is associated with a parent AWS Region and inherits its control plane, meaning management and orchestration happen through the parent Region, while data plane resources operate locally.

They support services like EC2, EBS, ALB/NLB, ECS, EKS, and Direct Connect, making them suitable for latency-sensitive workloads such as media processing, gaming, real-time analytics, or hybrid environments with on-premises infrastructure.


## Global Accelerator

AWS Global Accelerator is a global networking service that improves availability and performance for internet-facing applications by routing user traffic through the AWS global network. 
It provides two static anycast IP addresses (static IPs that route user traffic via closest AWS edge location) that serve as fixed entry points, abstracting away underlying regional endpoints and allowing seamless failover and routing optimization.

It uses AWS edge locations to terminate client connections and then routes traffic over the AWS backbone to the optimal endpoint, such as an Application Load Balancer, Network Load Balancer, or EC2 instance, based on health, geographic proximity, and routing policies. 

Global Accelerator operates at Layer 4 (TCP/UDP), making it protocol-agnostic and suitable for both web and non-web applications. 
Unlike Amazon CloudFront, which caches content and operates at Layer 7, Global Accelerator focuses on improving latency, throughput, and failover for active connections.


## Site-to-Site VPN

A Site-to-Site VPN establishes a secure, encrypted connection between a VPC and an on-premises network.
This connection runs over the public internet, ensuring data confidentiality and integrity while facilitating seamless communication between the two environments.

At the core of this setup are two key components: the Virtual Private Gateway (VGW) and the Customer Gateway (CGW).
The Virtual Private Gateway serves as the AWS endpoint and can be associated with route tables.
It is designed with two endpoints to enhance availability and redundancy. The Customer Gateway represents the on-premises side of the connection and can be either a physical networking device in the local data center or a logical representation of such a device in AWS.

The VPN Connection is the link between these two gateways, enabling secure data transmission.
AWS automatically provisions two redundant VPN tunnels between the Virtual Private Gateway and the Customer Gateway device, ensuring failover protection.
Each Customer Gateway setup includes a pair of tunnels for high availability.

There are two types of Site-to-Site VPNs: Static and Dynamic.
A Static VPN requires manual configuration of routing information, specifying which networks exist on either side of the connection.
In contrast, a Dynamic VPN leverages the Border Gateway Protocol (BGP), allowing automatic exchange of routing information between AWS and the on-premises network.


## Accelerated Site-to-Site VPN

Accelerated Site-to-Site VPN is an AWS feature that enhances VPN performance by integrating it with AWS Global Accelerator. 
Traditional Site-to-Site VPN connections route traffic over the public internet to the AWS VPN endpoints (Virtual Private Gateway or Transit Gateway), which can lead to unpredictable latency and throughput due to internet path variability. 
Accelerated VPN improves this by using AWS Global Accelerator to route VPN traffic over the AWS global backbone.

When enabled, VPN traffic from your on-premises network is directed to the nearest AWS edge location where Global Accelerator is present. 
From there, traffic is carried across the AWS global network to reach the VPN endpoint in the target AWS Region. 
This reduces latency, increases throughput, and improves overall reliability.


## Transit Gateway

AWS Transit Gateway is a regional network transit hub that enables scalable and centralized routing between Amazon VPCs, on-premises networks (connected via Site-to-Site VPN or AWS Direct Connect), and other AWS Transit Gateways through peering.

To use a Transit Gateway, resources must be attached to it.
Supported attachment types include:

- VPC Attachments – Connect VPCs by associating specific subnets with the Transit Gateway.
  This ensures that only instances within the attached subnets can communicate through the Transit Gateway.


- Site-to-Site VPN Attachments – Establish encrypted connectivity between an on-premises network and AWS.


- Direct Connect Gateway Attachments – Facilitate private, high-bandwidth connections between AWS and on-premises environments using Direct Connect.

Transit Gateways support peering across AWS accounts and across regions, allowing organizations to build global network architectures with high availability and scalability.
When peered, Transit Gateways in different regions communicate privately, eliminating the need to route traffic over the public internet.

Routing within the Transit Gateway is controlled through route tables.
A default route table is automatically created when the Transit Gateway is set up, defining how traffic flows between attachments.
You can add new route tables.

When Transit Gateways are peered, they do not automatically propagate routes between themselves.
Instead, routes must be manually configured in the route tables of each Transit Gateway.
This means that if two Transit Gateways in different regions are peered, you need to create static routes in each respective Transit Gateway route table to specify how traffic should be forwarded between them.
Without these static entries, communication will not happen across peered gateways.

A single Transit Gateway supports a maximum of 50 peering attachments.

Each attachment (VPC, VPN, or Direct Connect) is associated with one route table.
However, a single route table can be associated with multiple attachments.
This means that while an attachment can only reference one route table at a time, a route table itself can define routing for multiple attachments, providing centralized routing control.

Attachments have the ability to propagate dynamically learned routes to route tables they are not directly associated with.
This is useful when managing complex networks because a route learned from one attachment can be shared across multiple route tables, facilitating automatic route distribution without requiring manual updates.

Transit Gateway route tables are used when traffic is leaving an attachment.
The route table associated with an attachment determines the next-hop destination for outbound traffic.
This ensures that packets are correctly forwarded to the appropriate VPC, on-premises network, or another AWS region via a peered Transit Gateway.

----

## AWS Private Link

AWS PrivateLink facilitates private, secure connectivity between VPCs and AWS or third-party services over the AWS backbone, eliminating exposure to the public internet. 
It achieves this through Interface VPC Endpoints, which utilize Elastic Network Interfaces (ENIs) placed within the consumer’s VPC subnets. 
Communication occurs entirely through private IP addresses, allowing the consumer to access services as if they were natively hosted inside their own VPC. 


## VPC Endpoints

### Gateway

Gateway VPC Endpoints provide secure, private connectivity from a VPC directly to supported AWS services (specifically Amazon S3 and DynamoDB) without traversing the public internet. 
Unlike Interface endpoints (which use ENIs), Gateway endpoints operate by injecting an AWS-managed, regionally resilient route into the VPC's route tables (on specified subnet route tables). 
This route directs traffic intended for the specified AWS service through the endpoint and onto AWS's internal backbone network. 
As a result, instances within the VPC communicate with these services privately, enhancing security by avoiding exposure to public networks.


### Interface

Interface VPC Endpoints enable private, secure connectivity from your VPC to AWS services and third-party services without traversing the public internet. 
They operate by placing Elastic Network Interfaces (ENIs) directly into designated subnets within your VPC. 
Each ENI is assigned private IP addresses from the subnet's CIDR range, allowing resources in the VPC to communicate privately and securely, as though the services reside within your own network.
