### Public vs Private Services

- Public services in AWS are openly accessible (with proper authentication and authorization) over the internet by design (e.g. S3, DynamoDB, SNS, SQS, etc.), requiring no Virtual Private Cloud (VPC) configuration.
- Private services exist within a VPC (e.g. EC2, RDS, etc.) and require specific network configurations to be accessed.
- The core difference is accessibility - public services are publicly reachable, while private services remain isolated unless explicitly exposed.


### DHCP

- DHCP (Dynamic Host Configuration Protocol) in AWS is responsible for automatically assigning network configuration parameters to instances within a Virtual Private Cloud (VPC).
  It enables instances to receive critical network details, including IP addresses, subnet masks, default gateways, DNS servers, and domain names.
  DHCP functions through an L2 broadcast mechanism, meaning the requesting instance sends a broadcast query to locate an available DHCP server, which then responds with the necessary configuration.


- In AWS, DHCP is managed through DHCP Option Sets, which define the network configuration details that instances within a VPC receive.
  Each VPC can be associated with only one DHCP option set at a time, but a single option set can be linked to multiple VPCs.
  These option sets allow customization of DNS servers, domain names, NTP servers, etc.
  AWS provides default DNS resolution through AmazonProvidedDNS, but if custom domain names are needed, a custom DNS server must be used.


- One crucial aspect to remember is that associating a new DHCP option set is immediate, but changes require a DHCP lease renewal, which takes time.
  Additionally, DHCP option sets in AWS are immutable, meaning they cannot be edited once created - any modifications require the creation of a new set and re-association with the VPC.
  Instances relying on DHCP will resolve public and private domain names, and AWS supplies both, but using a custom domain requires explicit configuration of DNS servers.


### VPC Router

- The VPC Router in AWS is an internal component responsible for directing traffic between subnets, external networks, and AWS services within a Virtual Private Cloud (VPC).
  Every subnet has a VPC Router Interface, which uses the subnet + 1 address (for example, if a subnet is 10.16.32.0/24, its VPC router address will be 10.16.32.1).


- Routing in a VPC is controlled by Route Tables, which determine how traffic is forwarded.
  Each VPC is automatically assigned a Main Route Table, which acts as the default for all subnets unless explicitly overridden.
  Custom route tables can be created and assigned to specific subnets, replacing the default Main Route Table.
  However, a subnet can only be associated with one route table at a time.


- Route tables contain routes, and AWS follows a most-specific route first principle, meaning traffic is directed based on the most detailed match.
  Routes define where traffic is sent, whether within the VPC, to on-premises data centers via VPN or Direct Connect, or to the public internet via an Internet Gateway.
