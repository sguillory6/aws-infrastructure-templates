# SWA Infrastructure Templates

This repository contains the SWA sanctioned CloudFormation templates that can be used to create the infrastructure resources
needed to support application development in AWS accounts.

## Template Variable Naming Convention

The variables used in the templates are named with the following convention:

* Variables that start with a `m` are mappings
* Variables that start with a `c` are conditions
* Variables that start with a `r` are resources
* Variables that do not start with the above are parameters or outputs

## VPC

The `vpc.yml` template can be used to create a VPC.

The VPC template will generate the following resources:

* VPC
* Public subnets in up to 3 AZs (availability zones) - Used for access to services from public internet and host the NAT Gateway
  for servers in private subnets to access internet.
* Private subnets in up to 3 AZs - Used for services that will only be accessed from SWA or public subnets.
* Data subnets in up to 3 AZs - Used for running data services such as RDS or Gemfire. Should only be accessible from private subnets.
* Route tables and default entries for each subnet created
* NACLs
* Default SSH security group
* Internet Gateway (IGW) if any public subnet is created
* NAT Gateways for each AZ that contains a private subnet if the public subnet exists (Internet Gateway required for NAT)
* Elastic IPs - one for each NAT Gateway in each of the 3 AZs
* VPN Gateway (VPG) to connect to Direct Connect if requested

### Creating VPC Stack

The following is an example of how to use the CloudFormation template to generate a VPC:

```bash

### This call will create a VPC with all 3 subnet tiers in all 3 availability zones.

aws cloudformation deploy \
  --stack-name my-stack-name \
  --template-file vpc.yml \
  --s3-bucket my-bucket \
  --s3-prefix vpc-template \
  --parameter-overrides \
    'VpcCidrBlock=100.65.65.0/24' \
    'AppName=my-app' \
    'EnvName=dev' \
    'SWACostCenter=12345' \
    'SWAPID=IT-00000' \
    'SWAConfidentiality=SWA Confidential' \
    'SWACompliance=PII' \
    'SWABusinessService=CheckIn' \
    'ConfigureDirectConnect=false' \
    'PublicSubAz1=100.65.65.0/28' \
    'PrivateSubAz1=100.65.65.16/28' \
    'DataSubAz1=100.65.65.32/28' \
    'PublicSubAz2=100.65.65.64/28' \
    'PrivateSubAz2=100.65.65.80/28' \
    'DataSubAz2=100.65.65.96/28' \
    'PublicSubAz3=100.65.65.144/28' \
    'PrivateSubAz3=100.65.65.160/28' \
    'DataSubAz3=100.65.65.176/28' \
  --tags \
    'SWA:Name=my-stack-name' \
    'SWA:CostCenter=12345' \
    'SWA:PID=IT-00000' \
    'SWA:Confidentiality=SWA Confidential' \
    'SWA:Compliance=PII' \
    'SWA:BusinessService=CheckIn' \
    'SWA:Environment=dev'
```
You will need to replace the following parameters with valid values:

* `stack-name` - The name you wish to call your vpc stack
* `s3-bucket` - The name of a valid S3 bucket that can be used to upload the vpc template in order to create the stack
* `parameter-overrides` - see **VPC Parameters** section below
* `tags`
  * `SWA:Name` - Use the same value as the `stack-name`
  * `SWA:CostCenter` - The cost center for this application (5 digits)
  * `SWA:PID` - The PID for this application (IT-xxxxx... or IO-xxxxx...)
  * `SWA:Confidentiality` - Confidentiality of application data - SWA Public, SWA Internal or SWA Confidential
  * `SWA:Compliance` - Compliance required for application - PCI, PII or NA
  * `SWA:BusinessService` - Business service of application - Booking, CheckIn, Manage irregular operations, etc...
  * `SWA:Environment` - The environment - **lab**, **dev**, **qa** or **prod**

### VPC Parameters

In order to create a VPC that has associated resources such as subnets, additional parameters will need to be passed to the template.

