# AWS-Cloud-Migration

Simulating Real-World problem of migrating an "on-premises" application & database to Multicloud Architecture.

![Video App](https://share.vidyard.com/watch/3LkXhhKr5W3fPdY6ECt6Mh?)


### Walkthrough of how to deploy

1. Create a IAM User and grant `AdministratorAccess` ( we will change this later)
2. configure your CLI with new user creds : ```aws configure```
```
cd luxxy-terraform-aws
terraform init
terraform plan
terraform apply
```
3. connect to rds mysql database 
``` 
msql --host=<your-host-name> --port=3306 -u root -p
```
4. Enter the same password when you are prompted using `terraform apply`
5. 
```
use dbcovidtesting;
CREATE USER app@'%' IDENTIFIED BY '<password>';
GRANT ALL PRIVILEGES ON dbcovidtesting.* TO app@'%';
FLUSH PRIVELEGES;
source ~/db/create_table.sql
show tables;
exit;
cd ../luxxy-app/
```
6. Run docker command to generate the images
```
docker built -f Dockerfile -t <your-image-name>
docker tag <your-image-name> <docker-uername>/<your-image-name>:latest
docker push <docker-uername>/<your-image-name>:latest
```
7. If you are running it on MacOS M1,M2 run the following `docker build` command ```docker build --platform linux/amd64 -f Dockerfile -t <your-image-name>``` otherwise while running the pod you might get ```exec format error form gunicorn```. This error usually occurs when the executable binary is incompatible with the architecture of the system on which the container is running
8. Edit the Kubernetes deployement file
```
image: <your-docker-image>
...
- name: AWS_BUCKET
    value: "<your-s3-bucket-name>"
- name: S3_ACCESS_KEY
    value: "xxxxxxxxxxxxxxxxxxxxx"
- name: S3_SECRET_ACCESS_KEY
    value: "xxxxxxxxxxxxxxxxxxxx"
- name: DB_HOST_NAME
    value: "<rds-hostname-endpoint>"
```
9. Connect with the AWS EKS thorugh cli ```aws eks update-kubeconfig --name=<your-eks-cluster-name>  --region=<your-cluster-region>```
10. Now you can use `kubectl` commands as your local kubernetes can access the aws eks cluster
11. Run the kubernetes files
```
kubectl apply -f luxxy-covid-testing-system.yaml
kubectl get pods
kubectl get svc
```
12. If your service is not working then go to your security group check for `eks node security group` and in the manage tags remove the tag which has key not as `Name`.
13. Once again delete all the resource in the kubernetes and deploy them again. If copy the `external-ip` of the service load balancer you app should work