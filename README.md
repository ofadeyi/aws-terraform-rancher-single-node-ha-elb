# Single Node SSL/HA for Rancher Server in AWS

This repo contains Terraform code and supporting scripts to deploy single node HA Rancher server and Rancher hosts in AWS.

The Terraform plan is designed to be applied in two stages. It will build out and deploy the following resources.

* x1 VPC + IGW
* x2 Public subnets
* x2 Private subnets
* RDS DB subnet group
* Single-AZ or Multi-AZ RDS MySQL DB instance
* SSL enabled elastic load balancer + listeners
* Launch configuration + fixed Multi-AZ auto-scaling group of x1 instance for the Rancher server
* Launch configuration + fixed Multi-AZ auto-scaling group of a specified instance amount for the Rancher hosts
* EC2 IAM policy role for the Rancher server & hosts, granting full access to EC2, S3, Route 53, SNS & Cloudwatch
* RancherOS instance with active Docker running a password protected deployment of Rancher server
* RancherOS instances with active Docker running the Rancher host agent
* Route 53 DNS alias record for the ELB

The estimated deployment time from start to finish is 20-30 minutes.

### Prerequisites

* AWS account
* AWS IAM user account with AWS access/secret keys and permission to create specified resources
* Valid SSL certificate present in the AWS Certificate Manager
* Cygwin (or similar) installed to enable running of .sh scripts if using Windows
* Git installed and configured
* Terraform installed and configured

### How to use the Terraform plan to deploy Rancher server and Rancher hosts

#### Version advisories
Tested with the following versions.

* RancherOS v1.0.3
* Rancher server v1.6.3
* Rancher agent v1.2.4

#### Stage One

* Clone the repo
* Create an EC2 keypair in AWS
* Create an S3 bucket to hold remote state
* Create an S3 bucket to hold remote state
* Update `backend.tf` with the S3 bucket name and region
* Run `terraform init` to initialise remote state
* Create `terraform.tfvars` in the root of the cloned folder (see `terraform.tfvars.example`)
* Set `hst_max`, `hst_min` and `hst_des` in `terraform.tfvars` to zero (0)
* Make up a temporary reg_token in `terraform.tfvars`
* Run `terraform plan` from the root of the folder
* Run `terraform apply` from the root of the folder
* Wait until the installation has completed
* Access Rancher server at the displayed output URL
* Log in with the name and password specified in the `terraform.tfvars` file

#### Stage Two
* Enable hosts registration from within Rancher and copy the token from the registration string. The token will be in the format similar to `6C8B0D1B2E95DD1AA07A:1483142400000:PKQGzShMCv3wtD02DvlU4MkBY0`
* Update `reg_token` in `terraform.tfvars` with the registration token
* Update `hst_max`, `hst_min` and `hst_des` in `terraform.tfvars` with the max, min and desired amount of host instances
* Re-run `terraform plan`
* Re-run `terraform apply`
* The launch configuration will be replaced with a new version and applied to the auto scaling group
* The specified amount of host instances will launch and register with the Rancher server

#### How to remove
* To remove all deployed resources run `terraform destroy`

### Licence

Copyright (c) 2017 Skelton Thatcher Consulting Ltd.

Licensed under the Apache License, Version 2.0 (the "License"); you may not use this file except in compliance with the License. You may obtain a copy of the License at

http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software distributed under the License is distributed on an "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied. See the License for the specific language governing permissions and limitations under the License.

### Acknowledgments

* Based on works produced by [George Cairns](https://www.linkedin.com/in/george-cairns-9624b621/) from [Automation Logic](http://www.automationlogic.com/)
