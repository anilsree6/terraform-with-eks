# Pre requisites
AWS account
AWS CLI {Install and configure}
Terraform
AWS IAM authenticator
kubectl

# Provision an EKS Cluster using Terraform I have followed the below link 

[Provision an EKS Cluster tutorial](https://developer.hashicorp.com/terraform/tutorials/kubernetes/eks), containing
Terraform configuration files to provision an EKS cluster on AWS.

Open the above link given by hashicorp and clone the repo as suggested by the hashicorp in to your local machine navigate to the folder. I have followed the steps suggested by the hashicorp official docs. But
as per the requirements I made some of the modification as listed below.

In the main.tf I have made some of the modifications and I have created S3 bucket and dynamo db for locking the state file

I have used only single worker node group

I have created backend.tf file as well to provide locking mechanism. But you need to reinitialize this only after creating of s3 and dynamodb. First comment the this backend.tf tile
and proceed to create the resources once you are done you can uncomment this backend.tf file and reinitialize with terraform init

Command where we can use is:
A. terraform init
B. terraform plan 
C. terraform validate
D. terraform apply
E. terraform destroy {run this only if you want to destroy the resources as clean up is also mandatory only for testing purpose}

Please uncomment the backend.tf file and reinitialize the terraform init where we can store tf state file remote backend that is s3. 



