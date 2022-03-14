# Arcadia Practicum

This is an AWS application derived from some of the samples here:

https://aws.amazon.com/cloudformation/templates/aws-cloudformation-templates-us-west-1/

It contains the following:

1) an EC2 instance

* restricted via security groups
  * first, allows internal DB access on the VPC
  * second, external access for HTTP / HTTPS via Application Load Balancer
* uses a series of Ruby scripts to populate a database and start the server

2) a RDS instance, in the form of a MySQL database

* restricted via security group to only have access to the EC2 instance
* is multi-AZ, which improves high availability

3) an Application Load Balancer, along with a target group

* not needed with this simple example, but they are commonly used
* in theory could pipe an application running at port 8080 to port 80 or such
* could also connect it to a CloudFront Distribution, for a CDN

How to make this script scale? Some ideas include:

* old way was to increase the number of EC2 instances of this app
  * more modern way is to either containerize or functionalize your app
  * then it can be put onto Kubernetes or Lambda
    * Kubernetes can scale via replicas or auto-groups
    * AWS Lambda has scaling built-in

* if your app has a lot of common internal requests, can use some sort of key-value caching system
  * Redis and Memcached are often used

* add a CloudFront Distribution CDN in front of the load balancer
  * not needed for this simple example, but CDNs are essential for the modern web
  * can cache responses globally, improving customer performance world-wide
  * caching common responses also reduce load on the EC2 instance, slightly improving scaling

* use Postgres SQL instead of MySQL
  * Postgres is more feature-rich, faster, more reliable
  * mainly used it in this example since a lot of legacy applications have it

What could be done to improve this script?

* add Route 53 entries so that DNS is handled automatically
* consider using software Hashicorp packer to create the AMI with the needed files
  * Amazon's Ruby scripts contained in this are not good
  * alternatively, containerizing the application is a much better alternative

### Instructions

First, go to the AWS panel and find the desired ID of the VPC to deploy it to:

```
vpc-0288077e934b142d6
```

Second, go the Subnets section and select two subnets to use for this:

```
subnet-08316d59ab66809d3
subnet-0108c8886290980a9
```

Third, randomly generate a database password:

```
JgE4ejK5xFOF7FsqRnrRWVS38WUYViWE
```

Afterwards, to create the stack, run the following:

```bash
aws cloudformation create-stack \
    --stack-name arcadia-practicum \
    --region us-east-1 \
    --parameters \
      ParameterKey=DBPassword,ParameterValue=JgE4ejK5xFOF7FsqRnrRWVS38WUYViWE \
      ParameterKey=VpcId,ParameterValue=vpc-0288077e934b142d6 \
      ParameterKey=Subnets,ParameterValue=subnet-08316d59ab66809d3\\,subnet-0108c8886290980a9 \
    --template-body file://hello_world.yaml
```

To update the stack:

```bash
aws cloudformation update-stack \
    --stack-name arcadia-practicum \
    --region us-east-1 \
    --parameters \
      ParameterKey=DBPassword,ParameterValue=JgE4ejK5xFOF7FsqRnrRWVS38WUYViWE \
      ParameterKey=VpcId,ParameterValue=vpc-0288077e934b142d6 \
      ParameterKey=Subnets,ParameterValue=subnet-08316d59ab66809d3\\,subnet-0108c8886290980a9 \
    --template-body file://hello_world.yaml
```

To delete the stack:

```bash
aws cloudformation delete-stack \
    --stack-name arcadia-practicum \
    --region us-east-1
```