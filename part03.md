# Part 3: Modifying Resources with Terraform: Understanding Declarative Infrastructure and Managing Changes

In this section, we'll delve deeper into how Terraform manages infrastructure changes. You'll learn how Terraform's declarative approach affects resource management, how to modify existing resources, and how to interpret Terraform's output during these operations.

We'll cover:

1. **Understanding Terraform's Declarative Nature**
2. **Re-Creating the EC2 Instance**
3. **Adding Tags to Resources**
4. **Understanding the Terraform Plan and Apply Outputs**
5. **Verifying Changes in the AWS Console**

---

## 1. Understanding Terraform's Declarative Nature

Before we proceed, it's crucial to understand that Terraform is a **declarative** infrastructure-as-code tool. This means you declare the desired state of your infrastructure in your configuration files, and Terraform ensures that the actual infrastructure matches this state.

### **Imperative vs. Declarative**

- **Imperative Approach**: You specify **how** to achieve a result by defining step-by-step instructions.
- **Declarative Approach**: You specify **what** the desired end state is, and the tool figures out how to achieve it.

**In Terraform**, you describe the desired infrastructure, and Terraform determines the necessary steps to achieve that state, whether it's creating, updating, or deleting resources.

---

## 2. Re-Creating the EC2 Instance

Since we destroyed the EC2 instance in the previous steps, we'll need to recreate it to demonstrate modifying resources.

### **Step 1: Ensure Your Configuration is Correct**

Make sure your `main.tf` file contains the following configuration:

```hcl
provider "aws" {
  region     = "us-east-1"
  access_key = "YOUR_ACCESS_KEY_ID"
  secret_key = "YOUR_SECRET_ACCESS_KEY"
}

resource "aws_instance" "my_first_server" {
  ami           = "ami-0866a3c8686eaeeba" # Ubuntu Server in us-east-1
  instance_type = "t2.micro"
}
```

