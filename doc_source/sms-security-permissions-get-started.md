# Use IAM Managed Policies with Ground Truth<a name="sms-security-permissions-get-started"></a>

SageMaker and Ground Truth provide AWS managed policies that you can use to create a labeling job\. If you are getting started using Ground Truth and you do not require granular permissions for your use case, it is recommended that you use the following policies:
+ `[AmazonSageMakerFullAccess](https://console.aws.amazon.com/iam/home?#/policies/arn:aws:iam::aws:policy/AmazonSageMakerFullAccess)` – Use this policy to give an IAM user or role permission to create a labeling job\. This is a broad policy that grants an IAM entity permission to use SageMaker features, as well as features of necessary AWS services through the console and API\. This policy gives the IAM entity permission to create a labeling job and to create and manage workforces using Amazon Cognito\. To learn more, see [AmazonSageMakerFullAccess Policy](https://docs.aws.amazon.com/sagemaker/latest/dg/security-iam-awsmanpol-AmazonSageMakerFullAccess)\.
+ `[AmazonSageMakerGroundTruthExecution](https://console.aws.amazon.com/iam/home?#/policies/arn:aws:iam::aws:policy/AmazonSageMakerGroundTruthExecution)` – To create an *execution role*, you can attach the policy `[AmazonSageMakerGroundTruthExecution](https://console.aws.amazon.com/iam/home?#/policies/arn:aws:iam::aws:policy/AmazonSageMakerGroundTruthExecution)` to an IAM role\. An execution role is the role that you specify when you create a labeling job and it is used to start your labeling job\. This policy allows you to create both streaming and non\-streaming labeling jobs, and to create a labeling job using any task type\. Note the following limits of this managed policy\.
  + **Amazon S3 permissions**: This policy grants an execution role permission to access Amazon S3 buckets with the following strings in the name: `GroundTruth`, `Groundtruth`, `groundtruth`, `SageMaker`, `Sagemaker`, and `sagemaker` or a bucket with an [object tag](https://docs.aws.amazon.com/AmazonS3/latest/userguide/object-tagging.html) that includes `SageMaker` in the name \(case insensitive\)\. Make sure your input and output bucket names include these strings, or add additional permissions to your execution role to [grant it permission to access your Amazon S3 buckets](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_examples_s3_rw-bucket.html)\. You must give this role permission to perform the following actions on your Amazon S3 buckets: `AbortMultipartUpload`, `GetObject`, and `PutObject`\.
  + **Custom Workflows**: When you create a [custom labeling workflow](https://docs.aws.amazon.com/sagemaker/latest/dg/sms-custom-templates.html), this execution role is restricted to invoking AWS Lambda functions with one of the following strings as part of the function name: `GtRecipe`, `SageMaker`, `Sagemaker`, `sagemaker`, or `LabelingFunction`\. This applies to both your pre\-annotation and post\-annotation Lambda functions\. If you choose to use names without those strings, you must explicitly provide `lambda:InvokeFunction` permission to the execution role used to create the labeling job\.

To learn how to attach an AWS managed policy to an IAM user or role \(identity\), refer to [Adding and removing IAM identity permissions](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_manage-attach-detach.html#add-policies-console) in the IAM User Guide\.