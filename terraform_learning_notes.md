# 1: Introduction to Terraform

## Topics Covered
- Understanding Infrastructure as Code (IaC)
- Why we need IaC
- What is Terraform and its benefits
- Challenges with the traditional approach
- Terraform Workflow
- Installing Terraform

## Key Learning Points

### What is Infrastructure as Code?
Provisioning your infrastructure through code instead of manual processes.

### Why Infrastructure as Code?
- **Consistency**: Identical environments across dev, staging, and production
- **Time Efficiency**: Automated provisioning saves hours of manual work
- **Cost Management**: Easy to track costs and automate cleanup
- **Scalability**: Deploy to hundreds of servers with same effort as one
- **Version Control**: Track changes in Git
- **Reduced Human Error**: Eliminate manual configuration mistakes
- **Collaboration**: Team can work together on infrastructure
- **Testing**: Test before actually provisioning the infra.

### Benefits of IaC
- Consistent environment deployment
- Easy to track and manage costs
- Write once, deploy many (single codebase)
- Time-saving automation
- Reduced human error
- Cost optimization through automation
- Version control for infrastructure changes
- Automated cleanup and scheduled destruction
- Developer focus on application development
- Easy creation of identical production environments for troubleshooting

### What is Terraform?
Infrastructure as Code tool that helps automate infrastructure provisioning and management across multiple cloud providers.

### How Terraform Works
Write Terraform files → Run Terraform commands → Call AWS APIs through Terraform Provider

**Terraform Workflow Phases:**
1. `terraform init` - Initialize the working directory
2. `terraform validate` - Validate the configuration files
3. `terraform plan` - Create an execution plan
4. `terraform apply` - Apply the changes to reach desired state
5. `terraform destroy` - Destroy the infrastructure when needed

## Tasks for Practice

### Install Terraform
Follow the installation guide: https://developer.hashicorp.com/terraform/install

or 

### Common Installation Commands
```bash
# For macOS
brew install hashicorp/tap/terraform

# For Ubuntu/Debian
wget -O- https://apt.releases.hashicorp.com/gpg | sudo gpg --dearmor -o /usr/share/keyrings/hashicorp-archive-keyring.gpg
echo "deb [signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] https://apt.releases.hashicorp.com $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/hashicorp.list
sudo apt update && sudo apt install terraform
```

### Setup Commands
```bash
terraform -install-autocomplete
alias tf=terraform
terraform -version
```

### Common Installation Error (macOS)
If you encounter:
```
Error: No developer tools installed.
```
Install Command Line Tools:
```bash
xcode-select --install
```

### # Day 01 - Quick Tasks

## What to Do

- [ ] Watch the Day 01 video and take notes

## Tips

- Keep it simple and clear, feel free to take AI assistance but do not overuse it
- Use your own diagrams

---
---
---
---

# 2: Terraform Provider

## Topics Covered
- Terraform Providers
- Provider version vs Terraform core version
- Why version matters
- Version constraints
- Operators for versions

## Key Learning Points

### What are Terraform Providers?
Providers are plugins that allow Terraform to interact with cloud platforms, SaaS providers, and other APIs. For AWS, we use the `hashicorp/aws` provider.

### Provider vs Terraform Core Version
- **Terraform Core**: The main Terraform binary that parses configuration and manages state
- **Provider Version**: Individual plugins that communicate with specific APIs (AWS, Azure, Google Cloud, etc.)
- They have independent versioning and release cycles

### Why Version Matters
- **Compatibility**: Ensure provider works with your Terraform version
- **Stability**: Pin to specific versions to avoid breaking changes
- **Features**: New provider versions add support for new AWS services
- **Bug Fixes**: Updates often include important security and bug fixes
- **Reproducibility**: Same versions ensure consistent behavior across environments

### Version Constraints
Use version constraints to specify acceptable provider versions:

- `= 1.2.3` - Exact version
- `>= 1.2` - Greater than or equal to
- `<= 1.2` - Less than or equal to
- `~> 1.2` - Pessimistic constraint (allow patch releases) {Mostly used in production} 
- `>= 1.2, < 2.0` - Range constraint

### Best Practices
1. Always specify provider versions
2. Use pessimistic constraints for stability
3. Test provider upgrades in development first
4. Document version requirements in your README
5. Use terraform providers lock command for consistency

## Configuration Examples

### Basic Provider Configuration
```hcl
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}

provider "aws" {
  region = "us-east-1"
}
```

### Multiple Provider Versions
```hcl
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
    random = {
      source  = "hashicorp/random"
      version = "~> 3.1"
    }
  }
}
```

---
---
---
---

