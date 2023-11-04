# I have followed below links to create mysql, worpress, cert manager, ingress-nginx controller

[https://kubernetes.io/docs/tutorials/stateful-application/mysql-wordpress-persistent-volume/]

[https://medium.com/@ezekiel.umesi/deploy-a-wordpress-application-on-a-kubernetes-cluster-using-ingress-cert-manager-and-helm-1f3b34356197]

Below are the steps I have followed in the kubernetes official docs I have used as reference to create mysql-wordpress and wordpress-deployment yaml files. I have created
kustomize file for secret storing.

In the simpler way by using kubernetes official docs we can download  MySQL deployment configuration file and WordPress configuration file. And you can create kustomize
file and then apply it.

# As per officail docs you can follow the steps for creating wordpress and mysql manifest files.

1) For mysql deployment configuration file: curl -LO https://k8s.io/examples/application/wordpress/mysql-deployment.yaml

2) For wordpress configuration file: curl -LO https://k8s.io/examples/application/wordpress/wordpress-deployment.yaml

3) once the above two steps are done you can add them to kustomization file by running the below command

#cat <<EOF >>./kustomization.yaml
#resources:
#  - mysql-deployment.yaml
#  - wordpress-deployment.yaml
#EOF                              

You can apply now kubectl apply -k ./

# It's time to verify whether the objects are created or not

kubectl get all

For pods: kubectl get pods

For secrets: kubectl get secrets

For PVC: kubectl get pvc
For PV: kubectl get pv

For service: kubectl get svc wordpress where we can verify by accessing with that EXTERNAL-IP. Here we can able to access the worpress

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

# Once after creating wp_production_issuer.yaml apply it

kubectl apply -f wp_production_issuer.yaml

# After creating wordpress_ingress.yaml

kubectl apply -f wordpress_ingress.yaml

# For ingress you can run 

kubectl get ingress

And in DNS that is {route53 in aws} creating a hosting zone creating A recored and mapping the loadblanacer to which the traffic has to handle.








  




