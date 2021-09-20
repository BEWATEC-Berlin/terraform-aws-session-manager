# Terraform AWS Session Manager

A Terraform module to setup [AWS Systems Manager Session Manager](https://docs.aws.amazon.com/systems-manager/latest/userguide/session-manager.html).

This module creates the a SSM document to support encrypted session manager communication and logs.  It also creates a KMS key, S3 bucket, and CloudWatch Log group to store logs.  In addition, for EC2 instances without a public IP address it can create VPC endpoints to enable private session manager communication.  However, the VPC endpoint creation can also be facilitated by other modules such as [this](https://github.com/terraform-aws-modules/terraform-aws-vpc).  Be aware of the [AWS PrivateLink pricing](https://aws.amazon.com/privatelink/pricing/) before deployment.

## Usage

Update version to the latest release here: <https://github.com/bridgecrewio/terraform-aws-session-manager/releases>

Instances with Public IPs do not need VPC endpoints

```terraform
module "ssm" {
  source                    = "bridgecrewio/session-manager/aws"
  version                   = "0.2.0"
  bucket_name               = "my-session-logs"
  access_log_bucket_name    = "my-session-access-logs"
  enable_log_to_s3          = true
  enable_log_to_cloudwatch  = true
}
```

Private instances with VPC endpoints for S3 and CloudWatch logging

```terraform
module "ssm" {
  source                    = "bridgecrewio/session-manager/aws"
  version                   = "0.2.0"
  bucket_name               = "my-session-logs"
  access_log_bucket_name    = "my-session-access-logs"
  vpc_id                    = "vpc-0dc9ef19c0c23aeaa"
  tags                      = {
                                Function = "ssm"
                              }
  enable_log_to_s3          = true
  enable_log_to_cloudwatch  = true
  vpc_endpoints_enabled     = true
}
```

This module does not create any IAM policies for access to session manager.  To do that, look at example policies in the [AWS Documentation](https://docs.aws.amazon.com/systems-manager/latest/userguide/getting-started-restrict-access-quickstart.html)

<!-- BEGINNING OF PRE-COMMIT-TERRAFORM DOCS HOOK -->
## Requirements

| Name | Version |
|------|---------|
| terraform | >= 0.12 |
| aws | >= 1.36.0 |

## Providers

| Name | Version |
|------|---------|
| aws | >= 1.36.0 |

## Inputs

| Name | Description | Type | Default | Required |
|------|-------------|------|---------|:--------:|
| access\_log\_bucket\_name | Name prefix of S3 bucket to store access logs from session logs bucket | `string` | n/a | yes |
| access\_log\_expire\_days | Number of days to wait before deleting access logs | `number` | `30` | no |
| bucket\_name | Name prefix of S3 bucket to store session logs | `string` | n/a | yes |
| cloudwatch\_log\_group\_name | Name of the CloudWatch Log Group for storing SSM Session Logs | `string` | `"/ssm/session-logs"` | no |
| cloudwatch\_logs\_retention | Number of days to retain Session Logs in CloudWatch | `number` | `30` | no |
| enable\_log\_to\_cloudwatch | Enable Session Manager to Log to CloudWatch Logs | `bool` | `true` | no |
| enable\_log\_to\_s3 | Enable Session Manager to Log to S3 | `bool` | `true` | no |
| kms\_key\_alias | Alias prefix of the KMS key.  Must start with alias/ followed by a name | `string` | `"alias/ssm-key"` | no |
| kms\_key\_deletion\_window | Waiting period for scheduled KMS Key deletion.  Can be 7-30 days. | `number` | `7` | no |
| log\_archive\_days | Number of days to wait before archiving to Glacier | `number` | `30` | no |
| log\_expire\_days | Number of days to wait before deleting | `number` | `365` | no |
| region | AWS Region | `string` | n/a | yes |
| tags | A map of tags to add to all resources | `map(string)` | `{}` | no |
| vpc\_endpoints\_enabled | Create VPC Endpoints | `bool` | `false` | no |
| vpc\_id | VPC ID to deploy endpoints into | `string` | `null` | no |

## Outputs

| Name | Description |
|------|-------------|
| access\_log\_bucket\_name | n/a |
| cloudwatch\_log\_group\_arn | n/a |
| iam\_profile\_name | n/a |
| iam\_role\_arn | n/a |
| kms\_key\_arn | n/a |
| session\_logs\_bucket\_name | n/a |
| ssm\_security\_group | n/a |
| vpc\_endpoint\_ec2messages | n/a |
| vpc\_endpoint\_kms | n/a |
| vpc\_endpoint\_logs | n/a |
| vpc\_endpoint\_s3 | n/a |
| vpc\_endpoint\_ssm | n/a |
| vpc\_endpoint\_ssmmessages | n/a |

<!-- END OF PRE-COMMIT-TERRAFORM DOCS HOOK -->

## SSM Usage Example

* Launch an instance using the ssm_profile created by Terraform
* Install the session-manager-plugin and start a session

```bash
cd /tmp
curl "https://s3.amazonaws.com/session-manager-downloads/plugin/latest/mac/sessionmanager-bundle.zip" -o "sessionmanager-bundle.zip"
unzip sessionmanager-bundle.zip
sudo ./sessionmanager-bundle/install -i /usr/local/sessionmanagerplugin -b /usr/local/bin/session-manager-plugin

# Verify
session-manager-plugin

cd -

# Start an SSM session - Note the instance must have a public IP if you have not created VPC endpoints
aws ssm start-session --target <EC2 Instance ID>
```

* Review session logs in your CloudWatch logs group
* Review session logs in your S3 bucket