# 3. Terraform State - State File Management - Remote Backend

## Topics Covered
- How Terraform updates Infrastructure
- Terraform state file
- State file best practices
- Remote backend setup with S3
- S3 Native State Locking (No DynamoDB required)
- State management

## Key Learning Points

### How Terraform Updates Infrastructure
- **Goal**: Keep actual state same as desired state
- **State File**: Actual state resides in terraform.tfstate file
- **Process**: Terraform compares current state(terraform.tfstate file) with desired configuration
- **Updates**: Only changes the resources that need modification

### Terraform State File
The state file is a JSON file that contains:
- Resource metadata and current configuration
- Resource dependencies
- Provider information
- Resource attribute values

### State File Best Practices
1. **Never edit state file manually**
2. **Store state file remotely** (not in local file system)
3. **Enable state locking** to prevent concurrent modifications
4. **Backup state files** regularly
5. **Use separate state files** for different environments
6. **Restrict access** to state files (contains sensitive data)
7. **Encrypt state files** at rest and in transit

### Remote Backend Benefits
- **Collaboration**: Team members can share state
- **Locking**: Prevents concurrent state modifications
- **Security**: Encrypted storage and access control
- **Backup**: Automatic versioning and backup
- **Durability**: Highly available storage

### AWS Remote Backend Components

- **S3 Bucket**: Stores the state file
- **S3 Native State Locking**: Uses S3 conditional writes for locking (introduced in Terraform 1.10)
- **IAM Policies**: Control access to backend resources

## S3 Native State Locking

### What is S3 Native State Locking?

Starting with **Terraform 1.10** (released in 2024), you no longer need DynamoDB for state locking. Terraform now supports **S3 native state locking** using Amazon S3's **Conditional Writes** feature.

### How It Works

S3 native state locking uses the **If-None-Match** HTTP header to implement atomic operations:

1. When Terraform needs to acquire a lock, it attempts to create a lock file in S3
2. S3 conditional writes check if the lock file already exists
3. If the lock file exists, the write operation fails, preventing concurrent modifications
4. If the lock file doesn't exist, it's created successfully and the lock is acquired
5. When the operation completes, the lock file is deleted (appears as a delete marker with versioning)


**Previous Method (DynamoDB):**
- Required separate DynamoDB table creation
- Additional AWS service to monitor and maintain
- More complex IAM permissions
- Extra cost for DynamoDB read/write operations
- DynamoDB state locking is now **discouraged** and may be deprecated in future Terraform versions



## Tasks for Practice

### Setup Remote Backend

#### Step 1: Create S3 Bucket for State Storage

Create an S3 bucket with versioning and encryption enabled to store Terraform state files.You can use the test.sh script provided in the code folder to do it quickly using AWS CLI.



### Configuration Example

```hcl
terraform {
  backend "s3" {
    bucket       = "your-terraform-state-bucket"
    key          = "dev/terraform.tfstate"
    region       = "us-east-1"
    use_lockfile = true
    encrypt      = true
  }
}
```

**Key Parameters:**
- `bucket`: S3 bucket name for state storage
- `key`: Path within the bucket where state file will be stored
- `region`: AWS region for the S3 bucket
- `use_lockfile`: Enable S3 native state locking (set to `true`)
- `encrypt`: Enable server-side encryption for the state file



**Important:** S3 versioning MUST be enabled for S3 native state locking to work properly.



### How to Test State Locking

To verify that S3 native state locking is working:

1. **Terminal 1**: Run `terraform apply`
2. **Terminal 2**: While the first is running, try `terraform plan` or `terraform apply`
3. **Expected Result**: The second command should fail with an error like:
   ```
   Error: Error acquiring the state lock
   Error message: operation error S3: PutObject, https response error StatusCode: 412
   Lock Info:
     ID:        <lock-id>
     Path:      <bucket>/<key>
     Operation: OperationTypeApply
     Who:       <user>@<hostname>
   ```

4. **Check S3 Bucket**: During the operation, you'll see a `.tflock` file temporarily in your S3 bucket
5. **After Completion**: The lock file will be automatically deleted (delete marker with versioning)

### Backend Migration
```bash
# Initialize with new backend configuration
terraform init

# Terraform will prompt to migrate existing state
# Answer 'yes' to copy existing state to new backend

# Verify state is now remote
terraform state list
```

### State Commands
```bash
# List resources in state
terraform state list

# Show detailed state information
terraform state show <resource_name>

# Remove resource from state (without destroying)
terraform state rm <resource_name>

# Move resource to different state address
terraform state mv <source> <destination>

# Pull current state and display
terraform state pull
```

