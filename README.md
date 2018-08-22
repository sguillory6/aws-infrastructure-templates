SWA Infrastructure templates
============================

This repository contains the SWA sanctioned clouformation templates that can be used to create a VPC and all resources
needed to support the VPC accross three availability zones (AZs).

VPC
---

The vpc.yml template can be used to create a VPC. The VPC template will generate the following resources:
* VPC
* Public subnets in up to 3 AZs - Used for access to services from public internet and host the NAT Gateway
  for servers in public and management subnets to access internet.
* Private subnets in up to 3 AZs - User for services that will only be accessed from SWA or public subnets.
* Data subnets in up to 3 AZs - Used for running data services such as RDS or Gemfire. Should only be accessible from private subnets.
* Management Subnets in up to 3 AZs - Used for management of other subnets such as bastion hosts.
* Route tables and default entries for each subnet created
* NACLs
* Default SSH security group

**Template naming convention**

The variables used in the vpc template are named with the following convention

* Variables that start with a `c` are conditions
* Variables that start with a `p` are parameters
* Variables that start with a `r` are resources
* Variables that do not start with the above are outputs


#### Creating VPC stack
The following is an example of how to use the cloudformation template to generate a VPC. This call
will create a VPC with all 4 subnet tiers in all 3 availability zones.

You will need to replace the following paramters with valid values
* `stack-name` - the name you wish to call your vpc stack
* `s3-bucket` - The name of a valid S3 bucket that can be used to upload the vpc template in order to create the stack
* `parameter-overrides` - see parameters section below
* `tags`
  * `SWA:Name` - Use the same value as the `stack-name`
  * `SWA:CostCenter` - The cost center for this application
  * `SWA:PID` - The PID for this application
  * `SWA:Confidentiality` - Confidentiality of application data - SWA Public, SWA Internal or SWA Confidential
  * `SWA:Compliance` - Compliance required for application - PCI, PII or NA
  * `SWA:BusinessService` - Business service of application - Booking, CheckIn, Manage irregular operations, etc...
  * `SWA:Environment` - The environment - **lab**, **dev**, **qa** or **prod**


```
aws cloudformation deploy \
  --stack-name vpc-stack-name \
  --template-file vpc.yml \
  --s3-bucket my-bucket \
  --s3-prefix vpc-template \
  --parameter-overrides \
    'pVpcCidrBlock=100.65.65.0/24' \
    'pAppName=my-app' \
    'pEnvName=dev' \
    'pCostCenter=123456' \
    'pPID=IT-00000' \
    'pConfidentiality=SWA Confidential' \
    'pCompliance=PII' \
    'pBusinessService=CheckIn' \
    'pConfigureDirectConnect=false' \
    'pPublSubAz1=100.65.65.0/28' \
    'pPrivSubAz1=100.65.65.16/28' \
    'pDataSubAz1=100.65.65.32/28' \
    'pMgmtSubAz1=100.65.65.48/28' \
    'pPublSubAz2=100.65.65.64/28' \
    'pPrivSubAz2=100.65.65.80/28' \
    'pDataSubAz2=100.65.65.96/28' \
    'pMgmtSubAz2=100.65.65.128/28' \
    'pPublSubAz3=100.65.65.144/28' \
    'pPrivSubAz3=100.65.65.160/28' \
    'pDataSubAz3=100.65.65.176/28' \
    'pMgmtSubAz3=100.65.65.192/28' \
  --tags \
    'SWA:Name=vpc-stack-name' \
    'SWA:CostCenter=123456' \
    'SWA:PID=IT-00000' \
    'SWA:Confidentiality=SWA Confidential' \
    'SWA:Compliance=PII' \
    'SWA:BusinessService=CheckIn' \
    'SWA:Environment=dev'
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
| pInitiateSpoke | Create connection to direct connect | **true** or **false** | false | true |

#### Outputs

After the VPC stack has been generated, you will have the following outputs avaialble for use in other cloufrormation templates. `vpc-stack-name` indicates that the export key will start with the name that was given to vpc stack.

|Output Key | Export Key | Description|
|-----------|------------|------------|
|VpcId | *\<vpc-stack-name>*-VpcId | The VPC ID of this VPC|
|VpcCidrBlock | *\<vpc-stack-name>*-VpcCidrBlock | VPC CIDR Block|
|PublicSubnet1Id | *\<vpc-stack-name>*-PubSub1Id | The ID of Public Subnet AZ 1|
|PublicSubnet2Id | *\<vpc-stack-name>*-PubSub2Id | Public Subnet 2 ID|
|PublicSubnet3Id | *\<vpc-stack-name>*-PubSub3Id | Public Subnet 3 ID|
|PublicSubnetsRouteTable | *\<vpc-stack-name>*-PubRTId | Public Subnet Route Table ID|
|PrivateSubnet1Id | *\<vpc-stack-name>*-PrivSub1Id | Private Subnet 1 ID|
|PrivateSubnet2Id | *\<vpc-stack-name>*-PrivSub2Id | Private Subnet 2 ID|
|PrivateSubnet3Id | *\<vpc-stack-name>*-PrivSub3Id | Private Subnet 3 ID|
|PrivateSubnetRouteTableAZ1 | *\<vpc-stack-name>*-PrivRT1Id | Private Subnet Route Table ID of AZ 1|
|PrivateSubnetRouteTableAZ2 | *\<vpc-stack-name>*-PrivRT2Id | Private Subnet Route Table ID of AZ 2|
|PrivateSubnetRouteTableAZ3 | *\<vpc-stack-name>*-PrivRT3Id | Private Subnet Route Table ID of AZ 3|
|ManagementSubnet1Id | *\<vpc-stack-name>*-MgmtSub1Id | Management Subnet 1 ID|
|ManagementSubnet2Id | *\<vpc-stack-name>*-MgmtSub2Id | Management Subnet 2 ID|
|ManagementSubnet3Id | *\<vpc-stack-name>*-MgmtSub3Id | Management Subnet 3 ID|
|ManagementSubnetRouteTableAZ1 | *\<vpc-stack-name>*-MgmtRT1Id | Management Subnet Route Table ID of AZ 1|
|ManagementSubnetRouteTableAZ2 | *\<vpc-stack-name>*-MgmtRT2Id | Management Subnet Route Table ID of AZ 2|
|ManagementSubnetRouteTableAZ3 | *\<vpc-stack-name>*-MgmtRT3Id | Management Subnet Route Table ID of AZ 3|
|DataSubnet1Id | *\<vpc-stack-name>*-DataSub1Id | Data Subnet 1 ID|
|DataSubnet2Id | *\<vpc-stack-name>*-DataSub2Id | Data Subnet 2 ID|
|DataSubnet3Id | *\<vpc-stack-name>*-DataSub3Id | Data Subnet 3 ID|
|DataSubnetRouteTableAZ | *\<vpc-stack-name>*-DataRtId | Data Subnet Route Table ID|
