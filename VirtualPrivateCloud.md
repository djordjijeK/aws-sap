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