### Security Considerations

- **S3 Bucket Policy**: Restrict access to authorized users only
- **S3 Versioning**: Required for state locking; also provides rollback capability
- **Encryption**: Enable encryption for S3 bucket (server-side encryption)
- **Access Logging**: Enable CloudTrail for audit logging
- **IAM Permissions**: Grant minimal required S3 permissions (no DynamoDB permissions needed)

### Common Issues

- **State Lock Error**: If terraform process crashes, the lock file may remain. Manually delete it from S3 or use: `terraform force-unlock <lock-id>`
- **Permission Errors**: Ensure proper IAM permissions for S3 operations
- **Versioning Not Enabled**: S3 versioning MUST be enabled for native state locking to work
- **Region Mismatch**: Backend region should match your provider region
- **Bucket Names**: S3 bucket names must be globally unique
- **Terraform Version**: Requires Terraform 1.10+ for S3 native locking; 1.11+ recommended for stable GA release


---
---
---
---

# 4. Terraform Variables Demo

# A simple demo showing the three types of Terraform variables using a basic S3 bucket.

## 🎯 Three Types of Variables

### 1. **Input Variables** (`variables.tf`)
Values you provide to Terraform - like function parameters
```hcl
variable "environment" {
  description = "Environment name"
  type        = string
  default     = "staging"
}
```

### 2. **Local Variables** (`locals.tf`)
Internal computed values - like local variables in programming
```hcl
locals {
  common_tags = {
    Environment = var.environment
    Project     = "Terraform-Demo"
  }
  
  full_bucket_name = "${var.environment}-${var.bucket_name}-${random_string.suffix.result}"
}
```

### 3. **Output Variables** (`output.tf`)
Values returned after deployment - like function return values
```hcl
output "bucket_name" {
  description = "Name of the S3 bucket"
  value       = aws_s3_bucket.demo.bucket
}
```

## 📥 Understanding Input Variables in Detail

### What are Input Variables?
Input variables are like **function parameters** - they allow you to customize your Terraform configuration without hardcoding values.

### Basic Input Variable Structure
```hcl
variable "variable_name" {
  description = "What this variable is for"
  type        = string
  default     = "default_value"  # Optional
}
```

### How to Use Input Variables
```hcl
# Define in variables.tf
variable "environment" {
  description = "Environment name"
  type        = string
  default     = "staging"
}

variable "bucket_name" {
  description = "S3 bucket name"
  type        = string
  default     = "my-terraform-bucket"
}

# Reference with var. prefix in main.tf
resource "aws_s3_bucket" "demo" {
  bucket = var.bucket_name  # Using input variable
  
  tags = {
    Environment = var.environment  # Using input variable
  }
}
```

### Providing Values to Input Variables

**1. Default values** (in variables.tf)
```hcl
variable "environment" {
  default = "staging"
}
```

**2. terraform.tfvars file** (auto-loaded)
```hcl
environment = "demo"
bucket_name = "terraform-demo-bucket"
```

**3. Command line**
```bash
terraform plan -var="environment=production"
```

**4. Environment variables**
```bash
export TF_VAR_environment="development"
terraform plan
```

## 📤 Understanding Output Variables in Detail

### What are Output Variables?
Output variables are like **function return values** - they display important information after Terraform creates your infrastructure.

### Basic Output Variable Structure
```hcl
output "output_name" {
  description = "What this output shows"
  value       = resource.resource_name.attribute
}
```

### How to Use Output Variables

**Define in output.tf**
```hcl
# Output a resource attribute
output "bucket_name" {
  description = "Name of the S3 bucket"
  value       = aws_s3_bucket.demo.bucket
}

output "bucket_arn" {
  description = "ARN of the S3 bucket"
  value       = aws_s3_bucket.demo.arn
}

# Output an input variable (to confirm what was used)
output "environment" {
  description = "Environment from input variable"
  value       = var.environment
}

# Output a local variable (to see computed values)
output "tags" {
  description = "Tags from local variable"
  value       = local.common_tags
}
```

### Viewing Outputs

After running `terraform apply`, you can view outputs:

```bash
terraform output                    # Show all outputs
terraform output bucket_name        # Show specific output
terraform output -json              # Show all outputs in JSON format
```

**Example output:**
```
bucket_arn = "arn:aws:s3:::demo-terraform-demo-bucket-abc123"
bucket_name = "demo-terraform-demo-bucket-abc123"
environment = "demo"
tags = {
  "Environment" = "demo"
  "Owner" = "DevOps-Team"
  "Project" = "Terraform-Demo"
}
```

## 🏗️ What This Creates

