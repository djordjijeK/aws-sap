## How DNS Works In AWS

DNS is a hierarchical, distributed system that resolves human-readable domain names into IP addresses. 
AWS Route 53 is a DNS service that provides domain registration, DNS resolution, and health checking.

When a DNS query is made:

- The request is checked in the local DNS resolver. 
- If not found, the resolver queries the root nameservers.
- The root nameservers directs the request to the TLD (Top-Level Domain) nameservers (e.g., `.com`, `.org`, `.io`).
- The TLD nameserver directs it to the authoritative nameserver, which contains the hosted zone and its DNS records.
- The authoritative nameserver responds with the requested IP address or relevant DNS record.


## Hosted Zones: Public vs. Private

A hosted zone is a container for DNS records for a specific domain, managed within Route 53.

- Public Hosted Zone: Used for domains accessible over the internet. The records in this zone are propagated globally.
- Private Hosted Zone: Used for domains within a private VPC. These records resolve only within the associated VPCs and are not accessible over the internet.


## Nameservers

Nameservers are responsible for responding to DNS queries for a domain. 
Each hosted zone in Route 53 is assigned a set of four authoritative nameservers.

- When a domain is registered, the registrar (e.g., AWS, GoDaddy) must be updated with these nameservers.
- When a query is made, the request eventually reaches these nameservers, which resolve it using the records in the hosted zone.


## DNS Record Types

- A Record (Address Record): Maps a domain name to an IPv4 address.
- AAAA Record: Maps a domain name to an IPv6 address.
- CNAME Record: Points a domain alias to another fully qualified domain name (FQDN) instead of an IP.
- TXT (Text) Record: Stores arbitrary text data. Often used for verification of domain.
- MX (Mail Exchange) Record: Specifies the mail servers responsible for receiving emails for a domain.


## Health Checks

Route 53 health checks continuously monitor the health of endpoints and integrated AWS services to determine their availability. 
A health check operates by sending requests to a specified endpoint at configured intervals and evaluating the response against a set of conditions.

A health check monitors either an IP address, a domain name, an AWS service, or a CloudWatch alarm. 
When monitoring an IP or domain, Route 53 sends TCP, HTTP, or HTTPS requests and expects a valid response (code 2xx or 3xx or response body). 
If the response deviates from expectations, the health check is considered failed. 
If a CloudWatch alarm is used, the health check status is determined by the alarm state.
Health checks also support calculated health checks, which aggregate multiple checks using boolean logic to determine overall health.

Health checks support failover routing, where Route 53 removes unhealthy records from DNS resolution, directing traffic only to healthy endpoints. 
If an endpoint recovers, it is reinstated in DNS responses automatically. 

Health check observations come from multiple AWS regions to avoid false positives. 
The failure threshold determines how many consecutive failures trigger an unhealthy state. 


## Routing Policies

- Simple Routing resolves DNS queries to a single record. 
If multiple records exist for the same domain, all are returned, and the client selects one at random. 
No health checks can be applied, making it unsuitable for failover.


- Weighted Routing distributes traffic across multiple resources by assigning relative weights to DNS records. 
A higher weight increases the probability of selection. 
Route 53 responds based on weight proportions. 
Health checks can be applied, ensuring only healthy records are returned. 


- Latency-Based Routing (LBR) directs traffic to the region with the lowest network latency based on AWS measurements between user locations and AWS endpoints. 
If the lowest-latency resource is unavailable, traffic is routed to the next best option. 
Health checks ensure that only healthy endpoints are selected. 


- Failover Routing supports active-passive failover by directing traffic to a primary resource unless it becomes unhealthy. 
A secondary resource serves as a backup and takes over only if the primary endpoint fails health checks.


- Multivalue Answer Routing returns multiple healthy records in response to queries, allowing clients to perform load balancing.
Health checks ensure only available endpoints are returned.


- Geoproximity Routing directs traffic based on the physical distance between the user and AWS resources. 
Traffic flow can be influenced using a bias value, which expands or shrinks the geographic area covered by a resource.


- Each DNS record in Geolocation Routing is linked to a specific location (country, continent, or state).
When a user makes a request, Route 53 checks the user’s location and matches it with the closest configured record.


## DNSSEC 

1. Key Generation  - AWS Route 53 generates Key Signing Keys (KSKs) and Zone Signing Keys (ZSKs) for DNSSEC-enabled hosted zones. 
The ZSK's private key signs all DNS records in the hosted zone, producing RRSIG records. 
The KSK's private key signs only the DNSKEY record, which contains the public ZSK and public KSK. The public KSK is included in the Delegation Signer (DS) record, which is registered at the domain registrar to create a trust link to the parent zone.


2. DNS Response with RRSIG - When a resolver queries a DNSSEC-enabled domain, Route 53 returns:
   - The requested DNS record (e.g., A, CNAME, MX).
   - An RRSIG record, which is a cryptographic signature proving the authenticity of the DNS record.
   - A DNSKEY record, which contains the public ZSK and KSK necessary for signature verification.


3. Signature Verification Process - The resolver follows a chain of trust to verify authenticity:
   - It retrieves the DS record from the parent zone, which contains a hash of the domain's public KSK.
   - It then uses this public KSK to validate the DNSKEY record returned by the authoritative DNS server.
   - If the DNSKEY record is valid, the resolver extracts the public ZSK and uses it to verify the RRSIG signature for the requested DNS record (A, CNAME, etc.).
   - If all signatures match and the cryptographic chain is intact, the response is trusted. Otherwise, the response is rejected.


## Route53 Endpoints

Route 53 Resolver Endpoints provide DNS resolution between your VPC and your on-premises networks by enabling a seamless, hybrid DNS environment. 
They operate by forwarding DNS queries across network boundaries, specifically between AWS VPCs and external (on-premises) DNS infrastructures.

Inbound Resolver Endpoints allow DNS queries originating from your on-premises network to reach AWS Route 53 Resolver. 
When your local infrastructure sends a DNS request for AWS-hosted resources, the inbound endpoint accepts the request, resolves it using Route 53 private hosted zones or public DNS, and returns the result.

Outbound Resolver Endpoints send DNS queries originating from your AWS VPC to your on-premises DNS servers. 
When an EC2 instance or service inside your VPC queries a domain managed by your on-premises DNS, the Route 53 Resolver forwards the query through the outbound endpoint to your external DNS servers for resolution.

Route 53 Resolver Endpoints solve the critical challenge of hybrid DNS resolution, enabling consistent DNS lookups across AWS and on-premises resources.