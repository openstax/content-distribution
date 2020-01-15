# Content Distribution System

## Using Cloudformation

### Articles that are useful

- [AWS CloudFormation Documentation][aws-cloudformation]
- [Managing Lambda@Edge and CloudFront deployments by using a CI/CD pipeline][aws-cf-lambda-ci]
- [Amazon S3 + Amazon CloudFront: A Match Made in the Cloud][aws-cf-s3]
- [Chalice CloudFormation Support][aws-chalice-support]
- [Chalice Pure Lambda Functions][aws-chalice-pure-lambda]
- [AWS Serverless Application Model][aws-sam]

### Create the content-distribution stack in AWS once

#### Build SAM app

Same steps as in the [SAM app readme][./sam-app/README.md].

* Activate a virtualenv (Python3)
* Build SAM app `cd sam-app` then `sam build -b ../.aws-sam`
* Go to the base directory again `cd ..`

#### Create artifact S3 bucket

```
aws s3 mb s3://ce-artifacts-content-distribution-20200115 --region us-east-1
```

#### Package SAM app

```
aws cloudformation package --template-file ./cf-templates/cloudfront-single-origin-bucket.yaml \
--s3-bucket ce-artifacts-content-distribution-20200115 --output-template-file ./cf-templates/app-output-sam.yaml
```

#### Copy S3 url into cloudfront-single-origin-bucket.yaml

* Open `./cf-templates/app-output-sam.yaml`
* Copy&Paste whole S3 url into lambdaedge CodeUri Section `./cf-templates/cloudfront-single-origin-bucket.yaml`

Code snippet on how it should look like (example):
```yaml
  LambdaEdgeFunction:
    Type: AWS::Serverless::Function
    Properties:
      CodeUri: s3://ce-artifacts-content-distribution-20200115/7661c4754ccd7a4273cc6a631765d0e5
      Role: !GetAtt LambdaEdgeFunctionRole.Arn
      Runtime: python3.7
      Handler: lambda_function.lambda_handler
      Timeout: 5
      AutoPublishAlias: live
```

#### Create the stack now

To create the stack run the following using the `aws` cli:

```
aws cloudformation deploy --template-file cf-templates/cloudfront-single-origin-bucket.yaml --region us-east-1 --stack-name content-distribution --tags Project=content-distribution Application=content-distribution Environment=dev Owner=ConEng --capabilities CAPABILITY_IAM
```

Note: This step only needs to be run the first time to create the stack. If you need to make updates please follow the
instructions in [Update the content-distribution after changes](#update-the-content-distribution-after-changes).

Note: This command will take about 15-25min to complete. It will create the following resources in AWS:

- Cloudfront distribution
- [Lambda Function](./request-handler/lambda_function.py)
- Artifact S3 Bucket
- Raw JSON S3 Bucket
- Baked HTML S3 Bucket
- Resources S3 Bucket

### Update the content-distribution after changes

The `aws cloudformation` command can package up the changes and upload to an s3 bucket.
It allows you to do this by using the `package` argument along with `--s3-bucket` and `--output-template-file` options.

This will package up the lambda function, upload to the s3 bucket, and generate a new template that has
the s3 bucket substituted in the proper locations of the template.

To update the content-distribution stack after a merge run the following: 

    aws cloudformation package --template-file ./cf-templates/cf-cloudfront-content-buckets.yaml \
    --s3-bucket ce-artifacts-content-distribution-20200115 --output-template-file ./cf-templates/app-output-sam.yaml

This will output a SAM compiled version of the template that can be used to update the stack.

Run the following command after the one above to update the stack:

    aws cloudformation update-stack --stack-name content-distribution \
    --region us-east-1 --capabilities CAPABILITY_AUTO_EXPAND CAPABILITY_IAM \
    --template-body file://./cf-templates/app-output-sam.yaml

## Loading the books into s3

Follow the instructions in [./dump/README.md](./dump/README.md) file.

## Using SAM to build and test the Lambda@Edge function

Follow the instruction in [.sam-app/README.md](./sam-app/README.md) file.

[cnx-archive]: https://github.com/openstax/cnx-archive
[cnx-db]: https://github.com/openstax/cnx-db
[content-spike-concourse]: https://github.com/openstax/content-spike-concourse
[content-two-pager]: https://docs.google.com/document/d/1GW5VGrjKmIRw3nbFTIkBZgE0mlHD9ky2TJ_bSUIcJ_w/edit#heading=h.6u0c02buvzha
[aws-cloudformation]: https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/Welcome.html
[aws-chalice-support]: https://chalice.readthedocs.io/en/latest/topics/cfn.html
[aws-chalice-pure-lambda]: https://chalice.readthedocs.io/en/latest/topics/purelambda.html
[aws-sam]: https://aws.amazon.com/serverless/sam/
[aws-cf-lambda-ci]: https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-options.html
[aws-cf-s3]: https://aws.amazon.com/blogs/networking-and-content-delivery/amazon-s3-amazon-cloudfront-a-match-made-in-the-cloud/