Just one simple S3 bucket that demonstrates all three variable types:
- Uses **input variables** for environment and bucket name
- Uses **local variables** for computed bucket name and tags
- Uses **output variables** to show the created bucket details

## 🚀 Variable Precedence Testing

### 1. **Default Values** (temporarily hide terraform.tfvars)
```bash
mv terraform.tfvars terraform.tfvars.backup
terraform plan
# Uses: environment = "staging" (from variables.tf default)
mv terraform.tfvars.backup terraform.tfvars  # restore
```

### 2. **Using terraform.tfvars** (automatically loaded)
```bash
terraform plan
# Uses: environment = "demo" (from terraform.tfvars)
```

### 3. **Command Line Override** (highest precedence)
```bash
terraform plan -var="environment=production"
# Overrides tfvars: environment = "production"
```

### 4. **Environment Variables**
```bash
export TF_VAR_environment="staging-from-env"
terraform plan
# Uses environment variable (but command line still wins)
```

### 5. **Using Different tfvars Files**
```bash
terraform plan -var-file="dev.tfvars"        # environment = "development"
terraform plan -var-file="production.tfvars"  # environment = "production"
```
```

## 📁 Simple File Structure

```
├── main.tf           # S3 bucket resource
├── variables.tf      # Input variables (2 simple variables)
├── locals.tf         # Local variables (tags and computed name)
├── output.tf         # Output variables (bucket details)
├── provider.tf       # AWS provider
├── terraform.tfvars  # Default variable values
└── README.md         # This file
```

## 🧪 Practical Examples

### Example 1: Testing Different Input Values

```bash
# Test with defaults (temporarily hide terraform.tfvars)
mv terraform.tfvars terraform.tfvars.backup
terraform plan
# Shows: Environment = "staging", bucket will be "staging-my-terraform-bucket-xxxxx"

# Test with terraform.tfvars
mv terraform.tfvars.backup terraform.tfvars
terraform plan  
# Shows: Environment = "demo", bucket will be "demo-terraform-demo-bucket-xxxxx"

# Test with command line override
terraform plan -var="environment=test" -var="bucket_name=my-test-bucket"
# Shows: Environment = "test", bucket will be "test-my-test-bucket-xxxxx"
```

### Example 2: Viewing All Variable Types in Action

```bash
# Apply the configuration
terraform apply -auto-approve

# See all outputs (shows output variables)
terraform output
# bucket_arn = "arn:aws:s3:::demo-terraform-demo-bucket-abc123"
# bucket_name = "demo-terraform-demo-bucket-abc123"  
# environment = "demo"                                # (input variable)
# tags = {                                           # (local variable)
#   "Environment" = "demo"
#   "Owner" = "DevOps-Team"  
#   "Project" = "Terraform-Demo"
# }

# See how local variables computed the bucket name
echo "Input: environment = $(terraform output -raw environment)"
echo "Input: bucket_name = terraform-demo-bucket (from tfvars)"  
echo "Local: full_bucket_name = $(terraform output -raw bucket_name)"
echo "Random suffix was added by local variable!"
```

### Example 3: Variable Precedence in Action

```bash
# Start with terraform.tfvars (environment = "demo")
terraform plan | grep Environment
# Shows: "Environment" = "demo"

# Override with environment variable
export TF_VAR_environment="from-env-var"
terraform plan | grep Environment  
# Shows: "Environment" = "from-env-var"

# Override with command line (highest precedence)
terraform plan -var="environment=from-command-line" | grep Environment
# Shows: "Environment" = "from-command-line"

# Clean up
unset TF_VAR_environment
```

## 🔧 Try These Commands

```bash
# Initialize
terraform init

# Plan with defaults
terraform plan

# Plan with command line override
terraform plan -var="environment=test"

# Plan with different tfvars file
terraform plan -var-file="dev.tfvars"

# Apply and see outputs
terraform apply
terraform output

