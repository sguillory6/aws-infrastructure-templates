# My Infrastructure Templates

This repository contains the My sanctioned CloudFormation templates that can be used to create the infrastructure resources
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
* Private subnets in up to 3 AZs - Used for services that will only be accessed from My or public subnets.
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
  --stack-name my-app-dev-us-east-1-vpc-stack \
  --template-file vpc.yml \
  --s3-bucket my-bucket \
  --s3-prefix vpc-template \
  --parameter-overrides \
    'VpcCidrBlock=100.65.65.0/24' \
    'AppName=my-app' \
    'EnvName=dev' \
    'PublicSubAz1=100.65.65.0/28' \
    'PrivateSubAz1=100.65.65.16/28' \
    'DataSubAz1=100.65.65.32/28' \
    'PublicSubAz2=100.65.65.64/28' \
    'PrivateSubAz2=100.65.65.80/28' \
    'DataSubAz2=100.65.65.96/28' \
    'PublicSubAz3=100.65.65.144/28' \
    'PrivateSubAz3=100.65.65.160/28' \
    'DataSubAz3=100.65.65.176/28' \
```

You will need to replace the following parameters with valid values:

* `stack-name` - *\<app-name>*-*\<env>*-*\<region>*-vpc-stack
* `s3-bucket` - The name of a valid S3 bucket that can be used to upload the vpc template in order to create the stack
* `parameter-overrides` - see **VPC Parameters** section below

### VPC Parameters

In order to create a VPC that has associated resources such as subnets, additional parameters will need to be passed to the template.

At a minimum, the private subnets in availability zones 1, 2, and 3 are required to launch a VPC. The public and data subnets are not required, so if a CIDR block
is not passed as a parameter for those subnets, they will not be created.

In order for instances in the private subnets to be able to communicate to the internet (for example to install/update applications on the operating system),
a public subnet must exist for each private subnet in each AZ.

**WARNINGS!!! VERY IMPORTANT**
* **If connecting this VPC to Direct Connect, do not use the same CIDR block in multiple VPCs or this WILL create network outages.**
* **Do not attempt to use Direct Connect in a lab environment.**

| Parameter | Description | Format | Required | Default value |
|-----------|-------------|--------|----------|---------------|
| VpcCidrBlock | VPC CIDR Block | Valid CIDR x.x.x.x/x | **true** | N/A |
| AppName | Name of the Application | String with no white space e.g. my-app | **true** | N/A |
| EnvName | Environment Name | **lab**, **dev**, **qa** or **prod** | **true** | N/A |
| PrivateSubAz1 | Private Tier Subnet Cidr Block - AZ1 | Valid CIDR x.x.x.x/x | **true** | N/A |
| PrivateSubAz2 | Private Tier Subnet Cidr Block - AZ2 | Valid CIDR x.x.x.x/x | **true** | N/A |
| PrivateSubAz3 | Private Tier Subnet Cidr Block - AZ3 | Valid CIDR x.x.x.x/x | **true** | N/A |
| PublicSubAz1 | Public Tier Subnet Cidr Block - AZ1 | Valid CIDR x.x.x.x/x.<br>If no value passed, subnet will not be created. | false | Empty String |
| PublicSubAz2 | Public Tier Subnet Cidr Block - AZ2 | Valid CIDR x.x.x.x/x<br>If no value passed, subnet will not be created. | false | Empty String |
| PublicSubAz3 | Public Tier Subnet Cidr Block - AZ3 | Valid CIDR x.x.x.x/x<br>If no value passed, subnet will not be created. | false | Empty String |
| DataSubAz1 | Data Tier Subnet Cidr Block - AZ1 | Valid CIDR x.x.x.x/x<br>If no value passed, subnet will not be created. | false | Empty String |
| DataSubAz2 | Data Tier Subnet Cidr Block - AZ2 | Valid CIDR x.x.x.x/x<br>If no value passed, subnet will not be created. | false | Empty String |
| DataSubAz3 | Data Tier Subnet Cidr Block - AZ3 | Valid CIDR x.x.x.x/x<br>If no value passed, subnet will not be created. | false | Empty String |

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

You will also need to delete any resources that were created in the VPC prior to deleting the VPC stack.

## Route53

The `route53.yml` template can be used to create a Route53 Private Hosted Zone associated to a specific VPC.

### Creating the Route53 Stack

The following is an example of how to use the CloudFormation template to generate the Route53 Private Hosted Zone.

```bash
aws cloudformation deploy \
  --stack-name my-app-dev-us-east-1-route53-stack \
  --template-file route53.yml \
  --parameter-overrides \
    'HostedZoneName=myapp.example.com' \
    'VpcId=vpc-04d5960409f999d49' \
```

You will need to replace the following parameters with valid values:

* `stack-name` - *\<app-name>*-*\<env>*-*\<region>*-route53-stack
* `parameter-overrides` - see **Route53 Parameters** section below

### Route53 Parameters

In order to create the Private Hosted Zone these parameters will need to be passed to the template.

| Parameter | Description | Format | Required | Default value |
|-----------|-------------|--------|----------|---------------|
| HostedZoneName | The fully qualified domain name used to create the private hosted zone | ...xxx.example.com | **true** | N/A |
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


