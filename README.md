# Cloudformation High Availability Web App
This folder provides the starter code forthe "ND9991 - C2- Infrastructure as Code - Deploy a high-availability web app using CloudFormation" project. This folder contains the following files:

### Table of Contents

- [Project Description](#project)
- [Project Files](#files)
- [How To Run](#run)
- [Results](#results)
- [Licensing, Authors, Acknowledgements](#licensing-authors-acknowledgements)

## Project Details<a name="project"></a>

This project is part of the Udacity Nanodegree "Cloud DevOps Engineer".
Goal is to apply the learned content about Infrastruce As Code (IAC) and AWS to develop a Cloudformation script that deploys a high availability web app.

The deployment script creates the networking infrastructure as well as server infrastructure. The network infrastructure is build into a Virtual Private Network, which spreads over two Availability Zones containing each a Public Subnet and a Private Subnet. Two availability zones ensure high availability for the web app, even if one Availability Zone is down. The web app is served via EC2 instances in the private subnets. To allow the web app to scale elastically with higher inbound traffic, the EC2 instances serving the web app are placed into a EC2 Auto Scaling Group. An Internet Gateway allows incoming traffic on to be distributed to the different EC2 instances by the Elastic Load Balancer.

The web app architecture is depicted in the image below:

![](./controllers_brief.svg)
<img src="./images/HighAvailabilityWebApp.svg">

## File Descriptions <a name="files"></a>

Here you can see the files of the project with a short description:

```
cloudformation-high-availability-web-app/
│
├── README.md
├── app/ 
    ├── create.sh --> shell script to start a new deployment
    ├── update.sh --> shell script to update a running deployment
    ├── delete.sh --> shell script to delete a running deployment
    ├── network.yml --> YAML file which describes the network components to deploy the web app
    ├── network-parameters.json --> json file containing the parameters for the network 
    ├── server.yml --> YAML file which describes the servers components to deploy the web app
    ├── server-parameters.json --> json file containing the parameters for the servers 
├── images/
├── LICENSE


```
## How To Run<a name="run"></a>

### Prepare

- Step 1: Install the AWS CLI Tool on your machine. You can check if installed properly with:
```
$ aws --version
```

- Step 2: Create an Access Key ID for IAM User with programmatic access via the AWS Console.

- Step 3: Configure AWS CLI Tool with newly created keys of user and set 
```
$ aws configure
```

- Step 4: Check if AWS CLI configuration works properly: 
```
$ aws iam list-users
```

### Deploy Infrastructure

- Step 5: Create the network stack for the High Availability Web App:
```
$ ./create.sh hawa-network hawa-network.yml hawa-network-parameters.json
```

### Deploy Servers

```
$ ./create.sh hawa-server hawa-server.yml hawa-server-parameters.json
```

### Inspect Servers

#### Inspect private Servers using Jump Host

It is possible to ssh into the EC2 Web Server instances using a jump host. The jump host is another EC2 instance placed into one of the public subnets. The security group of the jump host allows incoming access from a configured IP to port 22 using a defined .pem file. Once connected via SSH to the jump host it is possible to connect with the jump host via SSH to the Web Server instances in the privat subnets, as the security groups allow instances in the private subnets to be accessed from within the same Virtual Private Cloud. Here the steps to connect to a EC2 Server Instance in the private subnets:

- Step 1: Grant rights on local machine to .pem file
```
$ chmod 400 jumpHostKey.pem 
```

- Step 2: Safe copy the necessary .pem file from local machine to jump host:
```
$ scp -i jumpHostKey.pem privateSubnetInstanceKey.pem ec2-user@<public-ip4-jump-host>:/home/ec2-user/privateSubnetInstanceKey.pem 
```

- Step 3: SSH into jump host
```
$ ssh ec2-user@<public-ip4-jump-host> -i jumpHostKey.pem 
```

- Step 4: Grant rights on jump host to .pem file
```
$ chmod 400 privateSubnetInstanceKey.pem 
```

- Step 5: SSH into private EC2 Web Server instance
```
$ ssh ubuntu@<private-ip4-ec-instance> -i privateSubnetInstanceKey.pem 
```

The logs of the UserData scripts are at /var/log/cloud-init-output.log

#### Inspect private Servers using user-data script

The User-Data script of the Web App Launch Configuration in this project contains a line which copies the logs of the private servers to the console (exec >>(/var/log/user-data.log|logger -t user-data -s 2>/dev/console) 2>&1). This allows to see the logs either using the aws web console EC2>Instances>(mark instance)>Actions>MonitorAndTroubleshoot>GetSystemLogs or via the aws cli:

```
$ aws ec2 get-console-output --instance-id i-0ab9802bd29642468
```

### Delete the instances and network

- Step 6: Delete stack:

```
$ ./delete.sh hawa-server
```

```
$ ./delete.sh hawa-network
```


## Results<a name="results"></a>

...


## Licensing, Authors, Acknowledgements<a name="licensing"></a>

Feel free to use my code as you please:

Copyright 2021 Leopold Walther

Permission is hereby granted, free of charge, to any person obtaining a copy of this software and associated documentation files (the "Software"), to deal in the Software without restriction, including without limitation the rights to use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of the Software, and to permit persons to whom the Software is furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.