# Clean up
terraform destroy
```

## 💡 Key Takeaways

- **Input variables**: Parameterize your configuration
- **Local variables**: Compute and reuse values
- **Output variables**: Share results after deployment
- **Precedence**: Command line > tfvars > environment vars > defaults

---
---
---
---
# 6: File Structure for terraform project

## Topics Covered
- Terraform file organization
- Sequence of file loading
- Best practices for file structure
- Code organization patterns

## Key Learning Points

### Terraform File Loading
- Terraform loads all `.tf` files in the current directory
- Files are loaded in **lexicographical order** (alphabetical)
- File names don't affect functionality, only organization
- All `.tf` files are merged into a single configuration

### Recommended File Structure
```
project-root/
├── backend.tf           # Backend configuration
├── provider.tf          # Provider configurations
├── variables.tf         # Input variable definitions
├── locals.tf           # Local value definitions
├── main.tf             # Main resource definitions
├── vpc.tf              # VPC-related resources
├── security.tf         # Security groups, NACLs
├── compute.tf          # EC2, Auto Scaling, etc.
├── storage.tf          # S3, EBS, EFS resources
├── database.tf         # RDS, DynamoDB resources
├── outputs.tf          # Output definitions
├── terraform.tfvars   # Variable values
└── README.md           # Documentation
```

### File Organization Principles
1. **Separation of Concerns**: Group related resources together
2. **Logical Grouping**: Organize by service or function
3. **Consistent Naming**: Use clear, descriptive file names
4. **Modular Approach**: Keep files focused on specific areas
5. **Documentation**: Include README and comments

## Tasks for Practice

### Task: Reorganize Previous Files
Using the files from Day 5, divide the configuration into separate files:

#### backend.tf
```hcl
terraform {
  required_version = ">= 1.0"
  
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
    random = {
      source  = "hashicorp/random"
      version = "~> 3.1"
    }
  }

  backend "s3" {
    bucket         = "your-terraform-state-bucket"
    key            = "dev/terraform.tfstate"
    region         = "us-east-1"
    dynamodb_table = "terraform-state-lock"
    encrypt        = true
  }
}
```

#### provider.tf
```hcl
provider "aws" {
  region = var.region

  default_tags {
    tags = local.common_tags
  }
}
```

#### variables.tf
```hcl
variable "environment" {
  description = "Environment name (dev, staging, production)"
  type        = string
  default     = "staging"
  
  validation {
    condition     = contains(["dev", "staging", "production"], var.environment)
    error_message = "Environment must be dev, staging, or production."
  }
}

variable "region" {
  description = "AWS region for resources"
  type        = string
  default     = "us-east-1"
}

variable "project_name" {
  description = "Name of the project"
  type        = string
}

variable "vpc_cidr" {
  description = "CIDR block for VPC"
  type        = string
  default     = "10.0.0.0/16"
  
  validation {
    condition     = can(cidrhost(var.vpc_cidr, 0))
    error_message = "VPC CIDR must be a valid IPv4 CIDR block."
  }
}

variable "availability_zones" {
  description = "List of availability zones"
  type        = list(string)
  default     = ["us-east-1a", "us-east-1b"]
}

variable "tags" {
  description = "Additional tags to apply to resources"
  type        = map(string)
  default     = {}
}
```

#### locals.tf
```hcl
locals {
  # Common tags applied to all resources
  common_tags = merge(var.tags, {
    Environment   = var.environment
    Project       = var.project_name
    ManagedBy     = "Terraform"
    CreatedDate   = formatdate("YYYY-MM-DD", timestamp())
  })

  # Naming convention
  name_prefix = "${var.project_name}-${var.environment}"
  
  # Network configuration
  vpc_name = "${local.name_prefix}-vpc"
  
  # Storage configuration  
  bucket_name = "${local.name_prefix}-${random_id.bucket_suffix.hex}"
}

# Random suffix for globally unique names
resource "random_id" "bucket_suffix" {
  byte_length = 4
  
  keepers = {
    project     = var.project_name
    environment = var.environment
  }
}
```

#### vpc.tf
```hcl
# VPC
resource "aws_vpc" "main" {
  cidr_block           = var.vpc_cidr
  enable_dns_hostnames = true
  enable_dns_support   = true

  tags = merge(local.common_tags, {
    Name = local.vpc_name
  })
}

# Internet Gateway
resource "aws_internet_gateway" "main" {
  vpc_id = aws_vpc.main.id

  tags = merge(local.common_tags, {
    Name = "${local.name_prefix}-igw"
  })
}

# Public Subnets
resource "aws_subnet" "public" {
  count = length(var.availability_zones)

  vpc_id                  = aws_vpc.main.id
  cidr_block              = cidrsubnet(var.vpc_cidr, 8, count.index)
  availability_zone       = var.availability_zones[count.index]
  map_public_ip_on_launch = true

  tags = merge(local.common_tags, {
    Name = "${local.name_prefix}-public-subnet-${count.index + 1}"
    Type = "Public"
  })
}

# Route Table for Public Subnets
resource "aws_route_table" "public" {
  vpc_id = aws_vpc.main.id

  route {
    cidr_block = "0.0.0.0/0"
    gateway_id = aws_internet_gateway.main.id
  }

  tags = merge(local.common_tags, {
    Name = "${local.name_prefix}-public-rt"
  })
}

