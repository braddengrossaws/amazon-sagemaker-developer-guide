# Give SageMaker Hosted Endpoints Access to Resources in Your Amazon VPC<a name="host-vpc"></a>

## Configure a Model for Amazon VPC Access<a name="host-vpc-configure"></a>

To specify subnets and security groups in your private VPC, use the `VpcConfig` request parameter of the [ `CreateModel`](https://docs.aws.amazon.com/sagemaker/latest/APIReference/API_CreateModel.html) API, or provide this information when you create a model in the SageMaker console\. SageMaker uses this information to create network interfaces and attach them to your model containers\. The network interfaces provide your model containers with a network connection within your VPC that is not connected to the internet\. They also enable your model to connect to resources in your private VPC\.

**Note**  
You must create at least two subnets in different availability zones in your private VPC, even if you have only one hosting instance\.

The following is an example of the `VpcConfig` parameter that you include in your call to `CreateModel`:

```
VpcConfig: {
      "Subnets": [
          "subnet-0123456789abcdef0",
          "subnet-0123456789abcdef1",
          "subnet-0123456789abcdef2"
          ],
      "SecurityGroupIds": [
          "sg-0123456789abcdef0"
          ]
       }
```

## Configure Your Private VPC for SageMaker Hosting<a name="host-vpc-vpc"></a>

When configuring the private VPC for your SageMaker models, use the following guidelines\. For information about setting up a VPC, see [Working with VPCs and Subnets](https://docs.aws.amazon.com/AmazonVPC/latest/UserGuide/working-with-vpcs.html) in the *Amazon VPC User Guide*\.

**Topics**
+ [Ensure That Subnets Have Enough IP Addresses](#host-vpc-ip)
+ [Create an Amazon S3 VPC Endpoint](#host-vpc-s3)
+ [Use a Custom Endpoint Policy to Restrict Access to Amazon S3](#host-vpc-policy)
+ [Add Permissions for Endpoint Access for Containers Running in a VPC to Custom IAM Policies](#host-vpc-endpoints)
+ [Configure Route Tables](#host-vpc-route-table)
+ [Connect to Resources Outside Your VPC](#model-vpc-nat)

### Ensure That Subnets Have Enough IP Addresses<a name="host-vpc-ip"></a>

Your VPC subnets should have at least two private IP addresses for each model instance\. For more information, see [VPC and Subnet Sizing for IPv4](https://docs.aws.amazon.com/AmazonVPC/latest/UserGuide/VPC_Subnets.html#vpc-sizing-ipv4) in the *Amazon VPC User Guide*\.

### Create an Amazon S3 VPC Endpoint<a name="host-vpc-s3"></a>

If you configure your VPC so that model containers don't have access to the internet, they can't connect to the Amazon S3 buckets that contain your data unless you create a VPC endpoint that allows access\. By creating a VPC endpoint, you allow your model containers to access the buckets where you store your data and model artifacts \. We recommend that you also create a custom policy that allows only requests from your private VPC to access to your S3 buckets\. For more information, see [Endpoints for Amazon S3](https://docs.aws.amazon.com/AmazonVPC/latest/UserGuide/vpc-endpoints-s3.html)\.

**To create an Amazon S3 VPC endpoint:**

1. Open the Amazon VPC console at [https://console\.aws\.amazon\.com/vpc/](https://console.aws.amazon.com/vpc/)\.

1. In the navigation pane, choose **Endpoints**, then choose **Create Endpoint**

1. For **Service Name**, choose **com\.amazonaws\.*region*\.s3**, where *region* is the name of the AWS Region where your VPC resides\.

1. For **VPC**, choose the VPC that you want to use for this endpoint\.

1. For **Configure route tables**, choose the route tables for the endpoint to use\. The VPC service automatically adds a route to each route table that you choose that points Amazon S3 traffic to the new endpoint\.

1. For **Policy**, choose **Full Access** to allow full access to the Amazon S3 service by any user or service within the VPC\. To restrict access further, choose **Custom**\. For more information, see [Use a Custom Endpoint Policy to Restrict Access to Amazon S3](#host-vpc-policy)\.

### Use a Custom Endpoint Policy to Restrict Access to Amazon S3<a name="host-vpc-policy"></a>

The default endpoint policy allows full access to Amazon Simple Storage Service \(Amazon S3\) for any user or service in your VPC\. To further restrict access to Amazon S3, create a custom endpoint policy\. For more information, see [Using Endpoint Policies for Amazon S3](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-endpoints-s3.html#vpc-endpoints-policies-s3)\. 

You can also use a bucket policy to restrict access to your S3 buckets to only traffic that comes from your Amazon VPC\. For information, see [Using Amazon S3 Bucket Policies](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-endpoints-s3.html#vpc-endpoints-s3-bucket-policies)\.

#### Restrict Package Installation on the Model Container with a Custom Endpoint Policy<a name="host-vpc-policy-repos"></a>

The default endpoint policy allows users to install packages from the Amazon Linux and Amazon Linux 2 repositories on the model container\. If you don't want users to install packages from those repositories, create a custom endpoint policy that explicitly denies access to the Amazon Linux and Amazon Linux 2 repositories\. The following is an example of a policy that denies access to these repositories:

```
{ 
    "Statement": [ 
      { 
        "Sid": "AmazonLinuxAMIRepositoryAccess",
        "Principal": "*",
        "Action": [ 
            "s3:GetObject" 
        ],
        "Effect": "Deny",
        "Resource": [
            "arn:aws:s3:::packages.*.amazonaws.com/*",
            "arn:aws:s3:::repo.*.amazonaws.com/*"
        ] 
      } 
    ] 
} 

{ 
    "Statement": [ 
        { "Sid": "AmazonLinux2AMIRepositoryAccess",
          "Principal": "*",
          "Action": [ 
              "s3:GetObject" 
              ],
          "Effect": "Deny",
          "Resource": [
              "arn:aws:s3:::amazonlinux.*.amazonaws.com/*" 
              ] 
         } 
    ] 
}
```

### Add Permissions for Endpoint Access for Containers Running in a VPC to Custom IAM Policies<a name="host-vpc-endpoints"></a>

The `SageMakerFullAccess` managed policy includes the permissions that you need to use models configured for Amazon VPC access with an endpoint\. These permissions allow SageMaker to create an elastic network interface and attach it to model containers running in a VPC\. If you use your own IAM policy, you must add the following permissions to that policy to use models configured for VPC access\. 

```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "ec2:DescribeVpcEndpoints",
                "ec2:DescribeDhcpOptions",
                "ec2:DescribeVpcs",
                "ec2:DescribeSubnets",
                "ec2:DescribeSecurityGroups",
                "ec2:DescribeNetworkInterfaces",
                "ec2:DeleteNetworkInterfacePermission",
                "ec2:DeleteNetworkInterface",
                "ec2:CreateNetworkInterfacePermission",
                "ec2:CreateNetworkInterface"
            ],
            "Resource": "*"
        }
    ]
}
```

For more information about the `SageMakerFullAccess` managed policy, see [`AmazonSageMakerFullAccess`](security-iam-awsmanpol.md#security-iam-awsmanpol-AmazonSageMakerFullAccess)\. 

### Configure Route Tables<a name="host-vpc-route-table"></a>

Use default DNS settings for your endpoint route table, so that standard Amazon S3 URLs \(for example, `http://s3-aws-region.amazonaws.com/MyBucket`\) resolve\. If you don't use default DNS settings, ensure that the URLs that you use to specify the locations of the data in your models resolve by configuring the endpoint route tables\. For information about VPC endpoint route tables, see [Routing for Gateway Endpoints](https://docs.aws.amazon.com/AmazonVPC/latest/UserGuide/vpce-gateway.html#vpc-endpoints-routing) in the *Amazon VPC User Guide*\.

### Connect to Resources Outside Your VPC<a name="model-vpc-nat"></a>

If you configure your VPC so that it doesn't have internet access, models that use that VPC do not have access to resources outside your VPC\. If your model needs access to resources outside your VPC, provide access with one of the following options:
+ If your model needs access to an AWS service that supports interface VPC endpoints, create an endpoint to connect to that service\. For a list of services that support interface endpoints, see [VPC Endpoints](https://docs.aws.amazon.com/AmazonVPC/latest/UserGuide/vpc-endpoints.html) in the *Amazon VPC User Guide*\. For information about creating an interface VPC endpoint, see [Interface VPC Endpoints \(AWS PrivateLink\)](https://docs.aws.amazon.com/AmazonVPC/latest/UserGuide/vpce-interface.html) in the *Amazon VPC User Guide*\.
+ If your model needs access to an AWS service that doesn't support interface VPC endpoints or to a resource outside of AWS, create a NAT gateway and configure your security groups to allow outbound connections\. For information about setting up a NAT gateway for your VPC, see [Scenario 2: VPC with Public and Private Subnets \(NAT\)](https://docs.aws.amazon.com/AmazonVPC/latest/UserGuide/VPC_Scenario2.html) in the *Amazon Virtual Private Cloud User Guide*\.