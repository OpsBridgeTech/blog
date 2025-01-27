
# Writing Terraform Modules: Best Practices and Example

Terraform has become an essential tool for managing infrastructure as code. By using Terraform modules, you can package reusable pieces of infrastructure, making your configurations more maintainable, consistent, and scalable. Here's how to approach writing Terraform modules, along with some best practices and an example.

**What Are Terraform Modules?**

[Terraform modules](https://developer.hashicorp.com/terraform/language/modules) are containers for multiple resources that are used together. A module can be used to create lightweight abstractions, so that you can describe your infrastructure in terms of its architecture, rather than getting bogged down in resource-specific configurations.

## Best Practices for Writing Terraform Modules

1. Modularity and Reusability:
    - Single Responsibility: Each module should handle one specific task or component of your infrastructure. This makes them easier to maintain and reuse.
    - Parameterize Variables: Use variables for any value that might change across different environments or deployments. This includes names, sizes, or configurations.
2. Version Control:
    - Store your modules in version control systems like Git. This helps in tracking changes, collaborating, and using different versions of your modules.
3. Documentation:
    - Write clear documentation for each module. Describe inputs (variables), outputs, and usage examples. Use README files or inline comments where necessary.
4. Input Validation:
    - Use `validation` blocks for variables to ensure inputs meet certain criteria before Terraform even starts planning.

    ```hcl
    variable "instance_type" {
    type = string
    validation {
        condition     = contains(["t2.micro", "t3.micro"], var.instance_type)
        error_message = "The instance_type value must be one of t2.micro, t3.micro."
        }
    }
    ```

5. Output Values:
    - Clearly define outputs for anything other modules or the root module might need to consume. This includes IDs, endpoints, or any other relevant data.
6. Local Testing:
    - Before publishing, test your modules locally using `terraform init`, `apply`, and `destroy`. Use tools like `terraform validate` to check for syntax errors.
7. Security:
    - Avoid hardcoding sensitive information. Use Terraform's built-in support for secret management, or integrate with external vault solutions like HashiCorp Vault.
8. Keep It Up-to-Date:
    - Regularly update your modules to remain compatible with the latest versions of Terraform and underlying services.

## Example: Writing and Using a Simple VPC Module

**Step 1**: Write the Module

Create a directory structure for your module:

```test
modules/
└── vpc/
    ├── main.tf
    ├── variables.tf
    └── outputs.tf
```

- main.tf: Define your [resources](https://developer.hashicorp.com/terraform/language/resources).

```hcl
resource "aws_vpc" "main" {
  cidr_block = var.cidr_block
  tags = {
    Name = var.name
  }
}

resource "aws_subnet" "public" {
  count             = length(var.public_subnets)
  vpc_id            = aws_vpc.main.id
  cidr_block        = var.public_subnets[count.index]
  availability_zone = var.azs[count.index]
  tags = {
    Name = "${var.name}-public-${count.index + 1}"
  }
}
```

- variables.tf: Define your [variables](https://developer.hashicorp.com/terraform/language/values/variables).

```hcl
variable "cidr_block" {
  description = "The CIDR block for the VPC"
  type        = string
}

variable "name" {
  description = "Name to be used on all the resources as identifier"
  type        = string
}

variable "public_subnets" {
  description = "A list of public subnets inside the VPC"
  type        = list(string)
}

variable "azs" {
  description = "A list of availability zones names or ids in the region"
  type        = list(string)
}
```

- outputs.tf: Define what [outputs](https://developer.hashicorp.com/terraform/language/values/outputs) are available.

```hcl
output "vpc_id" {
  description = "The ID of the VPC"
  value       = aws_vpc.main.id
}

output "public_subnet_ids" {
  description = "List of IDs of public subnets"
  value       = aws_subnet.public[*].id
}
```

**Step 2:** Use the Module

In your main Terraform configuration:

```hcl
module "vpc" {
  source = "./modules/vpc"

  cidr_block    = "10.0.0.0/16"
  name          = "my-vpc"
  public_subnets = ["10.0.1.0/24", "10.0.2.0/24"]
  azs           = ["us-west-1a", "us-west-1b"]
}
```

**Step 3:** Test and Apply

Navigate to your project directory and run:

```sh
terraform init
terraform plan
terraform apply
```

By following these practices, you can create robust, reusable Terraform modules that not only streamline your infrastructure management but also make your code easier to understand and maintain. Remember, the key to successful module usage is in planning and clear documentation.