- Replace `"YOUR_ACCESS_KEY_ID"` and `"YOUR_SECRET_ACCESS_KEY"` with your actual AWS credentials.
- Ensure the AMI ID corresponds to an existing AMI in your chosen region. You can verify the latest AMI ID on the [AWS EC2 AMI page](https://console.aws.amazon.com/ec2/v2/home?#Images).

### **Step 2: Initialize Terraform (No Need)**

In your terminal, run:

```shell
terraform init
```

- **What It Does**: Initializes the Terraform working directory and downloads the AWS provider plugin if not already present.

### **Step 3: Apply the Configuration**

Run:

```shell
terraform apply
```

- **Terraform Output**:
  - **Plan Summary**: Terraform will display a plan showing that it will create 1 resource.
  - **Confirmation Prompt**: You'll be asked to confirm by typing `yes`.

**Example Output**:

```plaintext
An execution plan has been generated and is shown below.
Resource actions are indicated with the following symbols:
  + create

Terraform will perform the following actions:

  # aws_instance.my_first_server will be created
  + resource "aws_instance" "my_first_server" {
      + ami = "ami-0866a3c8686eaeeba"
      # ... other attributes ...
    }

Plan: 1 to add, 0 to change, 0 to destroy.

Do you want to perform these actions?
  Terraform will perform the actions described above.
  Only 'yes' will be accepted to approve.

  Enter a value:
```

- Type **`yes`** and press **Enter**.

- **Terraform Actions**:
  - **Creating Resource**: Terraform will create the EC2 instance.
  - **State Update**: Updates the state file to include the new resource.

**Completion Message**:

```plaintext
aws_instance.my_first_server: Creation complete after 42s [id=i-0abcd1234efgh5678]

Apply complete! Resources: 1 added, 0 changed, 0 destroyed.
```

- **Note**: The `[id=...]` will show the actual instance ID assigned by AWS.

---

## 3. Adding Tags to Resources

Now that we've re-created the EC2 instance, let's modify it by adding tags. Tags are key-value pairs that help you identify and organize your AWS resources.

### **Step 1: Update the Terraform Configuration**

Modify your `main.tf` file to include the `tags` block:

```hcl
resource "aws_instance" "my_first_server" {
  ami           = "ami-0866a3c8686eaeeba"
  instance_type = "t2.micro"

  tags = {
    Name        = "MyUbuntuServer"
    Environment = "Development"
  }
}
```

- **Explanation**:
  - The `tags` argument accepts a map of key-value pairs.
  - We've added two tags:
    - **`Name`**: The common tag AWS uses to label resources.
    - **`Environment`**: A custom tag to indicate the environment type.

### **Step 2: Understand What Terraform Will Do**

Before applying changes, let's see what Terraform plans to do.

#### **Run the Plan Command**

```shell
terraform plan
```

#### **Interpret the Output**

- **Resource Changes**:
  - Terraform will **update** the existing EC2 instance to include the new tags.
- **Change Symbol**: The output will show a `~` (tilde) symbol indicating in-place modification.

**Example Output**:

```plaintext
An execution plan has been generated and is shown below.
Resource actions are indicated with the following symbols:
  ~ update in-place

Terraform will perform the following actions:

  # aws_instance.my_first_server will be updated in-place
  ~ resource "aws_instance" "my_first_server" {
      ~ tags = {} -> {
          + "Environment" = "Development"
          + "Name"        = "MyUbuntuServer"
        }
      # ... other attributes ...
    }

Plan: 0 to add, 1 to change, 0 to destroy.
```

- **Highlighted Sections**:
  - **`~ update in-place`**: Indicates that the resource will be modified without replacement.
  - **`tags = {} -> {...}`**: Shows that tags are being added from an empty set `{}` to the new tags.

### **Step 3: Apply the Changes**

Run:

```shell
terraform apply
```

- Review the plan output and confirm by typing `yes`.

**Example Confirmation**:

```plaintext
Plan: 0 to add, 1 to change, 0 to destroy.

Do you want to perform these actions?
  Terraform will perform the actions described above.
  Only 'yes' will be accepted to approve.

  Enter a value: yes
```

- **Terraform Actions**:
  - **Updating Resource**: Modifies the EC2 instance to include the new tags.
  - **State Update**: Updates the state file to reflect the changes.

**Completion Message**:

```plaintext
aws_instance.my_first_server: Modifications complete after 12s [id=i-0abcd1234efgh5678]

Apply complete! Resources: 0 added, 1 changed, 0 destroyed.
```

---

## 4. Understanding the Terraform Plan and Apply Outputs

### **Interpreting the Plan Output**

- **Symbols Used**:
  - **`+` (Plus)**: Resource will be created.
  - **`-` (Minus)**: Resource will be destroyed.
  - **`~` (Tilde)**: Resource will be updated in-place.
  - **`-/+`**: Resource will be destroyed and recreated.

- **Color Coding**:
  - **Green**: Additions (`+`).
  - **Red**: Deletions (`-`).
  - **Yellow/Orange**: Modifications (`~`).

- **Example**:

  ```plaintext
  # aws_instance.my_first_server will be updated in-place
  ~ resource "aws_instance" "my_first_server" {
      ~ tags = {} -> {
          + "Environment" = "Development"
          + "Name"        = "MyUbuntuServer"
        }
      # ... other attributes ...
    }
  ```

  - **`~ tags = {} -> {...}`**: Indicates that the `tags` attribute is changing from an empty map `{}` to the new tags.

### **Understanding Resource Dependencies**

- Terraform determines the order of operations based on resource dependencies.
- In this case, since we're updating an attribute of an existing resource, Terraform knows it can modify it in-place.

### **State Refresh**

- Before planning or applying changes, Terraform refreshes its state by querying AWS to get the current state of resources.
- **Output Snippet**:

  ```plaintext
  aws_instance.my_first_server: Refreshing state... [id=i-0abcd1234efgh5678]
  ```

---

## 5. Verifying Changes in the AWS Console

After applying the changes, let's confirm that the tags have been added to the EC2 instance.

### **Step 1: Navigate to EC2 Instances**

- Log in to the [AWS Management Console](https://console.aws.amazon.com/).
- Go to **Services** > **EC2** > **Instances**.

### **Step 2: Locate Your Instance**

- Find the EC2 instance named **`MyUbuntuServer`**.
  - The **Name** column should display this value due to the `Name` tag.

### **Step 3: View Instance Details**

- Click on the instance to view its details.
- **Tags Tab**:
  - Scroll down to the **Tags** section or select the **Tags** tab.
  - You should see the tags you specified:

| Key         | Value          |
|-------------|----------------|
| Name        | MyUbuntuServer |
| Environment | Development    |


### **Step 4: Observe the Impact**

- **Filtering Instances**:
  - You can filter instances based on tags.
  - For example, filter by `Environment = Development` to see all development instances.

---

## Additional Modifications

### **Removing Tags**

If you decide to remove the tags, you can modify the configuration accordingly.

#### **Step 1: Update the Configuration**

Comment out or remove the `tags` block:

```hcl
resource "aws_instance" "my_first_server" {
  ami           = "ami-0866a3c8686eaeeba"
  instance_type = "t2.micro"

  # tags = {
  #   Name        = "MyUbuntuServer"
  #   Environment = "Development"
  # }
}
```

#### **Step 2: Plan and Apply Changes**

Run:

```shell
terraform plan
```

- **Expected Output**:

  ```plaintext
  Terraform will perform the following actions:
  
  # aws_instance.my-first-server will be updated in-place
  ~ resource "aws_instance" "my-first-server" {
        id                                   = "i-0515c7a365618028a"
      ~ tags                                 = {
          - "Environment" = "Development" -> null
          - "Name"        = "MyUbuntuServer" -> null
        }
      ~ tags_all                             = {
          - "Environment" = "Development" -> null
          - "Name"        = "MyUbuntuServer" -> null
        }
        # (38 unchanged attributes hidden)
  
        # (8 unchanged blocks hidden)
    }
  
  Plan: 0 to add, 1 to change, 0 to destroy.
  ```

- **Explanation**:
  - The tags are being removed (`tags` changing from the current values to `null`).

Apply the changes:

```shell
terraform apply
```

- Confirm by typing `yes`.

#### **Step 3: Verify in AWS Console**

- Refresh the EC2 Instances page.
- The **Name** column should now be empty for your instance.
- In the **Tags** section, the tags should be removed.

---

## Important Concepts to Remember

### **Terraform State**

- Terraform maintains a **state file** that maps your configuration to real-world resources.
- **State Refresh**:
  - Before planning or applying, Terraform refreshes the state to ensure it has the latest information.

### **Declarative Approach**

- **Desired State**: You define the desired end state in your configuration.
- **Idempotent Operations**: Running `terraform apply` multiple times without changes will result in no action because the actual state matches the desired state.

### **Resource Identification**

- **Resource Names**:
  - The name you give in Terraform (e.g., `my_first_server`) is for internal reference.
  - AWS does not see this name unless it's specified as a tag.

### **Interpreting Terraform Output**

- **No Changes**:
  - When Terraform reports `No changes. Infrastructure is up-to-date.`, it means the actual state matches your configuration.
- **Resource Actions Summary**:
  - **`Plan: X to add, Y to change, Z to destroy.`** gives you a quick overview of what will happen.

---

## Conclusion

By following this section, you've learned how to:

- Re-create an EC2 instance using Terraform.
- Modify existing resources by adding or removing tags.
- Understand Terraform's declarative approach and how it manages state.
- Interpret Terraform's plan and apply outputs to anticipate changes.
- Verify changes in the AWS Management Console.

---

## Next Steps

With these concepts in mind, you can:

- Experiment with other resource attributes to see how Terraform handles different types of changes.
- Learn about **variables** to make your configurations more dynamic.
- Explore **resource dependencies** and how Terraform determines the order of operations.
- Understand **state management** and how to handle state files securely.

Remember, always cross-check the official [Terraform AWS Provider Documentation](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/instance) when working with resources to ensure you're using the correct arguments and understanding the resource's behavior.

---

**Happy Terraforming!**
#terraform
