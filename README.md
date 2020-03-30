

# Content Distribution System

Our Content Distribution System is how we serve our downstream users OpenStax content (book) data.
This repository holds our CloudFormation Template to deploy and update our Content Distribution Stack in AWS using the AWS CLI.

**Our Content Distribution Stack is made up of 3 AWS Resources:**  
1. [AWS CloudFront](https://aws.amazon.com/cloudfront/) (CDN Service) - For distribution of the content data   
2. [AWS S3](https://aws.amazon.com/s3/) (Object Storage Service) - For storage of the content data   
3. [AWS CloudFormation](https://aws.amazon.com/cloudformation/) (IaC templating) - For templating and deployment of the above AWS services

Exploration for the *Content Distribution Stack* was done during our [rap-lambda-spike](https://github.com/openstax/rap-spike-lambda#using-cloudformation). (September 2019)

In the final version of the *Content Distribution Stack*, we decided to move forward without the use of AWS Lambda@Edge Functions. Without AWS Lambda@Edge Functions we forgo the need of AWS SAM, which was a layer to help with Infrastructure as Code (IaC) templating with applications that use AWS Lambda Functions. References of SAM and Lambda Functions can be found in the history of this README.md or rap-spike-lambda repository (above).

Note: The difference between a Lambda Function and a Lambda@Edge Function is that a Lambda@Edge function is associated with a CloudFront Distribution.

## Getting Started
This stack is created and deployed with AWS CloudFormation. AWS CloudFormation gives us a single source of truth with a simple text file, in our case `cloudfront-single-origin-bucket.yaml`, to help us describe and provision our AWS cloud infrastructure as oppose to using the AWS Management Console directly.

Reference: [AWS CloudFormation Documentation](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/Welcome.html)

### Prerequisites
You will need to have [AWS CLI](https://aws.amazon.com/cli/) to run commands for AWS CloudFormation.

### Create Content Distribution Stack in AWS
This step only needs to be run once to create the stack. 
To make updates to the stack, see [Update Content Distribution After Changes](#update-content-distribution-after-changes).

Create stack, run AWS CLI Command:

```bash
aws cloudformation deploy \
--stack-name cops-distribution \
--template-file cf-templates/cloudfront-single-origin-bucket.yaml \
--region us-east-1 \
--tags Project=content-distribution Application=content-distribution Environment=dev Owner=ConEng \
--capabilities CAPABILITY_IAM
```
This command will take about 15-25min to complete. It will create the following resources in AWS:

- AN AWS CloudFront Distribution
- AN AWS S3 Bucket: Where Content Data is stored, this will initially be empty.

### Watch Content Distribution Stack Progress
Watch the progress of deployment in the AWS CloudFormation Console [here](https://console.aws.amazon.com/cloudformation/home?region=us-east-1#/stacks). 
Click on the stack name `cops-distribution` and open the `events` tab.

### Update Content Distribution Stack After Changes

If any update is made to the stack via the template the following can be ran:

```bash
aws cloudformation update-stack \ 
--stack-name cops-distribution \
--use-previous-template \
--region us-east-1 \
--capabilities CAPABILITY_AUTO_EXPAND CAPABILITY_IAM
```

### Load Data for Content Distribution Stack
Once the distribution is up the S3 buckets will contain no content (book) data. To load content (book) data into the S3 content bucket see the instructions in [./dump/README.md](./dump/README.md) file, the script to load Books into S3 is currently dependent on the (old) archive.