At a minimum, the private subnets in availability zones 1, 2, and 3 are required to launch a VPC. The public and data subnets are not required, so if a CIDR block
is not passed as a parameter for those subnets, they will not be created.

In order for instances in the private subnets to be able to communicate to the internet (for example to install/update applications on the operating system),
a public subnet must exist for each private subnet in each AZ.

**NOTE: If you plan on connecting this VPC to Direct Connect, you must make sure you have been given a valid CIDR block
to be used by the VPC. If you use a CIDR block that is already being used within SWA or AWS, you WILL create network outages.**

| Parameter | Description | Format | Required | Default value |
|-----------|-------------|--------|----------|---------------|
| VpcCidrBlock | VPC CIDR Block | Valid CIDR x.x.x.x/x | **true** | N/A |
| AppName | Name of the Application | String with no white space e.g. my-app | **true** | N/A |
| EnvName | Environment Name | **lab**, **dev**, **qa** or **prod** | **true** | N/A |
| SWACostCenter | Cost Center Number (5 digits) | XXXXXX | **true** | N/A |
| SWAPID | SWA PID | IT-XXXXX... or IO-XXXXX... (minimum 5 digits) | **true** | N/A |
| SWAConfidentiality | Confidentiality of application data | SWA Public, SWA Internal or SWA Confidential | **true** | N/A |
| SWACompliance | Compliance required for application | PCI, PII or NA | **true** | N/A |
| SWABusinessService | Business service of application | Booking, CheckIn, Manage irregular operations, etc... | **true** | N/A |
| PrivateSubAz1 | Private Tier Subnet Cidr Block - AZ1 | Valid CIDR x.x.x.x/x | **true** | N/A |
| PrivateSubAz2 | Private Tier Subnet Cidr Block - AZ2 | Valid CIDR x.x.x.x/x | **true** | N/A |
| PrivateSubAz3 | Private Tier Subnet Cidr Block - AZ3 | Valid CIDR x.x.x.x/x | **true** | N/A |
| PublicSubAz1 | Public Tier Subnet Cidr Block - AZ1 | Valid CIDR x.x.x.x/x.<br>If no value passed, subnet will not be created. | false | Empty String |
| PublicSubAz2 | Public Tier Subnet Cidr Block - AZ2 | Valid CIDR x.x.x.x/x<br>If no value passed, subnet will not be created. | false | Empty String |
| PublicSubAz3 | Public Tier Subnet Cidr Block - AZ3 | Valid CIDR x.x.x.x/x<br>If no value passed, subnet will not be created. | false | Empty String |
| DataSubAz1 | Data Tier Subnet Cidr Block - AZ1 | Valid CIDR x.x.x.x/x<br>If no value passed, subnet will not be created. | false | Empty String |
| DataSubAz2 | Data Tier Subnet Cidr Block - AZ2 | Valid CIDR x.x.x.x/x<br>If no value passed, subnet will not be created. | false | Empty String |
| DataSubAz3 | Data Tier Subnet Cidr Block - AZ3 | Valid CIDR x.x.x.x/x<br>If no value passed, subnet will not be created. | false | Empty String |
| ConfigureDirectConnect | Configure resources to use Direct Connect | **true** or **false**<br><br>If lab environment, DX will never be created | false | false |
| CsrPreferredPath | The preferred network path between VPC and SWA | **CSR1** or **CSR2** | false | CSR1 |
| InitiateSpoke | Create connection to direct connect | **true** or **false** | false | true |

### VPC Outputs

After the VPC stack has been generated, you will have the following outputs available for use in other CloudFormation templates.
`vpc-stack-name` indicates that the export key will start with the name that was given to VPC stack.