# Associate Route Table with Public Subnets
resource "aws_route_table_association" "public" {
  count = length(aws_subnet.public)

  subnet_id      = aws_subnet.public[count.index].id
  route_table_id = aws_route_table.public.id
}
```

#### storage.tf
```hcl
# S3 Bucket
resource "aws_s3_bucket" "main" {
  bucket = local.bucket_name

  tags = merge(local.common_tags, {
    Name        = local.bucket_name
    Purpose     = "General storage"
    Environment = var.environment
  })
}

# S3 Bucket Versioning
resource "aws_s3_bucket_versioning" "main" {
  bucket = aws_s3_bucket.main.id
  
  versioning_configuration {
    status = "Enabled"
  }
}

# S3 Bucket Server Side Encryption
resource "aws_s3_bucket_server_side_encryption_configuration" "main" {
  bucket = aws_s3_bucket.main.id

  rule {
    apply_server_side_encryption_by_default {
      sse_algorithm = "AES256"
    }
    bucket_key_enabled = true
  }
}

# S3 Bucket Public Access Block
resource "aws_s3_bucket_public_access_block" "main" {
  bucket = aws_s3_bucket.main.id

  block_public_acls       = true
  block_public_policy     = true
  ignore_public_acls      = true
  restrict_public_buckets = true
}
```

#### outputs.tf
```hcl
# VPC Outputs
output "vpc_id" {
  description = "ID of the VPC"
  value       = aws_vpc.main.id
}

output "vpc_cidr_block" {
  description = "CIDR block of the VPC"
  value       = aws_vpc.main.cidr_block
}

output "vpc_arn" {
  description = "ARN of the VPC"
  value       = aws_vpc.main.arn
}

# Subnet Outputs
output "public_subnet_ids" {
  description = "IDs of the public subnets"
  value       = aws_subnet.public[*].id
}

output "public_subnet_cidrs" {
  description = "CIDR blocks of the public subnets"
  value       = aws_subnet.public[*].cidr_block
}

# S3 Outputs
output "s3_bucket_name" {
  description = "Name of the S3 bucket"
  value       = aws_s3_bucket.main.bucket
}

output "s3_bucket_arn" {
  description = "ARN of the S3 bucket"
  value       = aws_s3_bucket.main.arn
}

output "s3_bucket_domain_name" {
  description = "Domain name of the S3 bucket"
  value       = aws_s3_bucket.main.bucket_domain_name
}

# Environment Outputs
output "environment" {
  description = "Environment name"
  value       = var.environment
}

output "region" {
  description = "AWS region"
  value       = var.region
}

output "common_tags" {
  description = "Common tags applied to resources"
  value       = local.common_tags
}
```

#### terraform.tfvars
```hcl
# Project Configuration
project_name = "aws-terraform-course"
environment  = "demo"
region       = "us-east-1"

# Network Configuration
vpc_cidr          = "10.0.0.0/16"
availability_zones = ["us-east-1a", "us-east-1b", "us-east-1c"]

# Tags
tags = {
  Owner       = "DevOps-Team"
  Department  = "Engineering"
  CostCenter  = "Engineering-001"
  Project     = "TerraformLearning"
}
```

### Advanced File Organization Patterns

#### Environment-Specific Structure
```
environments/
├── dev/
│   ├── backend.tf
│   ├── terraform.tfvars
│   └── main.tf
├── staging/
│   ├── backend.tf
│   ├── terraform.tfvars
│   └── main.tf
└── production/
    ├── backend.tf
    ├── terraform.tfvars
    └── main.tf

modules/
├── vpc/
├── security/
└── compute/

shared/
├── variables.tf
├── outputs.tf
└── locals.tf
```

#### Service-Based Structure
```
infrastructure/
├── networking/
│   ├── vpc.tf
│   ├── subnets.tf
│   └── routing.tf
├── security/
│   ├── security-groups.tf
│   ├── nacls.tf
│   └── iam.tf
├── compute/
│   ├── ec2.tf
│   ├── autoscaling.tf
│   └── load-balancers.tf
├── storage/
│   ├── s3.tf
│   ├── ebs.tf
│   └── efs.tf
└── data/
    ├── rds.tf
    ├── dynamodb.tf
    └── elasticache.tf
```

### Best Practices

1. **Consistent Naming**
   - Use clear, descriptive file names
   - Follow team conventions
   - Use lowercase with hyphens or underscores

2. **Logical Grouping**
   - Group related resources together
   - Separate by AWS service or function
   - Consider dependencies when organizing

3. **Size Management**
   - Keep files manageable (< 500 lines)
   - Split large files by functionality
   - Use modules for reusable components

4. **Dependencies**
   - Place provider and backend configs first
   - Define variables before using them
   - Output values at the end

5. **Documentation**
   - Include README.md
   - Comment complex configurations
   - Document variable purposes

### Commands for Testing
```bash
# Validate the reorganized structure
terraform validate

