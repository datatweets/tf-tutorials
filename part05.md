# Part 5: Deploying a Web Server on AWS Using Terraform

In this guide, we'll walk through setting up an EC2 instance running a web server on AWS using Terraform. We'll cover each step in detail, explain the code, and reference the official Terraform documentation to ensure accuracy. This guide is designed for beginners and uses simple English to make the concepts easy to understand.

---

## Table of Contents

1. [Introduction](#introduction)
2. [Prerequisites](#prerequisites)
3. [Setting Up the AWS Key Pair](#setting-up-the-aws-key-pair)
4. [Step-by-Step Terraform Configuration](#step-by-step-terraform-configuration)
   - [Step 1: Create a VPC](#step-1-create-a-vpc)
   - [Step 2: Create an Internet Gateway](#step-2-create-an-internet-gateway)
   - [Step 3: Create a Custom Route Table](#step-3-create-a-custom-route-table)
   - [Step 4: Create a Subnet](#step-4-create-a-subnet)
   - [Step 5: Associate Subnet with Route Table](#step-5-associate-subnet-with-route-table)
   - [Step 6: Create a Security Group](#step-6-create-a-security-group)
   - [Step 7: Create a Network Interface](#step-7-create-a-network-interface)
   - [Step 8: Assign an Elastic IP](#step-8-assign-an-elastic-ip)
   - [Step 9: Launch an EC2 Instance](#step-9-launch-an-ec2-instance)
5. [Applying the Terraform Configuration](#applying-the-terraform-configuration)
6. [Connecting to the EC2 Instance](#connecting-to-the-ec2-instance)
   - [For Windows Users (Using PuTTY)](#for-windows-users-using-putty)
   - [For Mac/Linux Users (Using Terminal)](#for-maclinux-users-using-terminal)
7. [Verifying the Web Server](#verifying-the-web-server)
8. [Cleaning Up](#cleaning-up)
9. [Conclusion](#conclusion)
10. [References](#references)

---

## Introduction

We will use Terraform to automate the deployment of an EC2 instance running an Apache web server. The instance will be accessible over the internet, allowing us to visit a webpage served by our server. We'll set up all necessary networking components, including a VPC, subnet, route table, security group, and more.

---

## Prerequisites

- **AWS Account**: You need an AWS account to deploy resources.
- **Terraform Installed**: Make sure Terraform is installed on your machine. [Install Terraform](https://learn.hashicorp.com/tutorials/terraform/install-cli).
- **AWS CLI (Optional)**: For some steps, having the AWS CLI installed can be helpful.
- **Basic Knowledge**: Familiarity with basic AWS concepts and the command line.

---

## Setting Up the AWS Key Pair

Before we begin, we need to create a key pair in AWS to securely connect to our EC2 instance.

### What is a Key Pair?

An **AWS Key Pair** consists of a public key that AWS stores and a private key file that you store. It is used to securely connect to your instance using SSH.

### Steps to Create a Key Pair:

1. **Log in to the AWS Management Console**.

2. **Navigate to EC2 Dashboard**:
   - Click on **Services** > **EC2**.

3. **Create a Key Pair**:
   - In the left navigation pane, click on **Key Pairs** under **Network & Security**.
   - Click **Create key pair**.
   - **Key pair name**: Enter `main-key`.
   - **Key pair type**: Choose **RSA**.
   - **Private key file format**:
     - **PEM**: For Linux/Mac.
     - **PPK**: For Windows (PuTTY).
     - We'll use **PEM** and convert it for Windows later.
   - Click **Create key pair**.
   - The `main-key.pem` file will be downloaded automatically. **Keep this file safe!**

### Important Notes:

- **Security**: Do not share your private key (`.pem` file). It grants access to your instance.
- **Permissions**: For Linux/Mac, you'll need to set the correct permissions on the `.pem` file.

---

## Step-by-Step Terraform Configuration

We'll now write the Terraform code to set up the infrastructure.

### Before You Begin:

- Create a new directory for your Terraform project.
- In that directory, create a file named `main.tf`.

### Let's start coding!

---

### Step 1: Create a VPC

#### What is a VPC?

A **Virtual Private Cloud (VPC)** is a virtual network dedicated to your AWS account. It allows you to launch AWS resources in a logically isolated virtual network.

#### Terraform Code:

```hcl
resource "aws_vpc" "prod-vpc" {
  cidr_block = "10.0.0.0/16"

  tags = {
    Name = "production"
  }
}
```

#### Explanation:

- **Resource Type**: `aws_vpc` tells Terraform we are creating a VPC.
- **Name**: `prod-vpc` is a name we give to the resource within Terraform.
- **`cidr_block`**: Specifies the IP address range for the VPC. `10.0.0.0/16` allows for 65,536 IP addresses.
- **`tags`**: We add a tag with `Name = "production"` to identify the VPC in AWS.

#### Reference:

- [AWS VPC Terraform Documentation](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/vpc)

---

### Step 2: Create an Internet Gateway

#### What is an Internet Gateway?

An **Internet Gateway** allows communication between your VPC and the internet. It enables resources in your VPC to connect to the internet and vice versa.

#### Terraform Code:

```hcl
resource "aws_internet_gateway" "gw" {
  vpc_id = aws_vpc.prod-vpc.id
}
```

#### Explanation:

- **Resource Type**: `aws_internet_gateway` is used to create an internet gateway.
- **Name**: `gw` is our name for the internet gateway resource.
- **`vpc_id`**: We associate the internet gateway with the VPC we created earlier by referencing `aws_vpc.prod-vpc.id`.

#### Reference:

- [AWS Internet Gateway Terraform Documentation](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/internet_gateway)

---

### Step 3: Create a Custom Route Table

#### What is a Route Table?

A **Route Table** contains rules (routes) that determine where network traffic is directed within your VPC.

#### Terraform Code:

```hcl
resource "aws_route_table" "prod-route-table" {
  vpc_id = aws_vpc.prod-vpc.id

  route {
    cidr_block = "0.0.0.0/0"
    gateway_id = aws_internet_gateway.gw.id
  }

  route {
    ipv6_cidr_block = "::/0"
    gateway_id      = aws_internet_gateway.gw.id
  }

  tags = {
    Name = "Prod"
  }
}
```

#### Explanation:

- **Resource Type**: `aws_route_table` is used to create a route table.
- **Name**: `prod-route-table` is our name for the route table.
- **`vpc_id`**: Associates the route table with our VPC.
- **Routes**:
  - **IPv4 Default Route**:
    - **`cidr_block = "0.0.0.0/0"`**: This route matches all IPv4 addresses.
    - **`gateway_id`**: Traffic is directed to the internet gateway.
  - **IPv6 Default Route**:
    - **`ipv6_cidr_block = "::/0"`**: This route matches all IPv6 addresses.
    - **`gateway_id`**: Traffic is directed to the internet gateway.
- **`tags`**: Adds a tag to identify the route table.

#### Reference:

- [AWS Route Table Terraform Documentation](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/route_table)

---

### Step 4: Create a Subnet

#### What is a Subnet?

A **Subnet** is a range of IP addresses in your VPC. You can use subnets to partition your network inside your VPC.

#### Terraform Code:

```hcl
resource "aws_subnet" "subnet-1" {
  vpc_id            = aws_vpc.prod-vpc.id
  cidr_block        = "10.0.1.0/24"
  availability_zone = "us-east-1a"

  tags = {
    Name = "prod-subnet"
  }
}
```

#### Explanation:

- **Resource Type**: `aws_subnet` creates a subnet.
- **Name**: `subnet-1` is our name for the subnet.
- **`vpc_id`**: Associates the subnet with our VPC.
- **`cidr_block`**: IP address range for the subnet. `10.0.1.0/24` allows for 256 IP addresses.
- **`availability_zone`**: Specifies which availability zone to place the subnet in (e.g., `us-east-1a`).
- **`tags`**: Adds a tag to identify the subnet.

#### Reference:

- [AWS Subnet Terraform Documentation](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/subnet)

---

### Step 5: Associate Subnet with Route Table

#### Why Associate a Subnet with a Route Table?

By default, subnets are associated with the main route table of the VPC. Associating our subnet with the custom route table ensures that the routes we defined are applied to our subnet.

#### Terraform Code:

```hcl
resource "aws_route_table_association" "a" {
  subnet_id      = aws_subnet.subnet-1.id
  route_table_id = aws_route_table.prod-route-table.id
}
```

#### Explanation:

- **Resource Type**: `aws_route_table_association` associates a subnet with a route table.
- **Name**: `a` is our name for this association.
- **`subnet_id`**: References the subnet we created.
- **`route_table_id`**: References the route table we created.

#### Reference:

- [AWS Route Table Association Terraform Documentation](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/route_table_association)

---

### Step 6: Create a Security Group

#### What is a Security Group?

A **Security Group** acts as a virtual firewall for your instance to control inbound and outbound traffic.

#### Terraform Code:

```hcl
resource "aws_security_group" "allow_web" {
  name        = "allow_web_traffic"
  description = "Allow Web inbound traffic"
  vpc_id      = aws_vpc.prod-vpc.id

  ingress {
    description = "HTTPS"
    from_port   = 443
    to_port     = 443
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  ingress {
    description = "HTTP"
    from_port   = 80
    to_port     = 80
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  ingress {
    description = "SSH"
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }

  tags = {
    Name = "allow_web"
  }
}
```

#### Explanation:

- **Resource Type**: `aws_security_group` creates a security group.
- **Name**: `allow_web` is our name for the security group.
- **`name`**: Gives the security group a name in AWS.
- **`description`**: Describes what the security group is for.
- **`vpc_id`**: Associates the security group with our VPC.
- **`ingress`** rules (incoming traffic):
  - **HTTPS Rule**:
    - **Ports**: Allows traffic on port 443 (HTTPS).
    - **Protocol**: TCP.
    - **Source**: From any IPv4 address (`0.0.0.0/0`).
  - **HTTP Rule**:
    - **Ports**: Allows traffic on port 80 (HTTP).
    - **Protocol**: TCP.
    - **Source**: From any IPv4 address.
  - **SSH Rule**:
    - **Ports**: Allows traffic on port 22 (SSH).
    - **Protocol**: TCP.
    - **Source**: From any IPv4 address.
- **`egress`** rule (outgoing traffic):
  - **Allows all outbound traffic**.
- **`tags`**: Adds a tag to identify the security group.

#### Note:

- The `cidr_blocks = ["0.0.0.0/0"]` means that the rule applies to any IPv4 address.

#### Reference:

- [AWS Security Group Terraform Documentation](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/security_group)

---

### Step 7: Create a Network Interface

#### What is a Network Interface?

A **Network Interface** is a virtual network interface card (NIC) that can be attached to an instance.

#### Terraform Code:

```hcl
resource "aws_network_interface" "web-server-nic" {
  subnet_id       = aws_subnet.subnet-1.id
  private_ips     = ["10.0.1.50"]
  security_groups = [aws_security_group.allow_web.id]
}
```

#### Explanation:

- **Resource Type**: `aws_network_interface` creates a network interface.
- **Name**: `web-server-nic` is our name for the network interface.
- **`subnet_id`**: Associates the network interface with our subnet.
- **`private_ips`**: Assigns a specific private IP address within the subnet.
- **`security_groups`**: Attaches the security group we created to the network interface.

#### Reference:

- [AWS Network Interface Terraform Documentation](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/network_interface)

---

### Step 8: Assign an Elastic IP

#### What is an Elastic IP?

An **Elastic IP** is a static, public IPv4 address that you can associate with your AWS resources.

#### Terraform Code:

```hcl
resource "aws_eip" "one" {
  vpc                       = true
  network_interface         = aws_network_interface.web-server-nic.id
  associate_with_private_ip = "10.0.1.50"
  depends_on                = [aws_internet_gateway.gw]
}

output "server_public_ip" {
  value = aws_eip.one.public_ip
}
```

#### Explanation:

- **Resource Type**: `aws_eip` creates an Elastic IP.
- **Name**: `one` is our name for the Elastic IP resource.
- **`vpc = true`**: Indicates that the Elastic IP is for a VPC.
- **`network_interface`**: Associates the Elastic IP with our network interface.
- **`associate_with_private_ip`**: Specifies the private IP to associate with the Elastic IP.
- **`depends_on`**: Ensures that the internet gateway is created before the Elastic IP. This is necessary because the Elastic IP requires the internet gateway to be attached to the VPC.
- **Output**:
  - **`server_public_ip`**: Outputs the public IP address after deployment.

#### Note:

- The `depends_on` argument is important here to specify the order of resource creation. Terraform doesn't automatically know that the Elastic IP depends on the internet gateway.

#### Reference:

- [AWS Elastic IP Terraform Documentation](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/eip)

---

### Step 9: Launch an EC2 Instance

#### What is an EC2 Instance?

An **EC2 Instance** is a virtual server in Amazon's Elastic Compute Cloud.

#### Terraform Code:

```hcl
resource "aws_instance" "web-server-instance" {
  ami               = "ami-085925f297f89fce1"
  instance_type     = "t2.micro"
  availability_zone = "us-east-1a"
  key_name          = "main-key"

  network_interface {
    device_index         = 0
    network_interface_id = aws_network_interface.web-server-nic.id
  }

  user_data = <<-EOF
                #!/bin/bash
                sudo apt update -y
                sudo apt install apache2 -y
                sudo systemctl start apache2
                sudo bash -c 'echo your very first web server > /var/www/html/index.html'
                EOF

  tags = {
    Name = "web-server"
  }
}
```

#### Explanation:

- **Resource Type**: `aws_instance` creates an EC2 instance.
- **Name**: `web-server-instance` is our name for the instance.
- **`ami`**: Amazon Machine Image ID for Ubuntu 18.04 LTS (make sure this AMI ID is valid in your region).
- **`instance_type`**: Type of instance (e.g., `t2.micro`).
- **`availability_zone`**: Must match the availability zone of the subnet.
- **`key_name`**: The name of the key pair for SSH access (we created this earlier as `main-key`).
- **`network_interface`**:
  - **`device_index = 0`**: Indicates this is the primary network interface.
  - **`network_interface_id`**: Attaches the network interface we created.
- **`user_data`**:
  - A script that runs on instance launch to:
    - Update the package list.
    - Install Apache web server.
    - Start the Apache service.
    - Write "your very first web server" to the default web page.
- **`tags`**: Adds a tag to identify the instance.

#### Note:

- The `user_data` script allows us to automate the configuration of the instance when it launches.
- Ensure the AMI ID (`ami-085925f297f89fce1`) is valid in your AWS region. You can find the correct AMI ID by searching for Ubuntu images in the AWS Console.

#### Reference:

- [AWS Instance Terraform Documentation](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/instance)

---

## Applying the Terraform Configuration

Now that we've written our Terraform code, let's apply it to create the resources.

### Initialize Terraform:

In your terminal, navigate to the directory containing `main.tf` and run:

```bash
terraform init
```

This command initializes your Terraform project and downloads the necessary provider plugins.

### Apply the Configuration:

Run:

```bash
terraform apply
```

Terraform will show you a plan of the actions it will take. Review the plan carefully.

### Confirm and Execute:

When prompted with:

```plaintext
Do you want to perform these actions?
  Terraform will perform the actions described above.
  Only 'yes' will be accepted to approve.

  Enter a value:
```

Type `yes` and press Enter.

Terraform will proceed to create the resources.

### Output:

Once completed, you'll see an output similar to:

```plaintext
Apply complete! Resources: X added, 0 changed, 0 destroyed.

Outputs:

server_public_ip = "your.ec2.public.ip"
```

**Note down the public IP address**. You'll need it to connect to your instance and verify the web server.

---

## Connecting to the EC2 Instance

### For Windows Users (Using PuTTY)

#### Install PuTTY and PuTTYgen:

- **Download PuTTY**: [PuTTY Download Page](https://www.putty.org/)
- Install both **PuTTY** and **PuTTYgen**.

#### Convert `.pem` to `.ppk` Using PuTTYgen:

1. **Open PuTTYgen**.
2. Click **Load**.
3. In the file dialog, set the file type to **All Files (\*.\*)**.
4. Navigate to where `main-key.pem` is saved.
5. Select `main-key.pem` and click **Open**.
6. You should see a message saying the key was successfully imported.
7. Click **Save private key**.
   - When prompted about saving without a passphrase, click **Yes**.
8. Save the file as `main-key.ppk` in a safe location.

#### Connect Using PuTTY:

1. **Open PuTTY**.
2. **Configure the session**:
   - **Host Name (or IP address)**: `ubuntu@your_public_ip` (replace `your_public_ip` with your EC2 instance's public IP).
   - **Port**: Ensure it's set to **22**.
3. **Load the private key**:
   - In the left sidebar, navigate to **Connection** > **SSH** > **Auth**.
   - Click **Browse** next to **Private key file for authentication**.
   - Select the `main-key.ppk` file you saved earlier.
4. **Start the session**:
   - Click **Open**.
   - If prompted with a security alert about the server's host key, click **Yes** to trust it.
5. **Login**:
   - A terminal window will open, and you should be connected as the `ubuntu` user.

### For Mac/Linux Users (Using Terminal)

#### Set Permissions on the `.pem` File:

In your terminal, set the permissions of the key file:

```bash
chmod 400 /path/to/main-key.pem
```

Replace `/path/to/main-key.pem` with the actual path to your key file.

#### Connect Using SSH:

Run:

```bash
ssh -i /path/to/main-key.pem ubuntu@your_public_ip
```

Replace `/path/to/main-key.pem` with the path to your key file and `your_public_ip` with your EC2 instance's public IP.

When prompted, type `yes` to accept the server's fingerprint.

---

## Verifying the Web Server

Now that you're connected to your EC2 instance, you can verify that Apache is running.

### Check Apache Service:

Run:

```bash
sudo systemctl status apache2
```

You should see that the service is **active (running)**.

### Access the Web Page:

Open a web browser and navigate to:

```
http://your_public_ip
```

You should see a webpage displaying:

```
your very first web server
```

This confirms that Apache is running and serving the page we set up in the `user_data` script.

---

## Cleaning Up

To avoid incurring charges on your AWS account, it's important to destroy the resources when you're done.

### Destroy the Resources:

In your terminal, run:

```bash
terraform destroy
```

Terraform will show you the resources that will be destroyed.

### Confirm:

When prompted, type `yes` to proceed.

Terraform will then delete all the resources it created.

---

## Conclusion

By following these steps, we've:

- Created a VPC, subnet, and associated networking components.
- Configured a security group to allow necessary inbound and outbound traffic.
- Created a network interface and assigned a private IP address.
- Assigned a public Elastic IP address to our instance.
- Launched an EC2 instance running Ubuntu and installed Apache web server using a `user_data` script.
- Created an SSH key pair and connected to the instance from both Windows and Mac/Linux systems.
- Verified that our web server is running and accessible over the internet.

This setup is a foundational pattern for many AWS deployments. Understanding each component helps you build more complex and secure infrastructures.

---

## References

- **Terraform AWS Provider Documentation**:
  - [VPC](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/vpc)
  - [Internet Gateway](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/internet_gateway)
  - [Route Table](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/route_table)
  - [Subnet](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/subnet)
  - [Route Table Association](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/route_table_association)
  - [Security Group](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/security_group)
  - [Network Interface](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/network_interface)
  - [Elastic IP](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/eip)
  - [Instance](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/instance)

- **AWS Documentation**:
  - [Amazon EC2 Key Pairs](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-key-pairs.html)
  - [Connecting to Your Linux Instance](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/AccessingInstancesLinux.html)
  - [PuTTY Download Page](https://www.putty.org/)

---

#terraform
