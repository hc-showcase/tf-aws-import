# tf-aws-import
This example shows hot to import existing AWS resources into TF.

To follow this example you need to have a couple of resources in AWS available. You can use this repo to create some: [tf-aws-simple](https://github.com/hc-showcase/tf-aws-simple). 

The following environment variables are set:

AWS_ACCESS_KEY_ID='abc123'
AWS_SECRET_ACCESS_KEY 'abc123'
AWS_DEFAULT_REGION 'eu-central-1'
Clone this repository to your local machine.

1. Clone this repository to your machine.
```
mkaesz@arch ~/workspace> git clone https://github.com/hc-showcase/tf-aws-import
Cloning into 'tf-aws-import'...
remote: Enumerating objects: 20, done.
remote: Counting objects: 100% (20/20), done.
remote: Compressing objects: 100% (14/14), done.
remote: Total 20 (delta 3), reused 14 (delta 1), pack-reused 0
Unpacking objects: 100% (20/20), 6.55 KiB | 2.18 MiB/s, done.
```

2. Initialize the providers
```
mkaesz@arch ~/workspace> cd tf-aws-import
mkaesz@arch ~/w/tf-aws-import (master)> terraform init
Initializing modules...
- mod1 in mod1
- mod2 in mod2

Initializing the backend...

Initializing provider plugins...

The following providers do not have any version constraints in configuration,
so the latest version was installed.

To prevent automatic upgrades to new major versions that may contain breaking
changes, it is recommended to add version = "..." constraints to the
corresponding provider blocks in configuration, with the constraint strings
suggested below.

* provider.aws: version = "~> 3.1"

Terraform has been successfully initialized!

You may now begin working with Terraform. Try running "terraform plan" to see
any changes that are required for your infrastructure. All Terraform commands
should now work.

If you ever set or change modules or backend configuration for Terraform,
rerun this command to reinitialize your working directory. If you forget, other
commands will detect it and remind you to do so if necessary.
```

3. The Terraform configuration looks like the following:
```
mkaesz@arch ~/w/tf-aws-import (master)> tree
.
├── LICENSE
├── README.md
├── main.tf
├── mod1
│   └── instance.tf
└── mod2
    └── vpc.tf

2 directories, 5 files

mkaesz@arch ~/w/tf-aws-import (master)> cat main.tf
provider "aws" {
}

resource "aws_s3_bucket" "example" {
}

module "mod1" {
  source = "./mod1"
}

module "mod2" {
  source = "./mod2"
}

mkaesz@arch ~/w/tf-aws-import (master)> cat mod1/instance.tf
resource "aws_instance" "example" {
}

mkaesz@arch ~/w/tf-aws-import (master)> cat mod2/vpc.tf
resource "aws_vpc" "example" {
}
```

4. Execute Terraform validate
The output will show which configuration is mandatory. Ignore for now.

```
mkaesz@arch ~/w/tf-aws-import (master)> terraform validate

Error: Missing required argument

  on mod1/instance.tf line 1, in resource "aws_instance" "example":
   1: resource "aws_instance" "example" {

The argument "ami" is required, but no definition was found.


Error: Missing required argument

  on mod1/instance.tf line 1, in resource "aws_instance" "example":
   1: resource "aws_instance" "example" {

The argument "instance_type" is required, but no definition was found.


Error: Missing required argument

  on mod2/vpc.tf line 1, in resource "aws_vpc" "example":
   1: resource "aws_vpc" "example" {

The argument "cidr_block" is required, but no definition was found.
```

5. Import existing AWS resources into Terraform.
To import a resource you need to have the ID of the resource. In the case of AWS you can go to the AWS Console or use the AWS CLI. In my case, the IDs that I collected are the following:
* EC2 instance: i-0b7402f4c0ed0f470
* VPC: vpc-0a7fb68129373de87
* S3 bucket: my-tf-test-bucket-lkjahk3333

To import each resource you have to execute the terraform import command once per resource:
```
mkaesz@arch ~/w/tf-aws-simple (master)> terraform import
The import command expects two arguments.
Usage: terraform import [options] ADDR ID

  Import existing infrastructure into your Terraform state.

  This will find and import the specified resource into your Terraform
  state, allowing existing infrastructure to come under Terraform
  management without having to be initially created by Terraform.

  The ADDR specified is the address to import the resource to. Please
  see the documentation online for resource addresses. The ID is a
  resource-specific ID to identify that resource being imported. Please
  reference the documentation for the resource type you're importing to
  determine the ID syntax to use. It typically matches directly to the ID
  that the provider uses.

  The current implementation of Terraform import can only import resources
  into the state. It does not generate configuration. A future version of
  Terraform will also generate configuration.

  Because of this, prior to running terraform import it is necessary to write
  a resource configuration block for the resource manually, to which the
  imported object will be attached.

  This command will not modify your infrastructure, but it will make
  network requests to inspect parts of your infrastructure relevant to
  the resource being imported.
```

The ADDR specifies the address to import the resource to. In hour case these are:
* EC2 instance: module.mod1.aws_instance.example
* VPC: module.mod2.aws_vpc.example
* S3 bucket: aws_s3_bucket.example

The address scheme follows the Terraform conventions: [Link](https://www.terraform.io/docs/internals/resource-addressing.html)

Execute now the following commands to import the resources:
```
mkaesz@arch ~/w/tf-aws-import (master) [1]> terraform import module.mod1.aws_instance.example i-0b7402f4c0ed0f470
module.mod1.aws_instance.example: Importing from ID "i-0b7402f4c0ed0f470"...
module.mod1.aws_instance.example: Import prepared!
  Prepared aws_instance for import
module.mod1.aws_instance.example: Refreshing state... [id=i-0b7402f4c0ed0f470]

Import successful!

The resources that were imported are shown above. These resources are now in
your Terraform state and will henceforth be managed by Terraform.


mkaesz@arch ~/w/tf-aws-import (master)> terraform import module.mod2.aws_vpc.example vpc-0a7fb68129373de87
module.mod2.aws_vpc.example: Importing from ID "vpc-0a7fb68129373de87"...
module.mod2.aws_vpc.example: Import prepared!
  Prepared aws_vpc for import
module.mod2.aws_vpc.example: Refreshing state... [id=vpc-0a7fb68129373de87]

Import successful!

The resources that were imported are shown above. These resources are now in
your Terraform state and will henceforth be managed by Terraform.


mkaesz@arch ~/w/tf-aws-import (master)> terraform import aws_s3_bucket.example my-tf-test-bucket-lkjahk3333
aws_s3_bucket.example: Importing from ID "my-tf-test-bucket-lkjahk3333"...
aws_s3_bucket.example: Import prepared!
  Prepared aws_s3_bucket for import
aws_s3_bucket.example: Refreshing state... [id=my-tf-test-bucket-lkjahk3333]

Import successful!

The resources that were imported are shown above. These resources are now in
your Terraform state and will henceforth be managed by Terraform.
```

6. Check if the local state is valid:
```
mkaesz@arch ~/w/tf-aws-import (master)> terraform state list
aws_s3_bucket.example
module.mod1.aws_instance.example
module.mod2.aws_vpc.example
```

7. Complete the Terraform configuration:
Terraform validate still throws errors about missing configuration items:

```
mkaesz@arch ~/w/tf-aws-import (master)> terraform validate

Error: Missing required argument

  on mod1/instance.tf line 1, in resource "aws_instance" "example":
   1: resource "aws_instance" "example" {

The argument "instance_type" is required, but no definition was found.


Error: Missing required argument

  on mod1/instance.tf line 1, in resource "aws_instance" "example":
   1: resource "aws_instance" "example" {

The argument "ami" is required, but no definition was found.


Error: Missing required argument

  on mod2/vpc.tf line 1, in resource "aws_vpc" "example":
   1: resource "aws_vpc" "example" {

The argument "cidr_block" is required, but no definition was found.
```

Now cherry-pick what is needed. Execute terraform show to get the field with its data:
```
mkaesz@arch ~/w/tf-aws-import (master)> terraform show
# aws_s3_bucket.example:
resource "aws_s3_bucket" "example" {
    arn                         = "arn:aws:s3:::my-tf-test-bucket-lkjahk3333"
    bucket                      = "my-tf-test-bucket-lkjahk3333"
    bucket_domain_name          = "my-tf-test-bucket-lkjahk3333.s3.amazonaws.com"
    bucket_regional_domain_name = "my-tf-test-bucket-lkjahk3333.s3.eu-central-1.amazonaws.com"
    hosted_zone_id              = "Z21DNDUVLTQW6Q"
    id                          = "my-tf-test-bucket-lkjahk3333"
    region                      = "eu-central-1"
    request_payer               = "BucketOwner"
    tags                        = {
        "Name" = "My bucket"
    }

    versioning {
        enabled    = false
        mfa_delete = false
    }
}


# module.mod1.aws_instance.example:
resource "aws_instance" "example" {
    **ami                          = "ami-0d359437d1756caa8"**
    arn                          = "arn:aws:ec2:eu-central-1:729476260648:instance/i-0b7402f4c0ed0f470"
    associate_public_ip_address  = true
    availability_zone            = "eu-central-1c"
    cpu_core_count               = 1
    cpu_threads_per_core         = 1
    disable_api_termination      = false
    ebs_optimized                = false
    get_password_data            = false
    hibernation                  = false
    id                           = "i-0b7402f4c0ed0f470"
    instance_state               = "running"
    **instance_type                = "t2.micro"**
    ipv6_address_count           = 0
    ipv6_addresses               = []
    monitoring                   = false
    primary_network_interface_id = "eni-0d54fa785aa65b016"
    private_dns                  = "ip-172-31-37-74.eu-central-1.compute.internal"
    private_ip                   = "172.31.37.74"
    public_dns                   = "ec2-35-158-107-247.eu-central-1.compute.amazonaws.com"
    public_ip                    = "35.158.107.247"
    secondary_private_ips        = []
    security_groups              = [
        "default",
    ]
    source_dest_check            = true
    subnet_id                    = "subnet-0dc6fc47"
    tags                         = {}
    tenancy                      = "default"
    volume_tags                  = {}
    vpc_security_group_ids       = [
        "sg-8b09abe0",
    ]

    credit_specification {
        cpu_credits = "standard"
    }

    metadata_options {
        http_endpoint               = "enabled"
        http_put_response_hop_limit = 1
        http_tokens                 = "optional"
    }

    root_block_device {
        delete_on_termination = true
        device_name           = "/dev/sda1"
        encrypted             = false
        iops                  = 100
        volume_id             = "vol-0b7d85e3b388122ae"
        volume_size           = 8
        volume_type           = "gp2"
    }

    timeouts {}
}


# module.mod2.aws_vpc.example:
resource "aws_vpc" "example" {
    arn                              = "arn:aws:ec2:eu-central-1:729476260648:vpc/vpc-0a7fb68129373de87"
    assign_generated_ipv6_cidr_block = false
    **cidr_block                       = "10.0.0.0/16"**
    default_network_acl_id           = "acl-0ecb9601fa7dc351b"
    default_route_table_id           = "rtb-00268b546c04800cf"
    default_security_group_id        = "sg-0b48ef368731f18f9"
    dhcp_options_id                  = "dopt-4f934827"
    enable_dns_hostnames             = false
    enable_dns_support               = true
    id                               = "vpc-0a7fb68129373de87"
    instance_tenancy                 = "default"
    main_route_table_id              = "rtb-00268b546c04800cf"
    owner_id                         = "729476260648"
    tags                             = {}
}
```

Your Terraform configuration should now look like the following:
```
mkaesz@arch ~/w/tf-aws-import (master)> cat main.tf
provider "aws" {
}

resource "aws_s3_bucket" "example" {
}

module "mod1" {
  source = "./mod1"
}

module "mod2" {
  source = "./mod2"
}

mkaesz@arch ~/w/tf-aws-import (master)> cat mod1/instance.tf
resource "aws_instance" "example" {
    ami                          = "ami-0d359437d1756caa8"
    instance_type                = "t2.micro"
}

mkaesz@arch ~/w/tf-aws-import (master)> cat mod2/vpc.tf
resource "aws_vpc" "example" {
    cidr_block                       = "10.0.0.0/16"
}
```

And another execution of terraform validate should now not complain anymore:
```
mkaesz@arch ~/w/tf-aws-import (master)> terraform validate
Success! The configuration is valid.
```

8. Execute terraform plan
When we execute Terraform plan, we see that Terraform recognizes that a Tag is not specified in the local Terraform config as well as two field weren't mandatory, but the creator of the original resource specified them. This is considered a change:
```
mkaesz@arch ~/w/tf-aws-import (master)> terraform plan
Refreshing Terraform state in-memory prior to plan...
The refreshed state will be used to calculate this plan, but will not be
persisted to local or remote state storage.

module.mod2.aws_vpc.example: Refreshing state... [id=vpc-0a7fb68129373de87]
module.mod1.aws_instance.example: Refreshing state... [id=i-0b7402f4c0ed0f470]
aws_s3_bucket.example: Refreshing state... [id=my-tf-test-bucket-lkjahk3333]

------------------------------------------------------------------------

An execution plan has been generated and is shown below.
Resource actions are indicated with the following symbols:
  ~ update in-place

Terraform will perform the following actions:

  # aws_s3_bucket.example will be updated in-place
  ~ resource "aws_s3_bucket" "example" {
      + acl                         = "private"
        arn                         = "arn:aws:s3:::my-tf-test-bucket-lkjahk3333"
        bucket                      = "my-tf-test-bucket-lkjahk3333"
        bucket_domain_name          = "my-tf-test-bucket-lkjahk3333.s3.amazonaws.com"
        bucket_regional_domain_name = "my-tf-test-bucket-lkjahk3333.s3.eu-central-1.amazonaws.com"
      + force_destroy               = false
        hosted_zone_id              = "Z21DNDUVLTQW6Q"
        id                          = "my-tf-test-bucket-lkjahk3333"
        region                      = "eu-central-1"
        request_payer               = "BucketOwner"
      ~ tags                        = {
          - "Name" = "My bucket" -> null
        }

        versioning {
            enabled    = false
            mfa_delete = false
        }
    }

Plan: 0 to add, 1 to change, 0 to destroy.

------------------------------------------------------------------------

Note: You didn't specify an "-out" parameter to save this plan, so Terraform
can't guarantee that exactly these actions will be performed if
"terraform apply" is subsequently run.
```

We need to add this Tag and the two fields manually to our local config. The resulting config looks like this:
```
mkaesz@arch ~/w/tf-aws-import (master)> cat main.tf
provider "aws" {
}

resource "aws_s3_bucket" "example" {
  acl            = "private"
  force_destroy  = false

  tags = {
    "Name"       = "My bucket"
  }
}

module "mod1" {
  source = "./mod1"
}

module "mod2" {
  source = "./mod2"
}
```

When we execute terraform plan again, we see that the fields are still recognized as change, but it is safe now:
```
mkaesz@arch ~/w/tf-aws-import (master)> terraform plan
Refreshing Terraform state in-memory prior to plan...
The refreshed state will be used to calculate this plan, but will not be
persisted to local or remote state storage.

module.mod2.aws_vpc.example: Refreshing state... [id=vpc-0a7fb68129373de87]
module.mod1.aws_instance.example: Refreshing state... [id=i-0b7402f4c0ed0f470]
aws_s3_bucket.example: Refreshing state... [id=my-tf-test-bucket-lkjahk3333]

------------------------------------------------------------------------

An execution plan has been generated and is shown below.
Resource actions are indicated with the following symbols:
  ~ update in-place

Terraform will perform the following actions:

  # aws_s3_bucket.example will be updated in-place
  ~ resource "aws_s3_bucket" "example" {
      + acl                         = "private"
        arn                         = "arn:aws:s3:::my-tf-test-bucket-lkjahk3333"
        bucket                      = "my-tf-test-bucket-lkjahk3333"
        bucket_domain_name          = "my-tf-test-bucket-lkjahk3333.s3.amazonaws.com"
        bucket_regional_domain_name = "my-tf-test-bucket-lkjahk3333.s3.eu-central-1.amazonaws.com"
      + force_destroy               = false
        hosted_zone_id              = "Z21DNDUVLTQW6Q"
        id                          = "my-tf-test-bucket-lkjahk3333"
        region                      = "eu-central-1"
        request_payer               = "BucketOwner"
        tags                        = {
            "Name" = "My bucket"
        }

        versioning {
            enabled    = false
            mfa_delete = false
        }
    }

Plan: 0 to add, 1 to change, 0 to destroy.

------------------------------------------------------------------------

Note: You didn't specify an "-out" parameter to save this plan, so Terraform
can't guarantee that exactly these actions will be performed if
"terraform apply" is subsequently run.
```

Now we can execute terraform apply:
```
mkaesz@arch ~/w/tf-aws-import (master)> terraform apply
module.mod2.aws_vpc.example: Refreshing state... [id=vpc-0a7fb68129373de87]
module.mod1.aws_instance.example: Refreshing state... [id=i-0b7402f4c0ed0f470]
aws_s3_bucket.example: Refreshing state... [id=my-tf-test-bucket-lkjahk3333]

An execution plan has been generated and is shown below.
Resource actions are indicated with the following symbols:
  ~ update in-place

Terraform will perform the following actions:

  # aws_s3_bucket.example will be updated in-place
  ~ resource "aws_s3_bucket" "example" {
      + acl                         = "private"
        arn                         = "arn:aws:s3:::my-tf-test-bucket-lkjahk3333"
        bucket                      = "my-tf-test-bucket-lkjahk3333"
        bucket_domain_name          = "my-tf-test-bucket-lkjahk3333.s3.amazonaws.com"
        bucket_regional_domain_name = "my-tf-test-bucket-lkjahk3333.s3.eu-central-1.amazonaws.com"
      + force_destroy               = false
        hosted_zone_id              = "Z21DNDUVLTQW6Q"
        id                          = "my-tf-test-bucket-lkjahk3333"
        region                      = "eu-central-1"
        request_payer               = "BucketOwner"
        tags                        = {
            "Name" = "My bucket"
        }

        versioning {
            enabled    = false
            mfa_delete = false
        }
    }

Plan: 0 to add, 1 to change, 0 to destroy.

Do you want to perform these actions?
  Terraform will perform the actions described above.
  Only 'yes' will be accepted to approve.

  Enter a value: yes

aws_s3_bucket.example: Modifying... [id=my-tf-test-bucket-lkjahk3333]
aws_s3_bucket.example: Modifications complete after 2s [id=my-tf-test-bucket-lkjahk3333]

Apply complete! Resources: 0 added, 1 changed, 0 destroyed.
```

Terraform plan show now an up-to-date state:
```
mkaesz@arch ~/w/tf-aws-import (master)> terraform plan
Refreshing Terraform state in-memory prior to plan...
The refreshed state will be used to calculate this plan, but will not be
persisted to local or remote state storage.

module.mod2.aws_vpc.example: Refreshing state... [id=vpc-0a7fb68129373de87]
aws_s3_bucket.example: Refreshing state... [id=my-tf-test-bucket-lkjahk3333]
module.mod1.aws_instance.example: Refreshing state... [id=i-0b7402f4c0ed0f470]

------------------------------------------------------------------------

No changes. Infrastructure is up-to-date.

This means that Terraform did not detect any differences between your
configuration and real physical resources that exist. As a result, no
actions need to be performed.
```

The resources have been successfully migrated!

9. Destroy all resources
```
mkaesz@arch ~/w/tf-aws-import (master)> terraform destroy
module.mod2.aws_vpc.example: Refreshing state... [id=vpc-0a7fb68129373de87]
module.mod1.aws_instance.example: Refreshing state... [id=i-0b7402f4c0ed0f470]
aws_s3_bucket.example: Refreshing state... [id=my-tf-test-bucket-lkjahk3333]

An execution plan has been generated and is shown below.
Resource actions are indicated with the following symbols:
  - destroy

Terraform will perform the following actions:

  # aws_s3_bucket.example will be destroyed
  - resource "aws_s3_bucket" "example" {
      - acl                         = "private" -> null
      - arn                         = "arn:aws:s3:::my-tf-test-bucket-lkjahk3333" -> null
      - bucket                      = "my-tf-test-bucket-lkjahk3333" -> null
      - bucket_domain_name          = "my-tf-test-bucket-lkjahk3333.s3.amazonaws.com" -> null
      - bucket_regional_domain_name = "my-tf-test-bucket-lkjahk3333.s3.eu-central-1.amazonaws.com" -> null
      - force_destroy               = false -> null
      - hosted_zone_id              = "Z21DNDUVLTQW6Q" -> null
      - id                          = "my-tf-test-bucket-lkjahk3333" -> null
      - region                      = "eu-central-1" -> null
      - request_payer               = "BucketOwner" -> null
      - tags                        = {
          - "Name" = "My bucket"
        } -> null

      - versioning {
          - enabled    = false -> null
          - mfa_delete = false -> null
        }
    }

  # module.mod1.aws_instance.example will be destroyed
  - resource "aws_instance" "example" {
      - ami                          = "ami-0d359437d1756caa8" -> null
      - arn                          = "arn:aws:ec2:eu-central-1:729476260648:instance/i-0b7402f4c0ed0f470" -> null
      - associate_public_ip_address  = true -> null
      - availability_zone            = "eu-central-1c" -> null
      - cpu_core_count               = 1 -> null
      - cpu_threads_per_core         = 1 -> null
      - disable_api_termination      = false -> null
      - ebs_optimized                = false -> null
      - get_password_data            = false -> null
      - hibernation                  = false -> null
      - id                           = "i-0b7402f4c0ed0f470" -> null
      - instance_state               = "running" -> null
      - instance_type                = "t2.micro" -> null
      - ipv6_address_count           = 0 -> null
      - ipv6_addresses               = [] -> null
      - monitoring                   = false -> null
      - primary_network_interface_id = "eni-0d54fa785aa65b016" -> null
      - private_dns                  = "ip-172-31-37-74.eu-central-1.compute.internal" -> null
      - private_ip                   = "172.31.37.74" -> null
      - public_dns                   = "ec2-35-158-107-247.eu-central-1.compute.amazonaws.com" -> null
      - public_ip                    = "35.158.107.247" -> null
      - secondary_private_ips        = [] -> null
      - security_groups              = [
          - "default",
        ] -> null
      - source_dest_check            = true -> null
      - subnet_id                    = "subnet-0dc6fc47" -> null
      - tags                         = {} -> null
      - tenancy                      = "default" -> null
      - volume_tags                  = {} -> null
      - vpc_security_group_ids       = [
          - "sg-8b09abe0",
        ] -> null

      - credit_specification {
          - cpu_credits = "standard" -> null
        }

      - metadata_options {
          - http_endpoint               = "enabled" -> null
          - http_put_response_hop_limit = 1 -> null
          - http_tokens                 = "optional" -> null
        }

      - root_block_device {
          - delete_on_termination = true -> null
          - device_name           = "/dev/sda1" -> null
          - encrypted             = false -> null
          - iops                  = 100 -> null
          - volume_id             = "vol-0b7d85e3b388122ae" -> null
          - volume_size           = 8 -> null
          - volume_type           = "gp2" -> null
        }

      - timeouts {}
    }

  # module.mod2.aws_vpc.example will be destroyed
  - resource "aws_vpc" "example" {
      - arn                              = "arn:aws:ec2:eu-central-1:729476260648:vpc/vpc-0a7fb68129373de87" -> null
      - assign_generated_ipv6_cidr_block = false -> null
      - cidr_block                       = "10.0.0.0/16" -> null
      - default_network_acl_id           = "acl-0ecb9601fa7dc351b" -> null
      - default_route_table_id           = "rtb-00268b546c04800cf" -> null
      - default_security_group_id        = "sg-0b48ef368731f18f9" -> null
      - dhcp_options_id                  = "dopt-4f934827" -> null
      - enable_dns_hostnames             = false -> null
      - enable_dns_support               = true -> null
      - id                               = "vpc-0a7fb68129373de87" -> null
      - instance_tenancy                 = "default" -> null
      - main_route_table_id              = "rtb-00268b546c04800cf" -> null
      - owner_id                         = "729476260648" -> null
      - tags                             = {} -> null
    }

Plan: 0 to add, 0 to change, 3 to destroy.

Do you really want to destroy all resources?
  Terraform will destroy all your managed infrastructure, as shown above.
  There is no undo. Only 'yes' will be accepted to confirm.

  Enter a value: yes

module.mod2.aws_vpc.example: Destroying... [id=vpc-0a7fb68129373de87]
module.mod1.aws_instance.example: Destroying... [id=i-0b7402f4c0ed0f470]
aws_s3_bucket.example: Destroying... [id=my-tf-test-bucket-lkjahk3333]
aws_s3_bucket.example: Destruction complete after 1s
module.mod2.aws_vpc.example: Destruction complete after 1s
module.mod1.aws_instance.example: Still destroying... [id=i-0b7402f4c0ed0f470, 10s elapsed]
module.mod1.aws_instance.example: Still destroying... [id=i-0b7402f4c0ed0f470, 20s elapsed]
module.mod1.aws_instance.example: Destruction complete after 30s

Destroy complete! Resources: 3 destroyed.
```

Everything got cleanup properly:
```
mkaesz@arch ~/w/tf-aws-import (master)> terraform show
<no output>
mkaesz@arch ~/w/tf-aws-import (master)> terraform state list
<no output>
```
