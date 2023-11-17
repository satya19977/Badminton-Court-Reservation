We are deploying our kubernetes cluster in EKS with 4 nodes of type t3.small

# Prerequisites
Download AWS CLI

```
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install
unzip awscliv2.zip
sudo ./aws/install
aws --version

```
Make sure to configure your AWS CLI. 

## Command to create EKS Cluster with the above specifications

```
eksctl create cluster --name=ultimate-cluster --node-type=t3.small --nodes=4  --region=us-east-1 --zones=us-east-1a,us-east-1b,us-east-1c,us-east-1d,us-east-1f

```
