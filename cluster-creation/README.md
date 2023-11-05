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

In the main.tf I have made some of the modifications and I have created S3 bucket and dynamo db for locking the state file, in the terraform.tf file as well 
as we are not using terraform cloud so i have removed that respectieve block.

I have used only single worker node group

I have created backend.tf file as well to provide locking mechanism. But you need to reinitialize this only after creating of s3 and dynamodb. First comment the this backend.tf tile
and proceed to create the resources once you are done you can uncomment this backend.tf file and reinitialize with terraform init

Command where we can use is:
A. terraform init

![image](https://github.com/anilsree6/terraform-with-eks/assets/149375170/29a4fc5b-1162-4a05-a09d-60c30afa37ae)


B. terraform plan [Since the output is I am posting as a summary]
![image](https://github.com/anilsree6/terraform-with-eks/assets/149375170/b2a2acc6-7026-45eb-ab3f-9ae92f10f436)



C. terraform validate
![image](https://github.com/anilsree6/terraform-with-eks/assets/149375170/05c43a2f-c107-43fe-afb2-0de0b97b0a01)


D. terraform apply { once you click on yes it will create resources }
![image](https://github.com/anilsree6/terraform-with-eks/assets/149375170/7bfde5c1-7cdc-459a-b86b-94842edc049a)

The output will look like below
![image](https://github.com/anilsree6/terraform-with-eks/assets/149375170/f77d3e59-1b28-4eca-b1cd-a030594e449a)

E. Now uncomment the backend.tf file and reinitialize with terraform init command, where you will observe that the following output will say
successfully configured backend as s3. The below image will can identify the output.
![image](https://github.com/anilsree6/terraform-with-eks/assets/149375170/96bd5d5f-6308-41ff-b718-347f0bb26f10)

F. Once the above resources and cluster is created you need kubectl to interact with it for this run the below command

aws eks --region $(terraform output -raw region) update-kubeconfig \
--name $(terraform output -raw cluster_name)

Now you will be having access and managing deployments in to the cluster by using kubectl. 

G . Now you can verify the cluster info and as we have created one node with the help of below commands
kubectl cluster-info
![image](https://github.com/anilsree6/terraform-with-eks/assets/149375170/479cde0f-b1a3-4374-ac85-7a7b468f2600)

H. Now you can verify the nodes and your cluster is ready to use now
kubectl get nodes
![image](https://github.com/anilsree6/terraform-with-eks/assets/149375170/3847e89e-6889-44bf-8dca-44f628fafeac)

I. Now we will start creating our mysql-wordpress deployments and as well installing cert manager and ingress nginx controller

J. Let's create kustomization.yaml which will help to store the PASSWORD
( cat <<EOF >./kustomization.yaml
secretGenerator:
- name: mysql-pass
  literals:
  - password=YOUR_PASSWORD
EOF )
 
For your reference see as below image
![image](https://github.com/anilsree6/terraform-with-eks/assets/149375170/e76524db-1b62-4000-a705-82a8cab89b80)

K. Let's download the mysql and wordpress deployment configuration files
MySQL deployment configuration file:
( curl -LO https://k8s.io/examples/application/wordpress/mysql-deployment.yaml )
![image](https://github.com/anilsree6/terraform-with-eks/assets/149375170/4fac66e5-145e-44f4-a55b-49159cb3fd28)

WordPress configuration file:
( curl -LO https://k8s.io/examples/application/wordpress/wordpress-deployment.yaml )
![image](https://github.com/anilsree6/terraform-with-eks/assets/149375170/5d7b492a-9425-4cb2-abbf-323f63facb78)

Once downloaded add them to kustomization.yaml file by running the below command
"cat <<EOF >>./kustomization.yaml
resources:
  - mysql-deployment.yaml
  - wordpress-deployment.yaml
EOF"
Please find the below image for your reference
![image](https://github.com/anilsree6/terraform-with-eks/assets/149375170/1e341992-c485-450f-b618-cde5f2ae8142)

L. Once after completion of the above K step run the command to apply( kubectl apply -k ./ )
The following output tells that your secret of mysql, service,persistent volume claim,deployment of mysql and wordpress will be created 
![image](https://github.com/anilsree6/terraform-with-eks/assets/149375170/2565d4d3-8aca-47a2-84ec-6ad39f495d3a)

M. Once after completion you can go to AWS Console and click on EC2 and in the left plane select LoadBalancers where it will navigate to another page and copy the
DNS name and go to the chrome browser and paste that dns name and hit enter.  you will see wordpress will be up and running.
![image](https://github.com/anilsree6/terraform-with-eks/assets/149375170/37eb6d60-2bee-4db1-a451-860d5cc45088)

Wordpress is up and runnig now.
![image](https://github.com/anilsree6/terraform-with-eks/assets/149375170/a6ee6db5-9483-4560-a662-bfcdf1ae8afc)
![image](https://github.com/anilsree6/terraform-with-eks/assets/149375170/4e6d739e-1600-4e83-beb6-8e686a362b3d)



# Now we will try to use cert manager, ingress nginx controller

We will install both nginx and cert manager using helm.

# Let's install helm we can use this for installing cert manager and ingress nginx

curl https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3 > get_helm.sh
chmod 700 get_helm.sh
./get_helm.sh

# Now let's install Ingress nginx and cert manager

Install ingress nginx:

#helm upgrade --install ingress-nginx ingress-nginx --repo https://kubernetes.github.io/ingress-nginx --namespace ingress-nginx --create-namespace

Install cert manager:

#helm repo add jetstack https://charts.jetstack.io

#helm repo update

#helm install cert-manager jetstack/cert-manager --namespace cert-manager --create-namespace --version v1.7.1 --set installCRDs=true

# You can verify the above two creations by running the below commands

#Kubectl -n ingress-nginx get all 

#kubectl -n cert-manager get all


# Now create wp_production_issuer.yaml file which in present under kubernetes-manifest-files folder in this repo only 

once you create then run kubectl apply -f wp_production_issuer.yaml

# Now we need to create certificate.yamlfile as well and we should have any valid domain registered where we can refer in this yaml file.

once you create certificate yaml file then run kubectl apply -f certificate.yaml

Then you can also follow and run the below commands for good understanding

#kubectl get certificate -w 

#kubectl describe certificate (certificate-name)

#kubectl get secrets

#kubectl describe secret (your-secret)

# I am going through the things mentioned in the official docs of cert manager how to use of let's encypt 
I have tried my best to cope up and implement with let's encrypt by following the below doc, but i was not. I will defenitely implement it.

https://cert-manager.io/docs/tutorials/acme/nginx-ingress/

# Let's destroy using terraform 

N. terraform destroy {run this only if you want to destroy the resources as clean up is also mandatory only for testing purpose}

![image](https://github.com/anilsree6/terraform-with-eks/assets/149375170/c3c83659-d3ff-444d-ad7f-5a9d1f42532c)





