# Setting Up Prometheus Observability Stack Using Docker

This project covers the step by step guide to setup Observability Stack that contains Prometheus, Grafana & Alert Manager using Terraform, Docker and Docker Compose.

## Prometheus Stack Architecture & Workflow

Here is the high level overview of our setup architecture and workflow.

![284796035-6a9f9d90-e56f-4038-82c7-e8eae29d8bcf](https://github.com/natan-barel/prometheus-observability-stack/assets/132342500/9fae0409-a666-4d0e-953e-db17fabf909d)

## DevOps Tools / Service Used
+ Docker
+ Prometheus
+ Alert Manager
+ Grafana
+ Prometheus Node Exporter
+ Terraform

## Following are the AWS services used
+ ec2
## Following are the Linux concepts covered as part of the setup
+ **Makefile**: Used to modify the server IP address in prometheus config.

+ **Linux SWAP**: To add swap to the ec2 instance.

+ **Shell Scripts**: To install Docker, Docker compose and add swap as part of ec2 system startup.

## Prerequisites
To deploy the Prometheus stack on Docker, we have the following prerequisites:

+ Create an [aws account](https://aws.amazon.com/) with the IAM `AdministratorAccess` permission with a key pair.

+ AWS CLI configured with the account.

+ Install and configure [terraform](https://www.terraform.io/downloads)







## Provision Server Using Terraform

Modify the values of `ec2.tfvars` file present in the `terraform-aws/vars` folder. You need to replace the values **highlighted in bold** with values relevant to your AWS account & region.

If you are using **us-west-2**, you can continue with the same AMI id.

+ **region**
+ **ami_id**
+ **key_name**
+ **vpc_id**
+ **subnet_ids**

```bash
# EC2 Instance Variables
region         = "us-east-2"
ami_id         = "ami-0e83be366243f524a"
instance_type  = "t2.micro"
key_name       = "natan-barel-ec2"
instance_count = 1
volume-size    = 20

# VPC id
vpc_id     = "vpc-0d599cef64779afa1"
subnet_ids = ["subnet-0f8c2a270b689e7ef"]

# Ec2 Tags
name        = "prometheus-stack"
owner       = "natan-barel"
environment = "dev"
cost_center = "natan-barel-center"
application = "monitoring"
```

Now we can provision the AWS EC2 & Security group using Terraform.

```bash
cd terraform-aws/prometheus-stack/
terraform fmt
terraform init
terraform validate
```

Execute the plan and apply the changes.

```bash
terraform plan --var-file=../vars/ec2.tfvars
terraform apply --var-file=../vars/ec2.tfvars
```
Before typing ‘yes‘ make sure the desired resources are being created.
After running Terraform Output should look like the following:

```bash
Apply complete! Resources: 2 added, 0 changed, 0 destroyed.

Outputs:

instance_public_dns = [
  "ec2-34-216-95-97.us-west-2.compute.amazonaws.com",
]
instance_public_ip = [
  "34.216.95.97",
]
instance_state = [
  "running",
]
```
Now we can connect to the AWS EC2 machine just created using the public IP. Replace the key path/name and IP accordingly.

    ssh -i ~/.ssh/natan-barel-ec2.pem ubuntu@34.216.95.97

We will check the cloud-init logs to see if the user data script has run successfully.

    tail /var/log/cloud-init-output.log

Let’s verify the docker and docker-compose versions again.

```bash
sudo docker version
sudo docker-compose version
```
Now that we have the instance ready with the required utilities, let’s deploy the Prometheus stack using docker-compose.

## Deploy Prometheus Stack Using Docker Compose

First, clone the project code repository to the server.

```bash
git clone https://github.com/natan-barel/prometheus-observability-stack
cd prometheus-observability-stack
```

Execute the following **make command** to update server IP in prometheus config file. Because we are running the node exporter on the same server to fetch the server metrics. We also update the alert manager endpoint to the servers public IP address.

```bash
make all
```

Bring up the stack using Docker Compose. It will deploy Prometheus, Alert manager, Node exporter and Grafana

```bash
sudo docker-compose up -d
```
Now, with your servers IP address you can access all the apps on different ports.


+ **Prometheus**: http://your-ip-address:9090
+ **Alert Manager**: http://your-ip-address:9093
+ **Grafana**: http://your-ip-address:3000

Now the stack deployment is done. The rest of the configuration and testing will be done the using the GUI.