| Output Key | Export Key | Description |
|------------|------------|-------------|
| VpcId | *\<vpc-stack-name>*-VpcId | The VPC ID of this VPC |
| VpcCidrBlock | *\<vpc-stack-name>*-VpcCidrBlock | VPC CIDR Block |
| PublicSubnet1Id | *\<vpc-stack-name>*-PubSub1Id | The ID of Public Subnet AZ 1 |
| PublicSubnet2Id | *\<vpc-stack-name>*-PubSub2Id | Public Subnet 2 ID |
| PublicSubnet3Id | *\<vpc-stack-name>*-PubSub3Id | Public Subnet 3 ID |
| PublicSubnetsRouteTable | *\<vpc-stack-name>*-PubRTId | Public Subnet Route Table ID |
| PrivateSubnet1Id | *\<vpc-stack-name>*-PrivSub1Id | Private Subnet 1 ID |
| PrivateSubnet2Id | *\<vpc-stack-name>*-PrivSub2Id | Private Subnet 2 ID |
| PrivateSubnet3Id | *\<vpc-stack-name>*-PrivSub3Id | Private Subnet 3 ID |
| PrivateSubnetRouteTableAZ1 | *\<vpc-stack-name>*-PrivRT1Id | Private Subnet Route Table ID of AZ 1 |
| PrivateSubnetRouteTableAZ2 | *\<vpc-stack-name>*-PrivRT2Id | Private Subnet Route Table ID of AZ 2 |
| PrivateSubnetRouteTableAZ3 | *\<vpc-stack-name>*-PrivRT3Id | Private Subnet Route Table ID of AZ 3 |
| DataSubnet1Id | *\<vpc-stack-name>*-DataSub1Id | Data Subnet 1 ID |
| DataSubnet2Id | *\<vpc-stack-name>*-DataSub2Id | Data Subnet 2 ID |
| DataSubnet3Id | *\<vpc-stack-name>*-DataSub3Id | Data Subnet 3 ID |
| DataSubnetRouteTableAZ | *\<vpc-stack-name>*-DataRtId | Data Subnet Route Table ID |

### Deleting VPC Stack

When deleting the VPC stack, if you have Direct Connect enabled, you will need to disconnect existing VPN connections before AWS will allow you to delete the VPC stack. 
To disconnect VPN connections, the spoke tag on the VGW will need to be set to false and you will need to wait for connections to be disabled/disconnected before attempting to delete the stack.

You will also need to delete any resources that were created in the VPC prior to deleting the VPC stack.

## Route53

The `route53.yml` template can be used to create a Route53 Private Hosted Zone associated to a specific VPC.

### Creating the Route53 Stack

The following is an example of how to use the CloudFormation template to generate the Route53 Private Hosted Zone.

You will need to replace the following parameters with valid values:

* `stack-name` - the name you wish to call your VPC stack
* `s3-bucket` - The name of a valid S3 bucket that can be used to upload the VPC template in order to create the stack
* `parameter-overrides` - see **Route53 Parameters** section below
* `tags`
  * `SWA:Name` - Use the same value as the `stack-name`
  * `SWA:CostCenter` - The cost center for this application (5 digits)
  * `SWA:PID` - The PID for this application (IT-xxxxx... or IO-xxxxx...)
  * `SWA:Confidentiality` - Confidentiality of application data - SWA Public, SWA Internal or SWA Confidential
  * `SWA:Compliance` - Compliance required for application - PCI, PII or NA
  * `SWA:BusinessService` - Business service of application - Booking, CheckIn, Manage irregular operations, etc...
  * `SWA:Environment` - The environment - **lab**, **dev**, **qa** or **prod**

```bash
aws cloudformation deploy \
  --stack-name my-stack-name \
  --template-file route53.yml \
  --s3-bucket my-bucket \
  --s3-prefix route53-template \
  --parameter-overrides \
    'HostedZoneName=myapp.swacorp.com' \
    'VpcId=vpc-04d5960409f999d49' \
  --tags \
    'SWA:Name=my-stack-name' \
    'SWA:CostCenter=12345' \
    'SWA:PID=IT-00000' \
    'SWA:Confidentiality=SWA Confidential' \
    'SWA:Compliance=PII' \
    'SWA:BusinessService=CheckIn' \
    'SWA:Environment=dev'
```

### Route53 Parameters

In order to create the Private Hosted Zone these parameters will need to be passed to the template.