# Format all files consistently
terraform fmt -recursive

# Plan to ensure no changes
terraform plan

# Apply if everything looks good
terraform apply
```

### Common File Organization Mistakes

1. **Everything in main.tf** - Makes code hard to navigate
2. **Inconsistent naming** - Confuses team members
3. **Mixed concerns** - Resources that don't belong together
4. **No documentation** - Difficult for others to understand
5. **Overly complex structure** - Simple is often better

---
---
---
---
# 7: Type Constraints in Terraform

## Topics Covered
- String, number, bool types
- Map, set, list, tuple, object types
- Type validation and constraints
- Complex type definitions

## Key Learning Points

### Basic Types
1. **string** - Text values
2. **number** - Numeric values (integers and floats)
3. **bool** - Boolean values (true/false)

### Collection Types
1. **list(type)** - Ordered collection of values
2. **set(type)** - Unordered collection of unique values
3. **map(type)** - Key-value pairs with string keys
4. **tuple([type1, type2, ...])** - Ordered collection with specific types for each element
5. **object({key1=type1, key2=type2, ...})** - Structured data with named attributes


### Common Type Patterns

1. **Environment-specific configurations**
2. **Resource sizing based on type**
3. **Tag standardization**
4. **Network configuration validation**
5. **Security policy enforcement**

## Best Practices

1. **Always specify types** for variables
2. **Use validation blocks** for business rules
3. **Provide meaningful error messages**
4. **Use appropriate collection types** (list vs set vs map)
5. **Validate complex objects** thoroughly
6. **Use type conversion functions** when needed
7. **Document type requirements** in descriptions

---
---
---
---
# 08: Terraform Meta-Arguments - Complete Guide

## 📚 What You'll Learn

- Understanding all Terraform meta-arguments
- **count** - Create multiple resources with numeric indexing
- **for_each** - Create multiple resources with maps/sets
- **depends_on** - Explicit resource dependencies
- **lifecycle** - Control resource creation and destruction behavior
- **provider** - Use alternate provider configurations
- Output transformations with `for` expressions
- Best practices for each meta-argument

## 🎯 Key Concepts

### Meta-Arguments Overview

Meta-arguments are special arguments that can be used with **any resource type** to change the behavior of resources:

1. **count** - Create multiple resource instances based on a number
2. **for_each** - Create multiple resource instances based on a map or set
3. **depends_on** - Explicit resource dependencies
4. **lifecycle** - Customize resource lifecycle behavior
5. **provider** - Select a non-default provider configuration
6. **provisioner** - Execute scripts on resource creation/destruction (not recommended)

**This lesson includes simple examples for all meta-arguments!**

### COUNT Meta-Argument

```hcl
resource "aws_s3_bucket" "example" {
  count  = 3
  bucket = "my-bucket-${count.index}"
}
```

**Use cases:**
- Creating N identical resources
- Simple iteration over a list
- When numeric index is sufficient

**Limitations:**
- Removing items from the middle of a list causes resource recreation
- Less stable resource addressing
- Harder to maintain

### FOR_EACH Meta-Argument

```hcl
resource "aws_s3_bucket" "example" {
  for_each = toset(["bucket1", "bucket2", "bucket3"])
  bucket   = each.value
}
```

**Use cases:**
- Creating resources from a map or set
- Stable resource addressing by key
- Production environments
- Complex resource configurations

**Benefits:**
- Adding/removing items doesn't affect other resources
- More readable resource references
- Better for production use

### DEPENDS_ON Meta-Argument

```hcl
resource "aws_s3_bucket" "dependent" {
  bucket = "my-bucket"
  
  depends_on = [aws_s3_bucket.primary]
}
```

**Use cases:**
- Explicit resource ordering
- Hidden dependencies not captured by references
- Ensuring resources are created in specific order

### LIFECYCLE Meta-Argument

```hcl
resource "aws_s3_bucket" "example" {
  bucket = "my-bucket"
  
  lifecycle {
    prevent_destroy       = true  # Prevent accidental deletion
    create_before_destroy = true  # Create new before destroying old
    ignore_changes        = [tags] # Ignore changes to tags
  }
}
```

**Use cases:**
- Protect critical resources from deletion
- Zero-downtime updates
- Ignore external changes to specific attributes

### PROVIDER Meta-Argument

```hcl
resource "aws_s3_bucket" "example" {
  provider = aws.west  # Use alternate provider
  bucket   = "my-bucket"
}
```

**Use cases:**
- Multi-region deployments
- Multi-account setups
- Cross-region replication

## 🚀 Quick Start

### Prerequisites

- Terraform >= 1.9.0
- AWS CLI configured with appropriate credentials
- Basic understanding of Terraform syntax

### Steps

1. **Clone and navigate to the lesson folder:**
   ```bash
   cd lessons/day08
   ```

2. **Update variables (important!):**
   - Edit `variables.tf` or create a `terraform.tfvars` file
   - Change S3 bucket names to be globally unique
   - Update AWS region if needed

3. **Initialize Terraform:**
   ```bash
   terraform init
   ```

4. **Format your code:**
   ```bash
   terraform fmt
   ```

5. **Validate configuration:**
   ```bash
   terraform validate
   ```

6. **Review the execution plan:**
   ```bash
   terraform plan
   ```

7. **Apply (optional):**
   ```bash
   terraform apply
   ```

8. **View outputs:**
   ```bash
   terraform output
   ```

9. **Cleanup:**
   ```bash
   terraform destroy
   ```

## 📝 Examples Included

### 1. COUNT Meta-Argument
- Creates multiple S3 buckets using a list variable
- Demonstrates `count.index` usage
- Index-based resource addressing

### 2. FOR_EACH Meta-Argument (Set)
- Creates S3 buckets using a set variable
- Demonstrates `each.key` and `each.value`
- More stable resource addressing

### 3. DEPENDS_ON Meta-Argument
- Shows explicit resource dependencies
- Primary and dependent bucket example
- Control resource creation order

### 4. LIFECYCLE Meta-Argument
- Demonstrates `prevent_destroy`, `create_before_destroy`, `ignore_changes`
- Protects critical resources
- Handles zero-downtime updates

### 5. PROVIDER Meta-Argument
- Shows how to use alternate providers (commented example)
- Multi-region deployment pattern
- Provider aliasing

### 6. Advanced Outputs
- Splat expressions (`[*]`)
- For loops in outputs
- Map transformations
- Combined outputs

## 🎓 Learning Path

1. **Start with Task 1-3** in `task.md` to understand the basics
2. **Practice with Task 4-5** to create your own resources
3. **Master outputs with Task 6**
4. **Deep dive with Task 7** to understand count vs for_each differences
5. **Apply knowledge with Task 8** for a real-world scenario

## ⚠️ Important Notes

### S3 Bucket Names
- S3 bucket names must be **globally unique** across all AWS accounts
- Update the default bucket names in `variables.tf` before applying
- Use your organization prefix or a unique identifier

### Backend Configuration
- The `backend.tf` uses S3 for remote state
- Comment out the backend block if you want to use local state
- Create the S3 bucket manually before running `terraform init`

### Costs
- Most resources in this lesson are free tier eligible
- S3 buckets incur minimal storage costs
- IAM users are free
- **Always run `terraform destroy` when done!**

## 🔍 Key Differences: COUNT vs FOR_EACH

| Feature | COUNT | FOR_EACH |
|---------|-------|----------|
| **Input Type** | Number or list | Map or set |
| **Addressing** | Numeric index `[0]` | Key-based `["name"]` |
| **Stability** | Less stable | More stable |
| **Item Removal** | May recreate resources | Only removes specific resource |
| **Use Case** | Simple scenarios | Production environments |
| **Readability** | Index-based | Name-based (better) |

## 💡 Best Practices

1. **Prefer for_each over count** in production environments
2. **Use meaningful keys** when using for_each with maps
3. **Use toset()** to convert lists to sets for for_each
4. **Add proper tags** to all resources for better organization
5. **Document your choices** - explain why you chose count or for_each
6. **Test removals** - understand what happens when you remove items

## 🔗 Additional Resources

- [Terraform Count Meta-Argument](https://www.terraform.io/language/meta-arguments/count)
- [Terraform For_Each Meta-Argument](https://www.terraform.io/language/meta-arguments/for_each)
- [For Expressions](https://www.terraform.io/language/expressions/for)
- [Splat Expressions](https://www.terraform.io/language/expressions/splat)

## 🐛 Troubleshooting

### Issue: "Bucket name already exists"
**Solution:** S3 bucket names are globally unique. Change the bucket names in your variables.

### Issue: "Invalid for_each argument"
**Solution:** for_each requires a map or set. Use `toset()` to convert a list to a set.

### Issue: "Resource not found when using count"
**Solution:** Make sure you're using the correct index. Remember that count uses numeric indices starting from 0.

