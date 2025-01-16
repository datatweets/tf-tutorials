# Part 9: Comprehensive Guide to Terraform Variables

In this guide, we'll explore how to use variables in Terraform to make your configurations more flexible and reusable. We'll cover:

- **Defining Variables**
- **Assigning Values to Variables**
- **Variable Types and Constraints**
- **Complex Variable Types (Lists, Maps, Objects)**
- **Best Practices**

We'll cross-check and extend our explanations based on the official Terraform documentation: [Terraform Input Variables](https://developer.hashicorp.com/terraform/language/values/variables).

---

## Why Use Variables in Terraform?

Variables in Terraform allow you to:

- **Reuse Values**: Avoid repeating the same values throughout your code.
- **Parameterize Configurations**: Make your code flexible and adaptable to different environments.
- **Simplify Changes**: Update values in one place without modifying multiple resources.

---

## Defining Variables

Variables are defined using the `variable` block in your `.tf` files.

### Syntax

```hcl
variable "<VARIABLE_NAME>" {
  description = "<DESCRIPTION>"
  type        = <TYPE>
  default     = <DEFAULT_VALUE>
}
```

- **`<VARIABLE_NAME>`**: The name you assign to the variable.
- **`description`** *(optional)*: A description of the variable.
- **`type`** *(optional)*: The data type of the variable.
- **`default`** *(optional)*: A default value if none is provided.

### Example

Let's define a variable for a subnet CIDR block:

```hcl
variable "subnet_prefix" {
  description = "The CIDR block for the subnet"
  type        = string
  default     = "10.0.1.0/24"
}
```

---

## Using Variables in Configuration

To reference a variable in your configuration, use the `var` namespace:

```hcl
resource "aws_subnet" "subnet" {
  vpc_id     = aws_vpc.prod-vpc.id
  cidr_block = var.subnet_prefix

  tags = {
    Name = "prod-subnet"
  }
}
```

---

## Assigning Values to Variables

There are several ways to assign values to variables:

1. **Default Values**: Set in the variable definition.
2. **Command-Line Flags**: Use `-var` or `-var-file`.
3. **Environment Variables**: Set variables using environment variables.
4. **Variable Definitions Files**: Use `terraform.tfvars` or any `*.tfvars` file.

### 1. Using Default Values

If a variable has a default value, Terraform uses it unless overridden.

**Example**:

```hcl
variable "instance_type" {
  description = "Type of EC2 instance"
  type        = string
  default     = "t2.micro"
}
```

### 2. Command-Line Flags

**Using `-var` Flag**:

```shell
terraform apply -var="subnet_prefix=10.0.100.0/24"
```

**Using `-var-file` Flag**:

Suppose you have a file named `custom.tfvars`:

```hcl
subnet_prefix = "10.0.200.0/24"
```

Run Terraform with:

```shell
terraform apply -var-file="custom.tfvars"
```

### 3. Environment Variables

Set environment variables with the prefix `TF_VAR_`.

**Example**:

```shell
export TF_VAR_subnet_prefix="10.0.150.0/24"
```

Terraform will automatically use this value.

### 4. Variable Definitions Files

Create a file named `terraform.tfvars` or `*.auto.tfvars`.

**`terraform.tfvars`**:

```hcl
subnet_prefix = "10.0.250.0/24"
```

Terraform automatically loads this file.

---

## Variable Type Constraints

Specifying the type of a variable ensures that users provide values in the expected format.

### Basic Types

- **string**
- **number**
- **bool**

**Example**:

```hcl
variable "instance_count" {
  description = "Number of instances"
  type        = number
  default     = 1
}
```

### Complex Types

- **list(type)**
- **set(type)**
- **map(type)**
- **object({ attribute_name = type, ... })**
- **tuple([type, ...])**

---

## Using Lists and Maps

### Lists

A list is an ordered collection of values of the same type.

**Defining a List Variable**:

```hcl
variable "availability_zones" {
  description = "List of availability zones"
  type        = list(string)
  default     = ["us-east-1a", "us-east-1b"]
}
```

**Referencing List Elements**:

```hcl
resource "aws_subnet" "subnet" {
  vpc_id            = aws_vpc.prod-vpc.id
  cidr_block        = var.subnet_cidrs[0]
  availability_zone = var.availability_zones[0]

  tags = {
    Name = "subnet-1"
  }
}
```

### Maps

A map is a collection of key-value pairs.

**Defining a Map Variable**:

```hcl
variable "subnet_cidrs" {
  description = "Map of subnets with their CIDR blocks"
  type        = map(string)
  default     = {
    "subnet-1" = "10.0.1.0/24"
    "subnet-2" = "10.0.2.0/24"
  }
}
```

**Referencing Map Elements**:

```hcl
resource "aws_subnet" "subnet" {
  vpc_id     = aws_vpc.prod-vpc.id
  cidr_block = var.subnet_cidrs["subnet-1"]

  tags = {
    Name = "subnet-1"
  }
}
```

---

## Using Objects and Tuples

### Objects

