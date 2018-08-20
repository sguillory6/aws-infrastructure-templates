SWA Infrastructure templates
============================

This repository contains the SWA sanctioned clouformation templates that can be used to create a VPC and all resources
needed to support the VPC accross three availability zones (AZs).

VPC
---

The vpc.yml template can be used to create a VPC. The VPC template will generate the following resources:
- VPC
- Public subnets in up to 3 AZs - Used for access to services from public internet and host the NAT Gateway
  for servers in public and management subnets to access internet.
- Private subnets in up to 3 AZs - User for services that will only be accessed from SWA or public subnets.
- Data subnets in up to 3 AZs - Used for running data services such as RDS or Gemfire. Should only be accessible from private subnets.
- Management Subnets in up to 3 AZs - Used for management of other subnets such as bastion hosts.
- Route tables and default entries for each subnet created
- NACLs
- Default SSH security group

#### Creating VPC stack
The following is an example of how to use the cloudformation template to generate a VPC. This call
will create a VPC with all 4 subnet tiers in all 3 availability zones.

```
aws cloudformation deploy \
  --stack-name vpc-stack-name \
  --template-file vpc.yml \
  --s3-bucket my-bucket \
  --s3-prefix vpc-template \
  --parameter-overrides \
    'pVpcCidrBlock=10.65.65.0/24' \
    'pAppName=my-app' \
    'pEnvName=dev' \
    'pCostCenter=123456' \
    'pPID=IT-00000' \
    'pConfidentiality=SWA Confidential' \
    'pCompliance=PII' \
    'pBusinessService=CheckIn' \
    'pConfigureDirectConnect=false' \
    'pPublSubAz1=10.65.65.0/28' \
    'pPrivSubAz1=10.65.65.16/28' \
    'pDataSubAz1=10.65.65.32/28' \
    'pMgmtSubAz1=10.65.65.48/28' \
    'pPublSubAz2=10.65.65.64/28' \
    'pPrivSubAz2=10.65.65.80/28' \
    'pDataSubAz2=10.65.65.96/28' \
    'pMgmtSubAz2=10.65.65.128/28' \
    'pPublSubAz3=10.65.65.144/28' \
    'pPrivSubAz3=10.65.65.160/28' \
    'pDataSubAz3=10.65.65.176/28' \
    'pMgmtSubAz3=10.65.65.192/28'

```

#### Template parameters
In order to create a VPC that has associated resources such as subnets, additional parameters will need to be passed to the template.

**NOTE: If you plan on connecting this VPC to Direct Connect, you must make sure you have been given a valid CIDR block
to be used by the VPC. If you use a CIDR block that is already being used within SWA or AWS, you WILL create network outages.**

| Parameter | Description | Format | Required | Default value |
|-----------|-------------|--------|----------|---------------|
| pVpcCidrBlock | VPC CIDR Block | Valid CIDR x.x.x.x/x | **true** | N/A |
| pAppName | Name of the Application | String with no white space | **true** | N/A |
| pEnvName | Environment Name | **lab**, **dev**, **qa** or **prod** | **true** | N/A |
| pCostCenter | Cost Center Number | XXXXXX | **true** | N/A |
| pPID | SWA PID | IT-XXXXX or IO-XXXXX | **true** | N/A |
| pConfidentiality | Confidentiality of application data | SWA Public, SWA Internal or SWA Confidential | **true** | N/A |
| pCompliance | Compliance required for application | PCI, PII or NA | **true** | N/A |
| pBusinessService | Business service of application | Booking, CheckIn, Manage irregular operations, etc... | **true** | N/A |
| pPublSubAz1 | Public Tier Subnet Cidr Block - AZ1 | Valid CIDR x.x.x.x/x.<br>If no value passed, subnet will not be created. | false | Empty String |
| pPublSubAz2 | Public Tier Subnet Cidr Block - AZ2 | Valid CIDR x.x.x.x/x<br>If no value passed, subnet will not be created. | false | Empty String |
| pPublSubAz3 | Public Tier Subnet Cidr Block - AZ3 | Valid CIDR x.x.x.x/x<br>If no value passed, subnet will not be created. | false | Empty String |
| pPrivSubAz1 | Private Tier Subnet Cidr Block - AZ1 | Valid CIDR x.x.x.x/x<br>If no value passed, subnet will not be created. | false | Empty String |
| pPrivSubAz2 | Private Tier Subnet Cidr Block - AZ2 | Valid CIDR x.x.x.x/x<br>If no value passed, subnet will not be created. | false | Empty String |
| pPrivSubAz3 | Private Tier Subnet Cidr Block - AZ3 | Valid CIDR x.x.x.x/x<br>If no value passed, subnet will not be created. | false | Empty String |
| pDataSubAz1 | Data Tier Subnet Cidr Block - AZ1 | Valid CIDR x.x.x.x/x<br>If no value passed, subnet will not be created. | false | Empty String |
| pDataSubAz2 | Data Tier Subnet Cidr Block - AZ2 | Valid CIDR x.x.x.x/x<br>If no value passed, subnet will not be created. | false | Empty String |
| pDataSubAz3 | Data Tier Subnet Cidr Block - AZ3 | Valid CIDR x.x.x.x/x<br>If no value passed, subnet will not be created. | false | Empty String |
| pMgmtSubAz1 | Management Tier Subnet Cidr Block - AZ1 | Valid CIDR x.x.x.x/x<br>If no value passed, subnet will not be created. | false | Empty String |
| pMgmtSubAz2 | Management Tier Subnet Cidr Block - AZ2 | Valid CIDR x.x.x.x/x<br>If no value passed, subnet will not be created. | false | Empty String |
| pMgmtSubAz3 | Management Tier Subnet Cidr Block - AZ3 | Valid CIDR x.x.x.x/x<br>If no value passed, subnet will not be created. | false | Empty String |
| pConfigureDirectConnect | Configure resources to use Direct Connect | **true** or **false**<br><br>If lab environment, DX will never be created | false | false |
| pCsrPreferredPath | The preferred network path between VPC and SWA | **CSR1** or **CSR2** | false | CSR1 |
| pInitiateSpoke | Create connection to direct connect | **true** or **false** | false | false |

