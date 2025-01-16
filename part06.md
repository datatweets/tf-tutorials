# Part 6: Understanding Terraform State Commands: `terraform state list` and `terraform state show`

In this guide, we'll explore two essential Terraform commands that help you interact with and inspect the state of your infrastructure:

- **`terraform state list`**
- **`terraform state show`**

These commands are useful for beginners and experienced users alike, providing insights into the resources managed by Terraform.

---

## What is Terraform State?

Before diving into the commands, it's important to understand what **Terraform State** is:

- **Terraform State File (`terraform.tfstate`)**: This file keeps track of the resources Terraform manages. It maps your configuration to real-world resources, keeping track of metadata, IDs, and attributes.

- **Purpose**: The state file enables Terraform to know what resources it controls, detect changes, and manage updates or deletions accordingly.

---

## The `terraform state` Command

The `terraform state` command is a subcommand used to interact with the state file. It provides several actions to inspect and modify the state.

### Common Subcommands:

- **`list`**: Lists all resources in the state.
- **`show`**: Shows detailed information about a specific resource in the state.

---

## Command: `terraform state list`

### Purpose:

- **List all resources**: Displays all resources that Terraform is currently managing, as recorded in the state file.

### Usage:

```shell
terraform state list
```

### Example Output:

```
aws_eip.one
aws_instance.web-server-instance
aws_internet_gateway.gw
aws_network_interface.web-server-nic
aws_route_table.prod-route-table
aws_route_table_association.a
aws_security_group.allow_web
aws_subnet.subnet-1
aws_vpc.prod-vpc
```

### Explanation:

- Each line represents a resource that Terraform manages.
- The format is `<resource_type>.<resource_name>`.
- **Resource Types**: Indicate the type of resource (e.g., `aws_vpc`, `aws_subnet`, `aws_instance`).
- **Resource Names**: The names you've assigned in your Terraform configuration.

### Why Use It?

- **Quick Overview**: See all resources managed by Terraform in your project.
- **Navigation**: Useful when you need to reference a resource in other commands.
- **Troubleshooting**: Helps identify resources that are part of your state.

---

## Command: `terraform state show`

### Purpose:

- **Show detailed information**: Provides comprehensive details about a specific resource in the state.

### Usage:

```shell
terraform state show <resource_address>
```

- **`<resource_address>`**: The full address of the resource, as shown in the `terraform state list` output.

### Example Steps:

1. **List Resources**:

   ```shell
   terraform state list
   ```

   Output:

   ```
   aws_eip.one
   aws_instance.web-server-instance
   aws_internet_gateway.gw
   aws_network_interface.web-server-nic
   aws_route_table.prod-route-table
   aws_route_table_association.a
   aws_security_group.allow_web
   aws_subnet.subnet-1
   aws_vpc.prod-vpc
   ```

2. **Show Details of a Resource**:

   Let's say we want to see details about the Elastic IP resource.

   ```shell
   terraform state show aws_eip.one
   ```

3. **Example Output**:

   ```
   # aws_eip.one:
   resource "aws_eip" "one" {
       id                              = "eipalloc-0abc123def456ghi7"
       association_id                  = "eipassoc-0abc123def456ghi7"
       domain                          = "vpc"
       instance                        = ""
       network_interface               = "eni-0abc123def456ghi7"
       private_ip                      = "10.0.1.50"
       public_ip                       = "54.123.45.67"
       tags                            = {}
       vpc                             = true
   }
   ```

### Explanation:

- **Resource Block**: Shows the resource type and name (`aws_eip.one`).
- **Attributes**:
  - **`id`**: The unique identifier of the resource in AWS.
  - **`public_ip`**: The public IP address assigned.
  - **`private_ip`**: The private IP associated with the EIP.
  - **`network_interface`**: The network interface the EIP is attached to.
  - **Other attributes**: Provide additional details about the resource.

### Why Use It?

- **Retrieve Information**: Quickly get important details like IP addresses, IDs, and configuration attributes.
- **Debugging**: Useful for troubleshooting issues by inspecting the current state.
- **Scripting**: Automate retrieval of resource details for scripts or automation tasks.

---

## Practical Example: Inspecting an EC2 Instance

Suppose you want to find out the public IP address of your EC2 instance managed by Terraform.

### Steps:

1. **List Resources**:

   ```shell
   terraform state list
   ```

2. **Identify the EC2 Instance Resource**:

   From the output, find the EC2 instance resource, e.g., `aws_instance.web-server-instance`.

3. **Show Details**:

   ```shell
   terraform state show aws_instance.web-server-instance
   ```

4. **Review Output**:

   ```
   # aws_instance.web-server-instance:
   resource "aws_instance" "web-server-instance" {
       ami                          = "ami-085925f297f89fce1"
       arn                          = "arn:aws:ec2:us-east-1:123456789012:instance/i-0abc123def456ghi7"
       availability_zone            = "us-east-1a"
       id                           = "i-0abc123def456ghi7"
       instance_type                = "t2.micro"
       key_name                     = "main-key"
       network_interface_id         = "eni-0abc123def456ghi7"
       private_ip                   = "10.0.1.50"
       public_ip                    = "54.123.45.67"
       tags                         = {
           "Name" = "web-server"
       }
       # ... other attributes ...
   }
   ```

### Key Details:

- **`public_ip`**: The public IP address to connect to the instance.
- **`private_ip`**: The private IP within the VPC.
- **`ami`**: The Amazon Machine Image used.
- **`instance_type`**: The size and type of the instance.

---

## Using `terraform` Command to Discover Subcommands

If you're unsure about available Terraform commands, you can use:

```shell
terraform
```

### Output:

- Lists all main commands and additional subcommands.
- **Main commands**: Commonly used commands like `init`, `plan`, `apply`, `destroy`.
- **All other commands**: Includes `state` and other advanced commands.

---

## Accessing `terraform state` Subcommands

To see what subcommands are available under `terraform state`, run:

```shell
terraform state
```

### Output:

```
Usage: terraform state <subcommand> [options] [args]

Subcommands:
    list        List resources in the state
    show        Show a resource in the state
    mv          Move an item in the state
    pull        Pull current state and output to stdout
    push        Update remote state from a local state file
    rm          Remove instances from the state
    replace-provider Replace provider in the state

For additional help, run 'terraform state <subcommand> -help'
```

---

## Why Are These Commands Important?

- **Visibility**: Understand what resources are managed by Terraform.
- **State Inspection**: View the current state without accessing cloud provider consoles.
- **Troubleshooting**: Identify discrepancies between your configuration and the actual state.
- **Automation**: Use in scripts to automate infrastructure checks.

---

## Additional Tips

### Getting Help for Commands

- For detailed help on any command, use:

  ```shell
  terraform <command> -help
  ```

  Example:

  ```shell
  terraform state show -help
  ```

### Safe Usage

- **Do Not Modify State Manually**: Avoid editing the `terraform.tfstate` file directly, as it may lead to inconsistencies.
- **Backup State Files**: Regularly back up your state files or use remote state backends for collaboration and safety.

### Remote State

- In team environments, consider using a **remote state backend** (like AWS S3, Terraform Cloud) to store state files centrally.

---

## Summary

- **`terraform state list`**: Lists all resources managed by Terraform.
- **`terraform state show <resource>`**: Displays detailed information about a specific resource.
- **Use Cases**:
  - Quickly find resource attributes like IP addresses, IDs.
  - Troubleshoot and ensure resources are in the expected state.
  - Automate infrastructure management tasks.


#terraform