Objects allow you to define a complex variable with multiple attributes.

**Defining an Object Variable**:

```hcl
variable "subnets" {
  description = "List of subnets with properties"
  type = list(object({
    name       = string
    cidr_block = string
  }))
}

# In your `terraform.tfvars` or `*.tfvars` file:

subnets = [
  {
    name       = "prod-subnet"
    cidr_block = "10.0.1.0/24"
  },
  {
    name       = "dev-subnet"
    cidr_block = "10.0.2.0/24"
  }
]
```

**Using the Object Variable**:

```hcl
resource "aws_subnet" "subnet" {
  count           = length(var.subnets)
  vpc_id          = aws_vpc.prod-vpc.id
  cidr_block      = var.subnets[count.index].cidr_block

  tags = {
    Name = var.subnets[count.index].name
  }
}
```

### Tuples

Tuples are similar to lists but can contain elements of different types.

**Defining a Tuple Variable**:

```hcl
variable "misc_values" {
  description = "A tuple of mixed types"
  type        = tuple([string, number, bool])
}

# In your `terraform.tfvars`:

misc_values = ["example", 42, true]
```

---

## Validating Variables

You can add custom validation rules to variables.

**Example**:

```hcl
variable "instance_type" {
  type = string

  validation {
    condition     = contains(["t2.micro", "t2.small"], var.instance_type)
    error_message = "Instance type must be t2.micro or t2.small."
  }
}
```

---

## Best Practices

- **Use Descriptive Names**: Make variable names meaningful.
- **Provide Descriptions**: Always add descriptions to variables.
- **Set Default Values When Appropriate**: Use defaults for common values.
- **Validate Inputs**: Use validation blocks for custom checks.
- **Separate Variable Definitions and Assignments**: Define variables in `.tf` files and assign values in `.tfvars` files.
- **Avoid Hardcoding Sensitive Information**: Do not hardcode passwords or secrets; consider using environment variables or secret management tools.
- **Use Type Constraints**: Specify variable types to catch errors early.
- **Organize Variables**: Group related variables together, possibly in separate files for large configurations.

---

## Putting It All Together: Example

### `variables.tf`

```hcl
variable "vpc_cidr" {
  description = "CIDR block for the VPC"
  type        = string
  default     = "10.0.0.0/16"
}

variable "subnets" {
  description = "List of subnet configurations"
  type = list(object({
    name            = string
    cidr_block      = string
    availability_zone = string
  }))
}

variable "instance_type" {
  description = "EC2 instance type"
  type        = string
  default     = "t2.micro"
}

variable "instance_count" {
  description = "Number of instances to launch"
  type        = number
  default     = 1
}
```

### `terraform.tfvars`

```hcl
subnets = [
  {
    name             = "public-subnet-1"
    cidr_block       = "10.0.1.0/24"
    availability_zone = "us-east-1a"
  },
  {
    name             = "public-subnet-2"
    cidr_block       = "10.0.2.0/24"
    availability_zone = "us-east-1b"
  }
]
```

### `main.tf`

```hcl
resource "aws_vpc" "main" {
  cidr_block = var.vpc_cidr

  tags = {
    Name = "main-vpc"
  }
}

resource "aws_subnet" "public" {
  count = length(var.subnets)

  vpc_id            = aws_vpc.main.id
  cidr_block        = var.subnets[count.index].cidr_block
  availability_zone = var.subnets[count.index].availability_zone

  tags = {
    Name = var.subnets[count.index].name
  }
}

resource "aws_instance" "web" {
  count         = var.instance_count
  ami           = "ami-085925f297f89fce1"
  instance_type = var.instance_type
  subnet_id     = aws_subnet.public[0].id

  tags = {
    Name = "web-server-${count.index}"
  }
}
```

---

## Additional Notes

- **Variable Precedence**: Values are assigned to variables in the following order (from lowest to highest precedence):
  1. Default values in variable definitions
  2. Values from `terraform.tfvars` and `*.auto.tfvars`
  3. Environment variables
  4. The `-var` and `-var-file` command-line options
- **Sensitive Variables**: Mark variables as sensitive to prevent Terraform from displaying their values.
  
  ```hcl
  variable "db_password" {
    type      = string
    sensitive = true
  }
  ```

- **Dynamic Blocks**: Use dynamic blocks to generate nested blocks based on variable values.

---

## Cross-Checking with Official Documentation

Based on the [Terraform Input Variables Documentation](https://developer.hashicorp.com/terraform/language/values/variables), we've covered:

- **Defining Variables**: Using the `variable` block.
- **Variable Types**: Including basic and complex types.
- **Assigning Values**: Through defaults, `.tfvars` files, environment variables, and command-line flags.
- **Variable Validation**: Using `validation` blocks.
- **Best Practices**: Including descriptions, type constraints, and avoiding hardcoding sensitive data.

---

## Conclusion

Variables in Terraform are powerful tools that enhance the flexibility and reusability of your configurations. By properly defining, assigning, and using variables, you can create scalable and maintainable infrastructure code.

---


#terraform
