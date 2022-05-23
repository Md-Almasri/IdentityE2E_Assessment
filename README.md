# Infrastructure as code

Solution that deploy microservices using infrastructure as code.

## Prerequisites:
* AWS account
* AWS-CLI installed and configured (Access and Secret Keys)
* Docker installed.
* Git installed.
* Python installed.
* Boto3 installed.
* Terraform installed.

## Deployment Solution Overview


![solution workflow](https://d2908q01vomqb2.cloudfront.net/ca3512f4dfa95a03169c5a670a4c91a19b3077b4/2021/04/05/awsjoe_Flask-Microservice_f1.png)


The diagram shows the process and a summary of what will be performed and delivered.

## Backend deployment

### Create Amazon Elastic Container Registry ECR
Since we have docker files for backend and front end app, we will use ECR to create repository image in our AWS account:

* Create an ECR repository by Running the following AWS CLI command in the terminal:
```
aws ecr create-repository --repository-name identity-e2e-backend --image-scanning-configuration scanOnPush=true --region eu-west-2
```

* Retrieve an authentication token and authenticate your Docker client to your registry:
```
aws ecr get-login-password --region eu-west-2 | docker login --username AWS --password-stdin <AWS_ID>.dkr.ecr.eu-west-2.amazonaws.com
```

The message recieved:
```
Login Succeeded
```

* Build our backend docker image:
```
docker build -t identity-e2e-backend .
```

* Tag your image so you can push the image to this repository:
```
docker tag identity-e2e-backend:latest <AWS_ID>.dkr.ecr.eu-west-2.amazonaws.com/identity-e2e-backend:latest
```

* Push this image to your newly created AWS repository:
```
docker push <AWS_ID>.dkr.ecr.eu-west-2.amazonaws.com/identity-e2e-backend:latest
```

### Define resources and infrastructure

We will use an open source tool called [Terraform](https://www.terraform.io/) that helps to define infrastructure as code on numerous platforms, including Amazon Web Services.

The Terraform code will deliver resources to the following configuration aspects:

* IAM: Identity access management policy configuration
* VPC: Public subnets, routes.
* EC2: Autoscaling implementation
* ECS: Cluster configuration
* ALB: Load balancer configuration
* DynamoDB: Table configuration
* CloudWatch: Alert metrics configuration

To define infrastructure:

* Get into \backend\terraform directory

* Initialize the working directory for Terraform on our workstation
```
terraform init
```

* Deploy the application environment
```
terraform apply
```
When prompted to provide the ECR image path, copy the URL from aws ECR consol.
Retrieve the DNS name for the load balancer from the alb_dns_name output.

## Test

* Backend is running on:
```
http://ecsalb-734033670.eu-west-2.elb.amazonaws.com
```
The output will be:
```
Healthy
```

And by checking different endpoint:
```
http://ecsalb-734033670.eu-west-2.elb.amazonaws.com/api/v1/get
```
The output will be:
```
{
    "Count":0,
    "Items":[],
    "ResponseMetadata":
        {"HTTPHeaders":
            {
                "connection":"keep-alive",
                "content-length":"39",
                "content-type":"application/x-amz-json-1.0",
                "date":"Sun, 22 May 2022 22:42:53 GMT",
                "server":"Server",
                "x-amz-crc32":"3413411624","x-amzn-requestid":"QL6Q2RVVMS0535N9KGNNCMEL8NVV4KQNSO5AEMVJF66Q9ASUAAJG"
            },
            "HTTPStatusCode":200,
            "RequestId":"QL6Q2RVVMS0535N9KGNNCMEL8NVV4KQNSO5AEMVJF66Q9ASUAAJG",
            "RetryAttempts":0
        },
    "ScannedCount":0}
```

## Future work

using CI.

The frontend needs to be deployed in the same way we deployed the backend taking into considration the connection between frontend and backend.

## Consideration

The solution would allow us to add more application by running more images and deploy them using AWS.

AWS provides a number of features that enable customers to easily encrypt data and manage the keys. All AWS services offer the ability to encrypt data at rest and in transit. [AWS KMS](https://docs.aws.amazon.com/whitepapers/latest/logical-separation/encrypting-data-at-rest-and--in-transit.html) gives you centralized control over the cryptographic keys used to protect your data. it integrates with the majority of services to let customers control the lifecycle of and permissions on the keys used to encrypt data on the customer’s behalf.

[AWS CodePipeline](https://aws.amazon.com/codepipeline/) is a fully managed continuous delivery service. It automates the build, test, and deploy phases of release process every time there is a code change, based on the release model we define.

## Resources

* [Deploying Python Flask microservices to AWS using open source tools](https://aws.amazon.com/blogs/opensource/deploying-python-flask-microservices-to-aws-using-open-source-tools/)