| Parameter | Description | Format | Required | Default value |
|-----------|-------------|--------|----------|---------------|
| HostedZoneName | The fully qualified domain name used to create the private hosted zone | ...xxx.swacorp.com | **true** | N/A |
| VpcId | The VPC ID to associate to the private hosted zone| vpc-xxxxxxxxxxxxxxxxx | **true** | N/A |

### Route53 Outputs

After the Route53 stack has been generated, you will have the following outputs available for use in other CloudFormation templates.
`route53-stack-name` indicates that the export key will start with the name that was given to the Route53 stack.

| Output Key | Export Key | Description |
|------------|------------|-------------|
| HostedZoneName | *\<route53-stack-name>*-HostedZoneName | The fully-qualified domain name |
| HostedZoneID | *\<route53-stack-name>*-HostedZoneID | The ID of the Hosted Zone |

### Deleting Route53 Stack

When deleting the Route53 stack, you will need to remove any additional RecordSets that were created in the Private Hosted Zone before AWS will allow the hosted zone to be removed.

## DNS

The `dns.yml` template can be used to create the DNS solution to support DNS between SWA and accounts in AWS.

The DNS template will generate the following resources:

* An Elastic Network Interface (ENI) in each AZ created with static IPs to be used by the unbound servers 
* An Auto Scaling Group (ASG) in each AZ that launches the unbound servers
* Launch configurations in each AZ that are used by the ASGs to launch and configure the unbound servers
* A DHCP Option Set that points to all the ENIs that is associated to the VPC
* Security groups that allow access to the unbound servers for SSH and DNS

### Creating DNS stack

The following is an example of how to use the CloudFormation template to generate the DNS solution.

You will need to replace the following parameters with valid values

* `stack-name` - the name you wish to call your VPC stack
* `s3-bucket` - The name of a valid S3 bucket that can be used to upload the VPC template in order to create the stack
* `parameter-overrides` - see **DNS Parameters** section below
* `tags`
  * `SWA:Name` - Use the same value as the `stack-name`
  * `SWA:CostCenter` - The cost center for this application (5 digits)
  * `SWA:PID` - The PID for this application (IT-xxxxx... or IO-xxxxx...)
  * `SWA:Confidentiality` - Confidentiality of application data - SWA Public, SWA Internal or SWA Confidential
  * `SWA:Compliance` - Compliance required for application - PCI, PII or NA
  * `SWA:BusinessService` - Business service of application - Booking, CheckIn, Manage irregular operations, etc...
  * `SWA:Environment` - The environment - **lab**, **dev**, **qa** or **prod**

```bash
aws cloudformation deploy \
  --stack-name dns-stack-name \
  --template-file dns.yml \
  --s3-bucket swa-devops-898203315548-us-east-1 \
  --s3-prefix vpc-template \
  --capabilities CAPABILITY_IAM \
  --parameter-overrides \
    'VpcId=vpc-04d5960409f999d49' \
    'AppName=my-app' \
    'EnvName=dev' \
    'SWACostCenter=12345' \
    'SWAPID=IT-00000' \
    'SWAConfidentiality=SWA Confidential' \
    'SWACompliance=PII' \
    'SWABusinessService=CheckIn' \
    'HostedZoneName=myapp.swacorp.com' \
    'SubnetIdAz1=subnet-0ec539a639eade280' \
    'SubnetIdAz2=subnet-082973d0b5fe54a77' \
    'SubnetIdAz3=subnet-0a7f218446a9cabfe' \
    'DnsProxyIpAz1=100.65.65.58' \
    'DnsProxyIpAz2=100.65.65.138' \
    'DnsProxyIpAz3=100.65.65.202' \
    'Ec2KeyPairName=shane-test'
  --tags \
    'SWA:Name=my-stack-name' \
    'SWA:CostCenter=12345' \
    'SWA:PID=IT-00000' \
    'SWA:Confidentiality=SWA Confidential' \
    'SWA:Compliance=PII' \
    'SWA:BusinessService=CheckIn' \
    'SWA:Environment=dev'
```

### DNS Parameters

