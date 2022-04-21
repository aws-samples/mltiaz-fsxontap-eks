# How to run a Multi-AZ Stateful application on EKS with AWS FSx for NetApp ONTAP

## About the Setup
The infrastructure comprises of an Amazon EKS cluster with three EC2 worker nodes and a FSxONTAP file system that spans across multiple availability zones. After infrastructure deployment, we will walk through how to leverage NetApp’s Trident Container Storage Interface (CSI) (https://netapp-trident.readthedocs.io/en/stable-v19.01/kubernetes/trident-csi.html)to create storage volume powered by FSxONTAP for a MySql database that runs on Amazon EKS cluster. NetApp's Trident Container Storage Interface (CSI) driver provides a CSI interface that allows Amazon EKS clusters to manage the lifecycle of Amazon FSx for NetApp ONTAP file systems.The test environment could be created quite easily with a Infrastructure of Code (IaC) approach thanks to AWS CloudFormation’s capability and we will dive deep into how to deploy Trident CSI Operator into the Amazon EKS cluster via Helm (https://helm.sh/), and creating the storage class, persistent volume claims so as to let the application pod mount on the volume provided by FSxONTAP.

## Architecture Diagram

![Diagram](/Architecture.png)

## Project Structure

```
.
├── FSxONTAP                                # Holds CloudFormation templates for creating the network environment and FSxONTAP file system
│   ├── FSxONTAP.yaml                       # CloudFormation template for creating FSxONTAP File System
│   └── vpc-subnets.yaml                    # CloudFormation template for creating a VPC with two public and private subnets
├── eks                                     # Holds artifacts for creating EKS Cluster and K8S resources to be deployed
│   ├── cluster.yaml                        # Config file for creating an EKS cluster with eksctl
│   └── backend-ontap-nas.yaml              # YAML file for configuring backend settings of Trident CSI
│   └── storage-class-csi-nas.yaml          # YAML file for creating storage class 
│   └── svm_secret.yaml                     # YAML file for creating a k8s secret that stores credentials of FSxONTAP
│   └── pvc-trident.yaml                    # A sample YAML file for creating PVC for trident CSI
│   └── pod_performance_same_AZ.yaml        # A YAML file for running a pod in the same AZ as the FSxONTAP File System for measuring storage performace of FSxONTAP
│   └── pod_performance_different_AZ.yaml.  # A YAML file for running a pod in a different AZ as the FSxONTAP File System for measuring storage performace of FSxONTAP
│   └── mysql                               # Holds MySQL artifacts for K8S
|       ├── mysql-configmap.yaml
|       └── mysql-service.yaml
|       └── mysql-statefulset.yaml
└── ...
```

## Prerequisites

* An AWS account with necessary permissions to create and manage Amazon VPC, Amazon EKS Cluster, Amazon FSx for NetApp ONTAP file system and Cloudformation stack. 
* eksctl, kubectl and Helm3 have been installed in your laptop. 
    * eksctl: https://docs.aws.amazon.com/eks/latest/userguide/eksctl.html 
    * kubectl: https://docs.aws.amazon.com/eks/latest/userguide/install-kubectl.html
    * Helm 3: https://docs.aws.amazon.com/eks/latest/userguide/helm.html
* The AWS Command Line Interface (AWS CLI) should have been configured in your working environment. For information about installing and configuring the AWS CLI, see Installing, updating, and uninstalling the AWS CLI version 2 (https://docs.aws.amazon.com/cli/latest/userguide/install-cliv2.html).
* A good understanding of Amazon EKS, Kubernetes CSI, and Amazon FSx for NetApp ONTAP
* The Trident Operator is supported with Kubernetes 1.17 or later . We will provide instructions on installing Trident using Helm in the following steps. 

# Getting started

## Launch VPC Stack
As you clone the referenced GitHub repo, change directory as below:
```
cd mltiaz-fsxontap-eks/FSxONTAP
```

Launch the CloudFormation stack to set up the network environment for both FSxONTAP and EKS cluster:
```
aws cloudformation create-stack --stack-name EKS-FSXONTAP-VPC --template-body file://./vpc-subnets.yaml --region <region-name>
```

## Create an Amazon FSx for NetApp ONTAP file system
Launch the Cloudformation stack as below to spin up an Amazon FSx for NetApp ONTAP file system. And you need to fill in the parameters and configuration accordingly. Make you choose those two private subnets and VPC as created in the previous step. 

```
aws cloudformation create-stack \
  --stack-name EKS-FSXONTAP \
  --template-body file://./FSxONTAP.yaml \
  --region <region-name> \
  --parameters \
  ParameterKey=Subnet1ID,ParameterValue=[your_preferred_subnet1] \
  ParameterKey=Subnet2ID,ParameterValue=[your_preferred_subnet2] \
  ParameterKey=myVpc,ParameterValue=[your_VPC] \
  ParameterKey=FSxONTAPRouteTable,ParameterValue=[your_routetable] \
  ParameterKey=FileSystemName,ParameterValue=EKS-myFSxONTAP \
  ParameterKey=ThroughputCapacity,ParameterValue=512 \
  ParameterKey=FSxAllowedCIDR,ParameterValue=[your_allowed_CIDR] \
  ParameterKey=FsxAdminPassword,ParameterValue=[Define password] \
  ParameterKey=SvmAdminPassword,ParameterValue=[Define password] \
  --capabilities CAPABILITY_NAMED_IAM    
```

## Create an Amazon EKS cluster
Change to the directory where cluster.yaml file exists:
```
eksctl create cluster -f ./cluster.yaml
```

## Clean up
```
eksctl delete cluster --name=FSxONTAP-eks --region <region-name>
```

```
aws cloudformation delete-stack --stack-name EKS-FSxONTAP --region <region-name>
```

```
aws cloudformation delete-stack --stack-name EKS-MyFSxONTAP --region <region-name>
```

# Security 
See [CONTRIBUTING](https://github.com/aws-samples/mltiaz-fsxontap-eks/blob/main/CONTRIBUTING.md) for more information.

# License
This library is licensed under the MIT-0 License. See the LICENSE file.
