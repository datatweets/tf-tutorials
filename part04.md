# Part 4: Understanding VPCs, Subnets, and CIDR Notation in Terraform

In this section, we'll expand our Terraform knowledge by creating a Virtual Private Cloud (VPC) and subnets within AWS. We'll explain:

- What a VPC is and why it's important.
- What subnets are and their role within a VPC.
- What CIDR notation is and how to define IP ranges.
- How to reference resources within Terraform configurations.
- How to interpret Terraform's output during these operations.
- How to verify the created resources in the AWS Management Console.

By the end of this tutorial, you'll have a solid understanding of these concepts and how to implement them using Terraform.

---

## Table of Contents

1. [Introduction to VPCs](#introduction-to-vpcs)
2. [Understanding Subnets](#understanding-subnets)
3. [CIDR Notation and IP Ranges](#cidr-notation-and-ip-ranges)
4. [Creating a VPC with Terraform](#creating-a-vpc-with-terraform)
5. [Creating Subnets within the VPC](#creating-subnets-within-the-vpc)
6. [Referencing Resources in Terraform](#referencing-resources-in-terraform)
7. [Applying and Verifying Changes](#applying-and-verifying-changes)
8. [Interpreting Terraform Output](#interpreting-terraform-output)
9. [Important Concepts and Best Practices](#important-concepts-and-best-practices)
10. [Conclusion](#conclusion)

---

## Introduction to VPCs

### What is a VPC?

A **Virtual Private Cloud (VPC)** is a virtual network dedicated to your AWS account. It is logically isolated from other virtual networks in the AWS cloud, providing you with control over your virtual networking environment.

### Why Use a VPC?

- **Isolation**: VPCs allow you to isolate your resources from other users in the AWS cloud.
- **Security**: You can control inbound and outbound traffic at the subnet and instance levels.
- **Customization**: You can configure IP address ranges, create subnets, and configure route tables and network gateways.

### Key Features of VPCs

- **Customizable IP Address Range**: You define the IP address range using CIDR notation.
- **Subnets**: Divide the VPC's IP address range into smaller ranges.
- **Security Groups and Network ACLs**: Control traffic at the instance and subnet levels.

---

## Understanding Subnets

### What is a Subnet?

A **subnet** is a range of IP addresses in your VPC. Subnets allow you to partition your network inside your VPC.

### Types of Subnets

- **Public Subnets**: Subnets that have a route to an Internet Gateway, allowing resources to access the internet.
- **Private Subnets**: Subnets that do not have a route to the Internet Gateway. Resources in these subnets are isolated from the internet.

### Why Use Subnets?

- **Network Organization**: Organize resources based on security, operational, or functional needs.
- **Security**: Isolate resources at the subnet level.
- **Availability Zones**: Distribute resources across different availability zones for high availability.

---

## CIDR Notation and IP Ranges

### What is CIDR Notation?

**CIDR (Classless Inter-Domain Routing)** notation is a compact representation of an IP address and its associated routing prefix.

### Understanding CIDR Blocks

- **Format**: `IP_address/prefix_length`
- **Example**: `192.168.0.0/16`
  - `192.168.0.0` is the base IP address.
  - `/16` indicates the number of bits in the subnet mask.

### Calculating IP Ranges

- **Subnet Mask**: Determines how many IP addresses are available in the network.
- **Calculations**:
  - `/16` provides 65,536 IP addresses (2^(32-16)).
  - `/24` provides 256 IP addresses (2^(32-24)).

### Choosing CIDR Blocks

- **Avoid Overlaps**: Ensure that CIDR blocks of different VPCs or subnets do not overlap.
- **Private IP Ranges**: Use private IP ranges as defined in RFC 1918:
  - `10.0.0.0 – 10.255.255.255` (10/8 prefix)
  - `172.16.0.0 – 172.31.255.255` (172.16/12 prefix)
  - `192.168.0.0 – 192.168.255.255` (192.168/16 prefix)

---

## Creating a VPC with Terraform

Now that we've covered the basics, let's create a VPC using Terraform.

### Step 1: Update Your Terraform Configuration

In your `main.tf` file, add the following code to define a VPC:

```hcl
resource "aws_vpc" "my_first_vpc" {
  cidr_block = "10.0.0.0/16"

  tags = {
    Name = "Production VPC"
  }
}
```

#### Explanation:

- **`resource "aws_vpc" "my_first_vpc"`**: Declares a resource of type `aws_vpc` with the local name `my_first_vpc`.
- **`cidr_block = "10.0.0.0/16"`**: Assigns the IP range for the VPC.
- **`tags`**: Adds a tag with `Name = "Production VPC"` to help identify the VPC in AWS.

### Step 2: Initialize Terraform (If Needed)

If you haven't initialized your Terraform project or if you added new providers, run:

```shell
terraform init
```

---

## Creating Subnets within the VPC

Next, we'll create a subnet within the VPC we just defined.

### Step 1: Define the Subnet Resource

Add the following code to your `main.tf` file:

```hcl
resource "aws_subnet" "subnet1" {
  vpc_id     = aws_vpc.my_first_vpc.id
  cidr_block = "10.0.1.0/24"

  tags = {
    Name = "Prod Subnet 1"
  }
}
```

#### Explanation:

- **`resource "aws_subnet" "subnet1"`**: Declares a resource of type `aws_subnet` with the local name `subnet1`.
- **`vpc_id = aws_vpc.my_first_vpc.id`**: References the VPC we created earlier by accessing its `id` attribute.
- **`cidr_block = "10.0.1.0/24"`**: Assigns an IP range for the subnet within the VPC's CIDR block.
- **`tags`**: Adds a tag with `Name = "Prod Subnet 1"`.

### Step 2: Understanding the CIDR Block for the Subnet

- **VPC CIDR Block**: `10.0.0.0/16` (provides 65,536 IP addresses).
- **Subnet CIDR Block**: `10.0.1.0/24` (provides 256 IP addresses).
- **Hierarchy**: The subnet's CIDR block must be within the VPC's CIDR block.

---

## Referencing Resources in Terraform

### How to Reference Other Resources

In Terraform, you can reference attributes of other resources using the following syntax:

```hcl
resource_type.resource_name.attribute
```

#### Example:

```hcl
vpc_id = aws_vpc.my_first_vpc.id
```

- **`aws_vpc`**: The type of the resource you are referencing.
- **`my_first_vpc`**: The local name given to the resource.
- **`id`**: The attribute of the resource you want to access.

### Why This Works

- **Dependency Graph**: Terraform builds a dependency graph to determine the order of resource creation.
- **Automatic Ordering**: Terraform knows that the VPC must be created before the subnet because the subnet depends on the VPC's ID.

---

## Applying and Verifying Changes

### Step 1: Run Terraform Apply

Apply your configuration to create the VPC and subnet:

```shell
terraform apply
```

#### Expected Output:

- Terraform will show a plan indicating that it will create two resources:
  - `aws_vpc.my_first_vpc`
  - `aws_subnet.subnet1`

**Example Plan Summary**:

```plaintext
Plan: 2 to add, 0 to change, 0 to destroy.
```

- Confirm by typing **`yes`** when prompted.

### Step 2: Review the Output

After the apply completes, you'll see messages like:

```plaintext
aws_vpc.my_first_vpc: Creation complete after 5s [id=vpc-0a1b2c3d4e5f6g7h8]
aws_subnet.subnet1: Creation complete after 3s [id=subnet-0h7g6f5e4d3c2b1a0]

Apply complete! Resources: 2 added, 0 changed, 0 destroyed.
```

- **Important Output Elements**:
  - **Resource Creation Order**: Terraform created the VPC first, then the subnet.
  - **Resource IDs**: The IDs assigned by AWS are displayed in the output.

### Step 3: Verify in the AWS Management Console

#### **1. Verify the VPC**

1. Log in to the AWS Management Console.
2. Navigate to **Services** > **VPC** > **Your VPCs**.
3. Locate the VPC named **"Production VPC"**.
   - Check the **CIDR Block**: It should be `10.0.0.0/16`.
   - Check the **Tags**: Should have `Name = Production VPC`.

#### **2. Verify the Subnet**

1. In the VPC Dashboard, navigate to **Subnets**.
2. Locate the subnet named **"Prod Subnet 1"**.
   - Check the **VPC ID**: Should match the VPC you created.
   - Check the **CIDR Block**: Should be `10.0.1.0/24`.
   - Check the **Tags**: Should have `Name = Prod Subnet 1`.

**Note**: You might see other default VPCs and subnets. Use the **Name** tag and **VPC ID** to identify your resources.

---

## Interpreting Terraform Output

### Understanding Resource Creation Order

- **Terraform's Dependency Graph**: Determines the order of resource creation based on dependencies.
- **Example**: Since `aws_subnet.subnet1` depends on `aws_vpc.my_first_vpc`, Terraform ensures the VPC is created first.

### Highlighted Output Sections

- **Resource Actions**:
  - **`+` (plus sign)**: Indicates a resource will be created.
- **Resource IDs**:
  - The `[id=...]` indicates the unique identifier assigned by AWS.
- **Plan Summary**:
  - **`Plan: 2 to add, 0 to change, 0 to destroy.`** provides a quick overview.

### Automatic Approval (Optional)

- **Skipping Confirmation**:
  - Use `terraform apply -auto-approve` to skip the `yes` confirmation.
  - **Caution**: This should be used carefully, especially in production environments.

---

## Important Concepts and Best Practices

### Terraform's Declarative Approach

- **Declarative Configuration**: You define **what** you want, and Terraform figures out **how** to achieve it.
- **Order of Resources in Code**:
  - The physical order of resource definitions in your `.tf` files doesn't affect the order of resource creation.
  - Terraform uses the dependency graph to manage resource creation.

### Resource Dependencies

- **Implicit Dependencies**:
  - Occur when you reference another resource's attributes.
  - **Example**: The subnet depends on the VPC because it uses `vpc_id = aws_vpc.my_first_vpc.id`.
- **Explicit Dependencies**:
  - Can be declared using the `depends_on` argument if needed.

### Tags

- **Importance of Tags**:
  - Helps organize and manage resources.
  - Useful for cost allocation, automation, and searching in the AWS console.
- **Defining Tags in Terraform**:
  - Use the `tags` argument within the resource block.
  - Tags are key-value pairs.

### Managing CIDR Blocks

- **Avoiding Overlaps**:
  - Ensure that subnets' CIDR blocks do not overlap with each other or with the VPC's CIDR block.
- **Subnetting**:
  - Plan your network architecture to allocate IP ranges appropriately.

### Verifying Resources

- **AWS Management Console**:
  - Always verify that resources are created as expected.
  - Check properties like CIDR blocks, tags, and relationships.

### Destroying Resources

- **Cleaning Up**:
  - Use `terraform destroy` to remove all resources defined in your configuration.
- **Selective Destruction**:
  - Use `terraform destroy -target=aws_subnet.subnet1` to destroy a specific resource.

---

## Conclusion

In this tutorial, you've learned:

- **What VPCs and subnets are** and their roles in AWS networking.
- **How to use CIDR notation** to define IP ranges for VPCs and subnets.
- **How to create a VPC and subnet using Terraform**, including writing the configuration code.
- **How to reference resources in Terraform**, enabling dependencies and resource relationships.
- **How to interpret Terraform's output**, understanding the plan and apply processes.
- **Best practices for managing infrastructure with Terraform**, including tagging and CIDR block management.

By understanding these concepts and practicing with Terraform, you're well on your way to mastering infrastructure as code and efficient cloud resource management.

---

## Next Steps

- **Experiment**: Try creating additional subnets, perhaps one public and one private, and see how Terraform manages the dependencies.
- **Variables**: Introduce variables to make your configurations more dynamic and reusable.
- **Modules**: Learn about Terraform modules to organize and encapsulate your code.
- **Security Groups and Network ACLs**: Explore adding security controls to your VPC and subnets.
- **Load Balancers and Auto Scaling**: Build upon this foundation to create more complex infrastructure.

---

**Happy Terraforming!**

---

**References:**

- [Terraform AWS Provider Documentation](https://registry.terraform.io/providers/hashicorp/aws/latest/docs)
- [AWS VPC User Guide](https://docs.aws.amazon.com/vpc/latest/userguide/what-is-amazon-vpc.html)
- [Understanding CIDR Notation](https://www.ipaddressguide.com/cidr)

---

**Note**: Always ensure that your AWS credentials are securely managed and that you have appropriate permissions to create and manage resources in your AWS account.
#terraform