| Parameter | Description | Format | Required | Default value |
|-----------|-------------|--------|----------|---------------|
| VpcId | The VPC ID to of the VPC used to create the DNS resources | vpc-xxxxxxxxxxxxxxxxx | **true** | N/A |
| AppName | Name of the Application | String with no white space e.g. "my-app" | **true** | N/A |
| EnvName | Environment Name | **lab**, **dev**, **qa** or **prod** | **true** | N/A |
| SWACostCenter | Cost Center Number (5 digits) | XXXXX | **true** | N/A |
| SWAPID | SWA PID | IT-XXXXX... or IO-XXXXX... (minimum 5 digits) | **true** | N/A |
| SWAConfidentiality | Confidentiality of application data | SWA Public, SWA Internal or SWA Confidential | **true** | N/A |
| SWACompliance | Compliance required for application | PCI, PII or NA | **true** | N/A |
| SWABusinessService | Business service of application | Booking, CheckIn, Manage irregular operations, etc... | **true** | N/A |
| HostedZoneName | Enter the name of the Route 53 private hosted zone. | ...xxx.swacorp.com | **true** | N/A |
| SubnetIdAz1 | Id of the subnet to launch the DNS instance in AZ 1 | subnet-xxxxxxxxxxxxxxxxx | **true** | N/A |
| SubnetIdAz2 | Id of the subnet to launch the DNS instance in AZ 2 | subnet-xxxxxxxxxxxxxxxxx | **true** | N/A |
| SubnetIdAz3 | Id of the subnet to launch the DNS instance in AZ 3 | subnet-xxxxxxxxxxxxxxxxx | **true** | N/A |
| DnsProxyIpAz1 | Enter a static IP from the AZ 1 subnet to use when launching the DNS proxy instance<br>This must be a valid IP from the `SubnetIdAz1` subnet | xxx.xxx.xxx.xxx | **true** | N/A |
| DnsProxyIpAz2 | Enter a static IP from the AZ 2 subnet to use when launching the DNS proxy instance<br>This must be a valid IP from the `SubnetIdAz2` subnet | xxx.xxx.xxx.xxx | **true** | N/A |
| DnsProxyIpAz3 | Enter a static IP from the AZ 3 subnet to use when launching the DNS proxy instance<br>This must be a valid IP from the `SubnetIdAz3` subnet | xxx.xxx.xxx.xxx | **true** | N/A |
| Ec2KeyPairName | Enter the key-pair name by which you will access the instances created. | Valid EC2 key-pair name | **true** | N/A |
| InstanceType | Instance class used for DNS servers | t2.micro<br>t2.small<br>t2.medium<br>t2.large<br>t2.xlarge<br>m4.large<br>m4.xlarge | false | t2.micro |
| DnsInstanceVolumeSize | Enter the disk volume size (in GB) for the DNS proxy instances. | Natural number | false | 10 |
| OnPremDomain | Enter the name of the on-premises domain. SWA internal domain. | swacorp.com | false | swacorp.com |
| OnPremDns1 | Enter the IP address of a primary on-premises DNS resolver (name server). | xxx.xxx.xxx.xxx | false | 172.31.10.11 |
| OnPremDns2 | Enter the IP address of a secondary on-premises DNS resolver (name server). | xxx.xxx.xxx.xxx | false | 172.31.10.10 |

### DNS Outputs

After the DNS stack has been generated, you will have the following outputs available for use in other CloudFormation templates.
`dns-stack-name` indicates that the export key will start with the name that was given to DNS stack.

| Output Key | Export Key | Description |
|------------|------------|-------------|
| DnsProxyIpAz1 | *\<dns-stack-name>*-DnsProxyIpAz1 | The static IP of the DNS proxy located in AZ 1 |
| DnsProxyIpAz2 | *\<dns-stack-name>*-DnsProxyIpAz2 | The static IP of the DNS proxy located in AZ 2 |
| DnsProxyIpAz3 | *\<dns-stack-name>*-DnsProxyIpAz3 | The static IP of the DNS proxy located in AZ 3 |
