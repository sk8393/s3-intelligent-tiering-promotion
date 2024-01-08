# S3 Intelligent-Tiering Promotion

## Summary
This is a reference solution to promote [Amazon Simple Storage Service (S3) Intelligent-Tiering](https://aws.amazon.com/s3/storage-classes/intelligent-tiering/) through Lifecycle rule, in large organization.

## What is the motivation for using S3 Intelligent-Tiering?
S3 Intelligent-Tiering is designed to optimize storage costs by automatically moving objects between multiple access tiers.
Less frequently accessed objects will be moved to colder tier which has smaller per Gigabyte storage cost.

S3 Intelligent-Tiering deserves to be the default storage class when using S3.
Because the default three access tiers, 1/ **Frequent Access Tier**, 2/ **Infrequent Access Tier**, and 3/ **Archive Instant Access Tier**, provide low latency access to the objects, and there is no fee for moving objects between tiers and retrieval of the data.
Just to be sure, there is a monitoring fee for objects bigger than 128 Kilobytes ($0.0025 per 1,000 objects).

If you are considering migration of objects stored in Standard storage class to Intelligent-Tiering, Lifecycle can automatically transition objects from Standard to Intelligent-Tiering by set rule.
User can off-load optimization to AWS by just storing objects as Intelligent-Tiering instead of Standard.

## Challenge for big organization
For a large organizations that have already created thousands of AWS accounts/S3 buckets, it is not easy to transition Standard storage class objects to Intelligent-Tiering in scale.
How to take a balance between "Introduce automation for scale" and "Avoid any impact on daily operations" becomes an challenge.

To automatically apply a Lifecycle rule to transition Standard object to Intelligent-Tiering, following conditions will be checked in this reference solution.

- Lifecycle rule is not applied
- All stored objects are Standard storage class
- There was no deleted object in the past 90 days
- Average object size is bigger than 128 Kilobytes

## Installation
This sample can be deployed via [AWS CloudFormation](https://aws.amazon.com/cloudformation/) template.
Following parameters can be set through CloudFormation.

![cloudformation_parameters](https://github.com/sk8393/rds-instance-generation-upgrade/assets/13175031/ea51ee61-9e53-4ce6-b73e-a4985794349c)

## Implementation Overview
To store objects as Intelligent-Tiering, this approach uses S3 Lifecycle rule.

This mechanism can be implemented by using a combination of [AWS Config](https://aws.amazon.com/config/) and [AWS Lambda](https://aws.amazon.com/pm/lambda/).

Since we can set any compliance standards (e.g. Amazon Elastic Block Store volume has to be encrypted) in combination with Lambda functions as Config Custom Rule, Config will be used to evaluate usage pattern of each S3 bucket in this approach.
If it is determined that Intelligent-Tiering is suitable for a certain S3 bucket according to its usage pattern, Lifecycle rule will be applied automatically.  

Lambda function of Config Custom Rule will be implemented as the flowchart below.
`Intelligent-Tiering Transition Exemption` tag is to exempt certain S3 bucket from the Lifecycle rule application.
`Notification Sent To User At` tag will be used to store timestamp, when notification was made to S3 bucket owner (e.g. through email, JIRA ticket, etc.).
**COMPLIANT**, **NON_COMPLIANT**, and **NOT_APPLICABLE** are compliance status recorded on Config.

![flowchart](https://github.com/sk8393/rds-instance-generation-upgrade/assets/13175031/c48b4393-c0ce-4a62-ae10-adf92a5bbe15)
