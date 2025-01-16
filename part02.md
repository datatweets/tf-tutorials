# Part 2: Getting Started with Terraform: Setting Up Your Development Environment and Deploying Your First AWS EC2 Instance

In this part of the tutorial, we'll set up a development environment using Visual Studio Code (VS Code) and create our first Terraform configuration to deploy an AWS EC2 instance. We'll cover:

1. **Installing Visual Studio Code**
2. **Installing the Terraform Extension for VS Code**
3. **Setting Up a Terraform Project Directory**
4. **Understanding Terraform Providers**
5. **Configuring the AWS Provider**
6. **Creating an EC2 Instance Resource**
7. **Initializing and Applying Terraform Configuration**
8. **Verifying the EC2 Instance in AWS**

---

## 1. Installing Visual Studio Code

Visual Studio Code is a free, open-source code editor developed by Microsoft. It supports all major operating systems and provides a rich ecosystem of extensions to enhance your development experience.

### Step 1: Download VS Code

- Navigate to the [Visual Studio Code website](https://code.visualstudio.com/).
- Click on the **"Download for [Your OS]"** button. The website should automatically detect your operating system (Windows, macOS, or Linux).

### Step 2: Install VS Code

- Run the downloaded installer.
- Follow the installation prompts to complete the setup.
  - **Windows Users**: You may be prompted to add VS Code to your system PATH during installation. It's recommended to do so.
  - **macOS Users**: Drag the VS Code application into your **Applications** folder.

---

## 2. Installing the Terraform Extension for VS Code

The Terraform extension provides syntax highlighting, IntelliSense, and other features to improve your Terraform development experience.

### Step 1: Open VS Code

- Launch Visual Studio Code from your applications menu or desktop shortcut.

### Step 2: Install the Terraform Extension

1. Click on the **Extensions** icon in the Activity Bar on the side of VS Code (it looks like four squares).

   ![VS Code Extensions Icon](https://i0.wp.com/www.phdata.io/wp-content/uploads/2021/06/VSCode-Extension-Icon-.png)

2. In the search bar, type **"Terraform"**.
3. Find the **"Terraform"** extension by **HashiCorp**.

   ![Terraform Extension](https://marketplace.visualstudio.com/_apis/public/gallery/publisher/hashicorp/extension/terraform/1.4.0/assetbyname/Microsoft.VisualStudio.Services.Icons.Default)

4. Click **"Install"**.

### Step 3: Verify the Installation

- The extension should now be active. You can confirm by checking that Terraform files (`.tf`) have syntax highlighting and IntelliSense features.

---

## 3. Setting Up a Terraform Project Directory

Organizing your Terraform code in a dedicated project directory helps maintain structure and manage multiple configurations.

### Step 1: Create a Project Directory

- Decide where you want to store your Terraform projects. For example, in your **Documents** folder.
- Create a new folder named **`terraform-projects`**.
- Inside **`terraform-projects`**, create another folder named **`project1`**.

### Step 2: Open the Directory in VS Code

1. In VS Code, click on **"File"** > **"Open Folder..."**.
2. Navigate to the **`project1`** folder you just created.
3. Click **"Select Folder"** (or **"Open"** on macOS).

---

## 4. Understanding Terraform Providers

Terraform uses **providers** to interact with various cloud providers, services, and APIs. Providers are responsible for understanding API interactions and exposing resources.

- **AWS Provider**: Allows Terraform to interact with Amazon Web Services.
- **Syntax**: Providers are configured in Terraform using the `provider` block.

---

## 5. Configuring the AWS Provider

Before Terraform can manage AWS resources, you need to configure the AWS provider with your credentials and region.

### Step 1: Create a Terraform Configuration File

1. In VS Code, within your **`project1`** directory, click on **"File"** > **"New File"**.
2. Name the file **`main.tf`**. The `.tf` extension indicates a Terraform configuration file.

### Step 2: Define the AWS Provider

Add the following code to **`main.tf`**:

```hcl
provider "aws" {
  region     = "us-east-1"
  access_key = "YOUR_ACCESS_KEY_ID"
  secret_key = "YOUR_SECRET_ACCESS_KEY"
}
```

- **`provider "aws"`**: Specifies that we're using the AWS provider.
- **`region`**: The AWS region where resources will be created. `"us-east-1"` corresponds to **Northern Virginia**.
- **`access_key`** and **`secret_key`**: Your AWS credentials.

> **Important Security Note**:
> 
> Hardcoding AWS credentials in your Terraform files is **not recommended** due to security risks. This practice can expose your credentials if the code is shared publicly. For this tutorial, we'll use this method for simplicity, but in production environments, consider using environment variables or AWS CLI configuration files. We'll cover secure methods in a later section.

### Step 3: Obtain Your AWS Credentials

1. **Access Key ID** and **Secret Access Key** are found in your AWS account.
2. Log in to the [AWS Management Console](https://console.aws.amazon.com/).
3. Click on your account name in the top right corner and select **"My Security Credentials"**.
4. Under **"Access keys for CLI, SDK, & API access"**, click **"Create New Access Key"**.
5. Click **"Show Access Key"** or **"Download .csv file"** to obtain your **Access Key ID** and **Secret Access Key**.
   - **Note**: You will not be able to view the secret key again after closing this window. Store it securely.
6. Replace `"YOUR_ACCESS_KEY_ID"` and `"YOUR_SECRET_ACCESS_KEY"` in **`main.tf`** with your actual credentials.

---

## 6. Creating an EC2 Instance Resource

Now that the provider is configured, let's define a resource to create an EC2 instance.

### Step 1: Understand the Resource Block

In Terraform, resources are defined using the following syntax:

```hcl
resource "provider_resource_type" "resource_name" {
  # Configuration options
}
```

- **`provider_resource_type`**: The type of resource (e.g., `aws_instance`).
- **`resource_name`**: An identifier for the resource within Terraform.
- **Configuration options**: Specific settings for the resource.

### Step 2: Add the EC2 Instance Resource

Append the following code to your **`main.tf`** file:

```hcl
resource "aws_instance" "my_first_server" {
  ami           = "ami-0c94855ba95c71c99"
  instance_type = "t2.micro"
}
```

- **`aws_instance`**: Specifies that we're creating an EC2 instance.
- **`my_first_server`**: A local name for the resource within Terraform.
- **`ami`**: The Amazon Machine Image ID for the instance.
  - **`ami-0c94855ba95c71c99`** corresponds to **Ubuntu Server 18.04 LTS** in the **`us-east-1`** region.
- **`instance_type`**: Specifies the EC2 instance type. `"t2.micro"` is eligible for the free tier.

### Step 3: Find the Latest Ubuntu AMI ID (Optional)

AMI IDs can change over time. To find the latest Ubuntu AMI ID:

1. Log in to the AWS Management Console.
2. Navigate to **EC2 Dashboard** > **Instances**.
3. Click **"Launch Instance"**.
4. Search for **"Ubuntu Server 18.04 LTS"**.
5. Note the **AMI ID** for the **`us-east-1`** region.
6. Replace the `ami` value in **`main.tf`** if necessary.

---

## 7. Initializing and Applying Terraform Configuration

With the configuration ready, we'll use Terraform commands to deploy the EC2 instance.

### Step 1: Initialize the Terraform Project

1. Open a terminal in VS Code:
   - Click on **"Terminal"** > **"New Terminal"**.
   - The terminal will open in your **`project1`** directory.

2. Run the initialization command:

   ```shell
   terraform init
   ```

   - This command downloads the AWS provider plugin and sets up the backend.
**Purpose**: Initializes a working directory containing Terraform configuration files.
 **What It Does:**
* **Backend Initialization**: Sets up the backend configuration (e.g., local or remote state storage).
* **Provider Plugins**: Downloads and installs the necessary provider plugins (e.g., AWS, Azure) specified in your configuration.
* **Module Installation**: Retrieves any modules referenced in your configuration.

**When to Run It:**
* Before running any other Terraform commands in a new project.
* After cloning an existing Terraform configuration from version control.
* When adding new providers or modules to your configuration.

**Additional Options:**
* **Reconfigure Backend**: Use terraform init -reconfigure to reinitialize the backend with new settings.
* **Upgrade Plugins**: Use terraform init -upgrade to upgrade provider plugins to the latest acceptable version.


### Step 2: Review the Execution Plan

Before applying changes, it's good practice to review what Terraform will do.

1. Run the plan command:

   ```shell
   terraform plan
   ```
**Purpose**: Creates an execution plan, showing what actions Terraform will take to achieve the desired state defined in your configuration files.
**What It Does:**
* **Reads Current State**: Loads the current state of your infrastructure from the state file or remote backend.
* **Calculates Differences**: Compares the current state with your configuration files to determine necessary changes.
* **Generates Execution Plan**: Outlines the steps Terraform will take to create, update, or destroy resources.

⠀**When to Run It:**
* Before applying changes to verify what Terraform will do.
* After modifying your configuration files to see the impact of your changes.

2. Terraform will output the actions it plans to take, such as:

   - Creating the `aws_instance` resource.
   - Displaying attributes that will be set after creation.

3. Review the output to ensure it matches your expectations.
**Output:**
* Terraform will display a list of actions, prefixed with symbols:
  * `+` (plus): Resource will be created.
  * `-` (minus): Resource will be destroyed.
  * `~` (tilde): Resource will be updated in-place.
* At the end, a summary like:
  ```bash
    Plan: 1 to add, 0 to change, 0 to destroy.
  ```

⠀**Additional Options:**
* **Output to File**: Use `terraform plan -out=plan.out` to save the plan to a file for later use with terraform apply.
* **Variable Overrides**: Use `terraform plan -var 'key=value'` to override variables.
* **Detailed Diff**: Use `terraform plan -detailed-exitcode` to get more granular exit codes.

### Step 3: Apply the Configuration

1. Run the apply command:

   ```shell
   terraform apply
   ```
**Purpose**: Applies the changes required to reach the desired state of the configuration, as outlined by the execution plan.

#### What It Does:

- **Executes Actions**: Performs the create, update, or destroy actions to align real infrastructure with the configuration.
- **Updates State**: Writes the new state of infrastructure to the state file.

#### When to Run It:

- After reviewing the execution plan and confirming the changes.
- When you're ready to apply your configuration changes to the actual infrastructure.

2. Terraform will present the execution plan again and prompt for confirmation:

   ```
   Do you want to perform these actions?
     Terraform will perform the actions described above.
     Only 'yes' will be accepted to approve.
   
     Enter a value:
   ```

3. Type **`yes`** and press **Enter**.

4. Terraform will begin creating the resources. You'll see output like:

   ```
   aws_instance.my_first_server: Creating...
   aws_instance.my_first_server: Creation complete after 1m17s [id=i-0abcd1234efgh5678]
   
   Apply complete! Resources: 1 added, 0 changed, 0 destroyed.
   ```
#### Additional Options:

- **Auto-Approve**: Use `terraform apply -auto-approve` to skip the confirmation prompt (use with caution).
- **Apply Saved Plan**: Use `terraform apply plan.out` if you previously saved a plan using `terraform plan -out=plan.out`.

---
### How to Destroy All Resources
###  `terraform destroy`

**Purpose**: Destroys all resources managed by your Terraform configuration.

#### What It Does:

- **Resource Deletion**: Removes all resources defined in your configuration from your infrastructure.
- **State Update**: Updates the state file to reflect the deletions.

#### When to Run It:

- When you need to tear down your infrastructure completely.
- To clean up resources in a test or development environment to avoid costs.

#### Example Usage:

```shell
terraform destroy
```

#### Interactive Approval:

- Terraform will prompt for confirmation:

```
Do you really want to destroy all resources?
Terraform will destroy all your managed infrastructure.
There is no undo. Only 'yes' will be accepted to confirm.

Enter a value:
```

- Type **`yes`** and press **Enter** to proceed.

#### Output:

- Shows progress messages for each resource being destroyed.
- Upon completion:

```
Destroy complete! Resources: 0 destroyed.
```

#### Additional Options:

- **Auto-Approve**: Use `terraform destroy -auto-approve` to skip the confirmation prompt.
- **Target Specific Resources**: Use `terraform destroy -target=resource_type.resource_name` to destroy specific resources.

---

### Other Useful Terraform Commands

#### `terraform fmt`

**Purpose**: Formats your Terraform code according to the standard style conventions.

#### Usage:

```shell
terraform fmt
```

#### What It Does:

- Scans your configuration files and rewrites them to follow Terraform's canonical style.
- Helps maintain consistency and readability in your codebase.

---

#### `terraform validate`

**Purpose**: Validates the syntax and internal consistency of your Terraform configuration files.

#### Usage:

```shell
terraform validate
```

#### What It Does:

- Checks for syntax errors, invalid arguments, or other issues in your code.
- Does not interact with remote services or check resource IDs.

---

#### `terraform show`

**Purpose**: Displays the current state or a saved plan.

#### Usage:

```shell
terraform show
```

#### What It Does:

- Outputs the contents of the state file or plan in a human-readable format.
- Useful for reviewing the current infrastructure state.

---

#### `terraform output`

**Purpose**: Displays the values of output variables defined in your configuration.

#### Usage:

```shell
terraform output
```

#### What It Does:

- Shows the values of outputs after an apply operation.
- Can be used to retrieve information programmatically.

---

#### `terraform state`

**Purpose**: Advanced command to interact with the state file.

#### Usage:

```shell
terraform state <subcommand>
```

#### Common Subcommands:

- `list`: Lists resources in the state file.
- `show`: Shows detailed information about a resource.
- `rm`: Removes a resource from the state file without destroying it.

---

## Best Practices with Terraform Commands

- **Always Initialize**: Run `terraform init` when setting up a new project or after modifying providers.
- **Plan Before Apply**: Use `terraform plan` to review changes before applying them.
- **Version Control**: Keep your configuration files under version control (e.g., Git).
- **Secure State Files**: Protect your state files, especially if they contain sensitive information.
- **Avoid Hardcoding Credentials**: Use environment variables or shared credentials files for sensitive data.

## Workflow Summary

1. **Write Configuration**: Define your infrastructure in `.tf` files.
2. **Initialize**: Run `terraform init` to set up the project.
3. **Format and Validate**:
- Use `terraform fmt` to format code.
- Use `terraform validate` to check for errors.
4. **Plan**: Run `terraform plan` to see what changes will occur.
5. **Apply**: Run `terraform apply` to implement the changes.
6. **Review Outputs**: Use `terraform output` to view output values.
7. **Destroy (If Needed)**: Run `terraform destroy` to remove resources.

---

By understanding these commands and their purposes, you'll be better equipped to manage your infrastructure effectively using Terraform. Feel free to integrate these explanations into your tutorial where they best fit.

---

## 8. Verifying the EC2 Instance in AWS

After Terraform reports successful creation, verify the EC2 instance in the AWS Management Console.

### Step 1: Navigate to EC2 Instances

- In the AWS Management Console, go to **Services** > **EC2** > **Instances**.

### Step 2: Locate Your Instance

- You should see a new instance with the status **"running"**.
- The instance details should match your configuration:
  - **AMI**: Ubuntu Server 18.04 LTS
  - **Instance Type**: t2.micro
  - **Region**: us-east-1

### Step 3: Review Instance Details

- Click on the instance to view more details, such as:
  - **Instance ID**
  - **Public IP address**
  - **Launch time**

### Step 4: Clean Up (Optional but Recommended)

To avoid unnecessary charges, you can terminate the instance when you're done.

1. In the AWS Console, select the instance.
2. Click on **"Actions"** > **"Instance State"** > **"Terminate"**.
3. Confirm the termination.

Alternatively, you can use Terraform to destroy the resources:

```shell
terraform destroy
```

- This command will prompt for confirmation before destroying the resources managed by your Terraform configuration.

---

## Conclusion

Congratulations! You've successfully set up your development environment and deployed your first AWS EC2 instance using Terraform. You've learned how to:

- Install and configure Visual Studio Code with the Terraform extension.
- Define the AWS provider in Terraform.
- Securely obtain and configure AWS credentials (note the importance of not hardcoding credentials in production).
- Write a Terraform configuration to create an EC2 instance.
- Initialize and apply Terraform configurations.
- Verify the created resources in AWS.

---

## Next Steps

Now that you have the basics down, consider exploring the following topics:

- **Secure Credential Management**: Learn how to use environment variables or AWS CLI credentials to securely manage AWS access keys.
- **Terraform State**: Understand how Terraform tracks resources and how to manage the state file.
- **Variables and Outputs**: Make your configurations more dynamic by using variables and outputs.
- **Provisioners**: Use provisioners to run scripts or commands on your resources after creation.
- **Modules**: Organize and reuse configurations with Terraform modules.

By continuing to build on this foundation, you'll be well on your way to mastering Terraform and infrastructure as code.

---

**Important Security Reminder**:

Remember that hardcoding sensitive information like AWS access keys in your configuration files is a security risk. In production environments, always use secure methods to manage credentials, such as:

- **Environment Variables**
- **AWS Shared Credentials File (`~/.aws/credentials`)**
- **AWS IAM Roles**

We'll cover these methods in more detail in the upcoming sections.

---

Happy Terraforming!
#terraform
