# Student's Guide: Deploying a Web Server with Terraform Cloud and AWS

Welcome to this hands-on project! In this guide, you'll deploy a web server on AWS using Terraform Cloud. The web server will display a simple webpage where users can submit their contact information, which will be stored in an AWS RDS (Relational Database Service) database. Additionally, users will receive a confirmation message upon successful submission and can view existing contacts.

This project will help you apply your Terraform knowledge and understand how to manage AWS infrastructure as code.

---

## Table of Contents

1. [Introduction](#introduction)
2. [Prerequisites](#prerequisites)
3. [Project Overview](#project-overview)
4. [Step 1: Set Up Terraform Cloud](#step-1-set-up-terraform-cloud)
5. [Step 2: Set Up AWS Free Tier Account](#step-2-set-up-aws-free-tier-account)
6. [Step 3: Create Terraform Configuration Files](#step-3-create-terraform-configuration-files)
   - [3.1. providers.tf](#31-providerstf)
   - [3.2. variables.tf](#32-variablestf)
   - [3.3. main.tf](#33-maintf)
   - [3.4. outputs.tf](#34-outputstf)
7. [Step 4: Configure Terraform Cloud Variables](#step-4-configure-terraform-cloud-variables)
8. [Step 5: Apply the Configuration](#step-5-apply-the-configuration)
9. [Step 6: Test the Web Server and Data Submission](#step-6-test-the-web-server-and-data-submission)
10. [Step 7: Clean Up Resources](#step-7-clean-up-resources)
11. [Conclusion](#conclusion)
12. [References](#references)

---

## Introduction

By the end of this project, you'll have:

- Deployed AWS infrastructure using Terraform Cloud.
- Configured a web server that interacts with an RDS database.
- Gained hands-on experience with Terraform's `main.tf`, `providers.tf`, `variables.tf`, and `outputs.tf` files.
- Enhanced your understanding of Infrastructure as Code (IaC) principles.

---

## Prerequisites

Before starting, ensure you have:

- **Terraform Cloud Account**: [Sign up here](https://app.terraform.io/signup)
- **AWS Free Tier Account**: [Create an account](https://aws.amazon.com/free/)
- **Basic Knowledge of Terraform and AWS**
- **GitHub Account** (optional but recommended)

---

## Project Overview

You'll:

- Set up Terraform Cloud and connect it to a GitHub repository.
- Configure AWS credentials in Terraform Cloud.
- Write Terraform configuration files to deploy AWS resources.
- Set up a web server on an EC2 instance that collects and stores user data in an RDS database.
- Test the web server to ensure it's functioning correctly.

---

## Step 1: Set Up Terraform Cloud

### 1.1. Create a Terraform Cloud Account

1. Go to [Terraform Cloud](https://app.terraform.io/signup).
2. Sign up for a free account using your email.
3. Verify your email address by clicking the link sent to your inbox.

### 1.2. Create an Organization

1. Log in to Terraform Cloud.
2. Click **"New Organization"**.
3. Enter an **Organization Name** (e.g., `my-terraform-org`).
4. Click **"Create Organization"**.

### 1.3. Create a Workspace

1. Within your organization, click **"New Workspace"**.
2. Choose **"Version control workflow"**.
3. Connect your GitHub account and select your repository, or create a new one.
   - If you don't have a repository, you can create one with an empty `README.md` file.
4. Name your workspace (e.g., `webserver-project`).
5. Click **"Create Workspace"**.

---

## Step 2: Set Up AWS Free Tier Account

### 2.1. Create an AWS Account

1. If you don't have an AWS account, [sign up here](https://aws.amazon.com/free/).
2. Follow the on-screen instructions. You may need to provide credit card information, but AWS won't charge you if you stay within the Free Tier limits.

### 2.2. Create AWS Access Keys

1. Log in to the AWS Management Console.
2. Navigate to **IAM (Identity and Access Management)**.
3. Click on **"Users"** in the sidebar.
4. Click **"Add user"**.

   - **User name**: `terraform-user`
   - **Access type**: Check **"Programmatic access"**

5. Click **"Next: Permissions"**.
6. **Attach Policies**:

   - Choose **"Attach existing policies directly"**.
   - Search for **"AdministratorAccess"** and check it.
     - **Note**: For production environments, it's better to follow the principle of least privilege. Here, we're using admin access for simplicity.

7. Click **"Next: Tags"**, then **"Next: Review"**, and finally **"Create user"**.
8. **Save Access Keys**:

   - You'll see an **Access Key ID** and a **Secret Access Key**.
   - Click **"Download .csv"** to save them securely.
   - **Important**: Treat these credentials like passwords. Do not share them.

---

## Step 3: Create Terraform Configuration Files

Create a new directory for your project and navigate to it in your terminal or code editor.

```shell
mkdir terraform-webserver-project
cd terraform-webserver-project
```

You'll create the following files:

- `providers.tf`
- `variables.tf`
- `main.tf`
- `outputs.tf`

### 3.1. providers.tf

The `providers.tf` file tells Terraform which providers to use and any necessary configuration.

#### Instructions:

1. **Create** a file named `providers.tf`.
2. **Define** the required version of Terraform and the AWS provider.
3. **Configure** the AWS provider to use the region specified in a variable.

#### Sample Content:

```hcl
terraform {
  required_version = ">= 1.0.0"

  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 4.0"
    }
  }
}

provider "aws" {
  region = var.aws_region
}
```

**Explanation**:

- The `terraform` block specifies the required versions.
- The `provider` block configures the AWS provider using the `aws_region` variable (which you'll define in `variables.tf`).

### 3.2. variables.tf

The `variables.tf` file declares input variables.

#### Instructions:

1. **Create** a file named `variables.tf`.
2. **Define** variables for:

   - AWS region
   - EC2 instance type
   - RDS database username and password
   - VPC CIDR block
   - Availability zones

3. **Mark** the `db_password` variable as `sensitive` to prevent it from being displayed.

#### Sample Content:

```hcl
variable "aws_region" {
  description = "The AWS region to deploy resources in."
  type        = string
  default     = "us-east-1"
}

variable "instance_type" {
  description = "EC2 instance type."
  type        = string
  default     = "t2.micro"
}

variable "db_username" {
  description = "Username for the RDS database."
  type        = string
  default     = "admin"
}

variable "db_password" {
  description = "Password for the RDS database."
  type        = string
  sensitive   = true
}

variable "vpc_cidr" {
  description = "CIDR block for the VPC."
  type        = string
  default     = "10.0.0.0/16"
}

variable "availability_zones" {
  description = "List of availability zones to use."
  type        = list(string)
  default     = ["us-east-1a", "us-east-1b"]
}
```

**Explanation**:

- Variables allow you to parameterize your configuration.
- Sensitive variables like `db_password` are not displayed in logs or output.

### 3.3. main.tf

The `main.tf` file contains resource definitions.

#### Instructions:

1. **Create** a file named `main.tf`.
2. **Define** resources for:

   - VPC
   - Subnets (in multiple availability zones)
   - Internet gateway
   - Route table and associations
   - Security groups for EC2 and RDS
   - EC2 instance
   - RDS database instance
   - RDS subnet group

3. **Use** variables where appropriate.
4. **Configure** the EC2 instance to run a web server that interacts with the RDS database.

#### Guidance on Writing `main.tf`:

- **VPC and Subnets**:

  - Create a VPC using `aws_vpc`.
  - Use `aws_subnet` with a `count` to create subnets in different availability zones.
  - Use the `cidrsubnet` function to calculate subnet CIDR blocks.

- **Internet Gateway and Route Table**:

  - Create an internet gateway using `aws_internet_gateway`.
  - Create a route table with a default route to the internet gateway.
  - Associate the route table with each subnet.

- **Security Groups**:

  - **EC2 Security Group**:

    - Allows inbound HTTP (port 80) from anywhere.
    - Allows all outbound traffic.

  - **RDS Security Group**:

    - Allows inbound MySQL (port 3306) from the EC2 security group.
    - Allows all outbound traffic.

- **EC2 Instance**:

  - Use `aws_instance` to create the EC2 instance.
  - Set `ami` to the latest Ubuntu 20.04 AMI using a data source (`data "aws_ami"`).
  - Use `user_data` to run a bash script on startup.

- **user_data Script**:

  - Update packages and install Apache, PHP, and PHP MySQL extension.
  - Start and enable Apache.
  - Install MySQL client.
  - Create a PHP script (`index.php`) that:

    - Connects to the RDS database.
    - Handles form submissions to insert data into the database.
    - Displays a form for data entry.
    - Provides a link to view existing contacts.
    - Displays existing contacts in a table.

  - Create the `contacts` database and table by executing SQL commands via the MySQL client.

- **RDS Database Instance**:

  - Use `aws_db_instance` to create the MySQL database.
  - Ensure it is not publicly accessible and uses the RDS subnet group.

- **RDS Subnet Group**:

  - Use `aws_db_subnet_group` to include the subnets.

#### Tips:

- **Placeholders**:

  - Use `${}` to insert variable values into strings.
  - For example, `${var.db_username}` inserts the value of `db_username`.

- **Comments**:

  - Use `#` for comments in HCL files.

- **Indentation and Formatting**:

  - Keep your code well-indented for readability.

#### Example Snippets:

- **Creating Multiple Subnets**:

  ```hcl
  resource "aws_subnet" "main" {
    count                   = length(var.availability_zones)
    vpc_id                  = aws_vpc.main.id
    cidr_block              = cidrsubnet(var.vpc_cidr, 8, count.index)
    availability_zone       = var.availability_zones[count.index]
    map_public_ip_on_launch = true
  
    tags = {
      Name = "main-subnet-${count.index}"
    }
  }
  ```

- **Defining `user_data` for EC2 Instance**:

  The `user_data` section is a bash script that runs when the instance launches.

  - Begin with `#!/bin/bash`.
  - Use `sudo` to execute commands with root privileges.
  - Install necessary packages.
  - Create and write the PHP script to `/var/www/html/index.php`.

### 3.4. outputs.tf

The `outputs.tf` file defines output values that are displayed after Terraform applies the configuration.

#### Instructions:

1. **Create** a file named `outputs.tf`.
2. **Define** outputs for:

   - EC2 instance public IP.
   - RDS database endpoint.
   - URL to access the web server.

#### Sample Content:

```hcl
output "ec2_public_ip" {
  description = "Public IP address of the EC2 instance"
  value       = aws_instance.web.public_ip
}

output "rds_endpoint" {
  description = "Endpoint of the RDS instance"
  value       = aws_db_instance.main.address
}

output "web_url" {
  description = "URL to access the web server"
  value       = "http://${aws_instance.web.public_ip}/index.php"
}
```

**Explanation**:

- Outputs help you quickly find important information after deployment.
- You can reference resource attributes using the syntax `resource_type.resource_name.attribute`.

---

## Step 4: Configure Terraform Cloud Variables

Terraform Cloud needs to know your AWS credentials and any Terraform variables you haven't provided defaults for.

### 4.1. Set AWS Credentials

1. Go to your Terraform Cloud workspace.
2. Click on the **"Variables"** tab.
3. Under **"Environment Variables"**, add:

   - **Key**: `AWS_ACCESS_KEY_ID`
     - **Value**: Your AWS Access Key ID.
     - **Sensitive**: **Check the box** to mark it sensitive.
   - **Key**: `AWS_SECRET_ACCESS_KEY`
     - **Value**: Your AWS Secret Access Key.
     - **Sensitive**: **Check the box**.

### 4.2. Set Terraform Variables

1. Under **"Terraform Variables"**, add:

   - **Key**: `db_password`
     - **Value**: A secure password for your RDS database.
     - **Sensitive**: **Check the box**.
   - You can also set `db_username` if you wish to use a username other than the default (`admin`).

### 4.3. Review Variables

- Ensure all variables are correctly set.
- Sensitive variables will not display their values after saving.

---

## Step 5: Apply the Configuration

### 5.1. Start a New Plan

1. In your Terraform Cloud workspace, click **"Start new plan"** or **"Queue plan"**.
2. Terraform Cloud will start the plan automatically.

### 5.2. Review the Plan

- Once the plan completes, review the proposed changes.
- Ensure that resources are being created as expected.
- Look for any errors or warnings.

### 5.3. Confirm and Apply

1. Click **"Confirm & Apply"**.
2. Optionally, enter a message describing the changes.
3. Click **"Confirm Plan"**.
4. Wait for the apply phase to complete.

**Note**: The apply phase may take several minutes, especially while AWS provisions the RDS instance.

---

## Step 6: Test the Web Server and Data Submission

### 6.1. Access the Web Server

1. Once the apply is complete, navigate to the **"Outputs"** section of your workspace.
2. Find the `web_url` output.
3. Open the URL in your web browser.

### 6.2. Submit Contact Information

1. You should see a form with fields for **Name** and **Email**.
2. Fill in the fields with test data.
3. Click **"Submit"**.
4. You should see a confirmation message: **"New record created successfully"**.

### 6.3. View Existing Contacts

1. Click on the link **"View Contacts"**.
2. A table should display with the contact(s) you have submitted.
3. Add more contacts to test the functionality.

**Troubleshooting Tips**:

- If the webpage doesn't load, wait a few minutes and try again. The EC2 instance may still be initializing.
- Ensure that the security group allows inbound HTTP traffic on port 80.
- Check the AWS Console for the EC2 instance to confirm it's running.

---

## Step 7: Clean Up Resources

It's important to destroy the infrastructure you created to avoid unexpected charges.

### 7.1. Destroy via Terraform Cloud

1. In your Terraform Cloud workspace, click on **"Settings"**.
2. Select **"Destruction and Deletion"**.
3. Click **"Queue destroy plan"**.
4. Confirm the plan when prompted.
5. Wait for Terraform to destroy the resources.

**Note**: Always ensure that the destroy operation completes successfully.

---

## Conclusion

Congratulations! You've successfully:

- Deployed AWS infrastructure using Terraform Cloud.
- Configured a web server that interacts with an RDS database.
- Gained practical experience with Terraform configuration files.
- Enhanced your understanding of Infrastructure as Code.

---

## References

- **Terraform Documentation**:

  - [Terraform AWS Provider](https://registry.terraform.io/providers/hashicorp/aws/latest/docs)
  - [Terraform Variables](https://developer.hashicorp.com/terraform/language/values/variables)
  - [Terraform Outputs](https://developer.hashicorp.com/terraform/language/values/outputs)

- **AWS Documentation**:

  - [AWS EC2](https://docs.aws.amazon.com/ec2/index.html)
  - [AWS RDS](https://docs.aws.amazon.com/rds/index.html)

- **Terraform Cloud**:

  - [Terraform Cloud Getting Started](https://developer.hashicorp.com/terraform/tutorials/cloud-get-started)

---

**Keep Exploring**:

- Experiment with modifying the Terraform configuration to add more features.
- Try deploying the infrastructure in a different AWS region.
- Explore how to manage state and collaborate with others using Terraform Cloud.

**Happy Terraforming!**

---

# Additional Guidance

In this section, we'll delve deeper into some of the critical parts of the project to ensure you have a solid understanding.

## Understanding the `user_data` Script

The `user_data` script is crucial as it configures the EC2 instance to run the web server and interact with the RDS database.

### Breaking Down the Script

1. **Updating and Installing Packages**:

   ```bash
   sudo apt-get update -y
   sudo apt-get install -y apache2 php php-mysql
   sudo systemctl start apache2
   sudo systemctl enable apache2
   ```

   - Updates the package list.
   - Installs Apache web server, PHP, and PHP MySQL extension.
   - Starts and enables Apache to run on boot.

2. **Creating the PHP Script**:

   - The PHP code is written as a heredoc into `/var/www/html/index.php`.
   - It connects to the RDS database using the endpoint and credentials.
   - Handles form submissions to insert data.
   - Displays existing contacts when requested.

3. **Setting Up the Database**:

   ```bash
   sudo apt-get install -y mysql-client
   mysql -h ${aws_db_instance.main.address} -u ${var.db_username} -p${var.db_password} -e "CREATE DATABASE IF NOT EXISTS contacts; USE contacts; CREATE TABLE IF NOT EXISTS contacts (id INT AUTO_INCREMENT PRIMARY KEY, name VARCHAR(255), email VARCHAR(255));"
   ```

   - Installs the MySQL client to interact with the RDS instance.
   - Creates the `contacts` database and table.

### Tips for Writing the `user_data` Script

- **Use EOF Correctly**:

  - The `<<-EOF` and `EOF` syntax allows you to include multi-line strings.
  - Ensure there are no spaces or extra characters after `EOF`.

- **Variable Substitution**:

  - Use double quotes (`"`) in the heredoc to enable variable substitution.
  - Variables like `${aws_db_instance.main.address}` and `${var.db_username}` will be replaced with their actual values.

- **Escaping Characters**:

  - In the PHP code, escape double quotes and dollar signs as needed.
  - For example, `\$_POST['name']` ensures the dollar sign is correctly interpreted in the shell script.
#### The full `user_date` code:
```hcl
 user_data = <<-EOF
              #!/bin/bash
              sudo apt-get update -y
              sudo apt-get install -y apache2 php php-mysql
              sudo systemctl start apache2
              sudo systemctl enable apache2

              # Create a simple PHP script
              echo "<?php
              \$conn = new mysqli('${aws_db_instance.main.address}', '${var.db_username}', '${var.db_password}', 'contacts');
              if (\$conn->connect_error) {
                  die('Connection failed: ' . \$conn->connect_error);
              }

              if (\$_SERVER['REQUEST_METHOD'] == 'POST') {
                  \$name = \$_POST['name'];
                  \$email = \$_POST['email'];
                  \$sql = \"INSERT INTO contacts (name, email) VALUES ('\$name', '\$email')\";
                  if (\$conn->query(\$sql) === TRUE) {
                      echo 'New record created successfully<br>';
                  } else {
                      echo 'Error: ' . \$sql . '<br>' . \$conn->error;
                  }
              }

              echo '<form method=\"POST\">
                      Name: <input type=\"text\" name=\"name\"><br>
                      Email: <input type=\"email\" name=\"email\"><br>
                      <input type=\"submit\" value=\"Submit\">
                    </form>';

              echo '<br><a href=\"?action=view\">View Contacts</a>';

              if (isset(\$_GET['action']) && \$_GET['action'] == 'view') {
                  \$result = \$conn->query(\"SELECT * FROM contacts\");
                  if (\$result->num_rows > 0) {
                      echo '<table border=\"1\"><tr><th>ID</th><th>Name</th><th>Email</th></tr>';
                      while(\$row = \$result->fetch_assoc()) {
                          echo '<tr><td>'.\$row['id'].'</td><td>'.\$row['name'].'</td><td>'.\$row['email'].'</td></tr>';
                      }
                      echo '</table>';
                  } else {
                      echo 'No contacts found.';
                  }
              }

              \$conn->close();
              ?>" | sudo tee /var/www/html/index.php

              # Create the contacts database and table
              sudo apt-get install -y mysql-client
              mysql -h ${aws_db_instance.main.address} -u ${var.db_username} -p${var.db_password} -e "CREATE DATABASE IF NOT EXISTS contacts; USE contacts; CREATE TABLE IF NOT EXISTS contacts (id INT AUTO_INCREMENT PRIMARY KEY, name VARCHAR(255), email VARCHAR(255));"

              EOF
```
## Tips for Debugging

- **Terraform Plan Errors**:

  - If you encounter errors during the plan phase, read the error messages carefully.
  - Common issues include typos, missing variables, or incorrect resource references.

- **AWS Resource Conflicts**:

  - Ensure that CIDR blocks for subnets do not overlap.
  - Use functions like `cidrsubnet` to calculate non-overlapping subnets.

- **Application Issues**:

  - If the web application isn't working, check:

    - Security group rules (especially inbound rules for the EC2 instance).
    - The status of the EC2 instance and RDS database in the AWS Console.
    - Logs on the EC2 instance (requires SSH access, which is beyond the scope of this guide).

## Learning More

- **Terraform Modules**:

  - Explore how to create reusable modules for common infrastructure patterns.

- **State Management**:

  - Understand how Terraform manages state and how to handle state locking and remote state.

- **Variables and Outputs**:

  - Use variable validation and complex variable types (lists, maps, objects).

- **Provisioners**:

  - Learn about provisioners for executing scripts on resources (use sparingly).

---

**Remember**: Practice is key to mastering Terraform and AWS. Don't hesitate to experiment and explore beyond this guide.

#terraform
