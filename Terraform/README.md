

### TERRAFORM  
###### What is Terraform and what can be done by Terraform.  
- (Terra=Earth)(Form=Shaping) Shaping the earth.    
- Firs released 2014 by HashiCorp. Tool for automation of shaping your insfrastructure of your choice.  
- You can codify a complete DC through archivable version controlled code.  
- Orchestrate across multiple cloud services like AWS, Azure, GCP within a single definition. 
- It's cloud agnostic.   

###### What is infrastructure as code (`IAC`) ?  
- Describing an infrastructure as a code, keeping in `VCS` like `Git` and shaping the infrustructure by code changes done in `Git` automatedly by using a CI/CD tool like `Jenkins`, CircleCI, Travis.  
- Same tools and processes software developers use.   

###### How `Terraform` differs from other automation tools like Chef, Puppet, Ansible ?  
- Puppet and chef configuration management tools focuses software installation and configuration on machines.  
- Terraform is used for one level down to shape your infrastructue and it's cloud agnostic.  
- Difference from `CloudFormation`, `Azure ARM (Resource Manager)`, `Heat`, it supports multiple clouds.  
- `Boto and Fog` `API` libraries that focueses on specific clouds to provide native access. It provides low-level access.  
- Easier syntax.  
- Terraform manages any resource and tools required to automate the building of infrastructure. No need to learn and manage tools.  

###### Advantages of `Terraform`  
- Orchestrate multiple services/provides in a single definition as a single solution for many solutions.  
- Build in protection for when you need to add more solutions later.  
- It's very popular and had been market standard.  

### INSTALLATION OF TERRAFORM

###### Installation on Linux
- https://www.terraform.io/downloads.html
- A script to install latest versions.  
```
#!/bin/bash
sudo apt-get install update
sudo apt-get -y install unzip jq curl
function terraform-install() {
  [[ -f ${HOME}/bin/terraform ]] && echo "`${HOME}/bin/terraform version` already installed at ${HOME}/bin/terraform" && return 0
  LATEST_URL=$(curl -sL https://releases.hashicorp.com/terraform/index.json | jq -r '.versions[].builds[].url' | sort --version-sort | egrep -v 'rc|beta' | egrep 'linux.*amd64' |tail -1)
  curl ${LATEST_URL} > /tmp/terraform.zip
  mkdir -p ${HOME}/bin
  (cd ${HOME}/bin && unzip /tmp/terraform.zip)
  if [[ -z $(grep 'export PATH=${HOME}/bin:${PATH}' ~/.bashrc) ]]; then
  	echo 'export PATH=${HOME}/bin:${PATH}' >> ~/.bashrc
  fi
  
  echo "Installed: `${HOME}/bin/terraform version`"
  
  cat - << EOF 
 
Run the following to reload your PATH with terraform:
  source ~/.bashrc
EOF
}

terraform-install
```
- Validate the installation  
```
terraform version
```

###### Installation on Windows  
- Download zip file.  
- Extract to a directory.  
- Add the path where files extracted, to `Path` key in `User Environment Variables`.  
- Validate the installation.  
```
terraform version
```

### Terraform Deployment Example
###### Prerquisites:  
- Create a user in `AWS` with `AdministativeRights`
- Create a `terraform.tf` file.
- Never
```
provider "aws" {
access_key = "ACCESS_KEY"
secret_key = "SECRET_KEY"
region = "eu-central-1"
}

resource "aws_instance" "example" {
ami = ""
instance_type = "t2.micro"
}

###### Run
- init: Initializes, does the first configuration and plugins like aws provider described in `tf` file.  
```
```
terraform init
```
- plan: Shows you a list of options you can apply.  
- You can add those parameters to the configuration file for more specific configurations.  
```
terraform plan
```
- apply: Apply the terraform file.  
```
terraform apply
```
- show: Show the configuration done.  
```
terraform show
```
- destroy: Destroys the configuration described in terraform.tf file.  
```
terraform destroy
```
- Additionally terraform creates a file `terraform.tfstate` which keeps state of the current infrastructure in JSON format during `apply`. Do not edit it. When you `destoy` or change, it creates `terraform.tfstate.backup` file to keep the track of previous state. 

### Terraform Basics:  
- Config File Structure  
- CLI and Common Executions  
- Resources  
- Providers  
- Variables  
- Outputting Attributer

###### Config File Structure:
- `HCL` language (HashiCorp Configuration Language).  
- `HCL` and `JSON` are interoperable.  
- It can also understand JSON but it's harder and less human readable.  
- Syntax allows hierarchies of sections.  
```
## Comment
# comment
## Wrap multi-line comments
/* comment */
## Assign key and values
'key = value'
## Strings
"string"  can insert net value when wrapped ${}  
## shell-style here doc.
<< EOF
EOF
## Boolean values
true & false
## Lists
[ "lony","semi","namer" ]
## Making maps
{ "semi":"lony","namer":"tali" }
```

###### CLI and Common Executions  
```
terraform plan -out=path
```
```
terraform plan --destroy -out destroy-plan
terraform apply destroy-plan
terraform destroy
```
```
# Refresh: Gets the tfstate
terraform refresh
# Show
terraform show
# Validate the state, for using maybe in a pipeline
terraform validate
```
- An example, this ways it doesn't ask yes or no.  
```
mkdir folder_to_tf_files
cp terraform.tf folder_to_tf_files
# Initialize according to a tf file
terraform init -upgrade folder_to_tf_files/
terraform plan -out build-plan
# Will apply witout asking yes or no
terraform apply build-plan
terraform plan --destroy -out destroy-plan
terraform apply destroy-plan
```
- Validating files for mistakes
```
terraform validate
```
####
 RESOURCES
- Resources are actual infrastructure components  
- They may be low and high level resources
###### depends_on, count
```
resource "aws_instance" "backend" {
#COUNT
count = 2
ami = "ami-086a09d5b9fa35dc7"
instance_type = "t2.micro"
}
```
```
resource "aws_instance"  "frontend" {
#DEPENDS_ON
depends_on = [ "aws_instance.backend" ]
ami = "ami-086a09d5b9fa35dc7"
instance_type = "t2.micro"
}
```
###### lifecycle, create_before_destroy, prevent_destroy
-  During an instance update, create first and than destroy  
```
lifecycle {
  create_before_destroy = true
}
```
- Prevent Destroy: It will not let you destroy and will give error during plan  
```
lifecycle {
  prevent_destroy = true
}
```
###### timeouts, create, delete
```
timeouts {
create = "60m"
delete = "2h"
}
```
###### DEMO CODE WITH ALL KNOWLEDGE TILL NOW
```
provider "aws" {
access_key = "AWS_ACCESS_KEY"
secret_key = "AWS_ACCESS_SECRET"
region = "eu-central-1"
}

resource "aws_instance" "backend" {
count = 2
ami = "ami-086a09d5b9fa35dc7"
instance_type= "t2.micro"
timeouts {
  create = "5m"
  delete = "5m"
  }
lifecycle {
  create_before_destroy = true
  }
}

resource "aws_instance" "frontend" {
count = 1
ami = "ami-086a09d5b9fa35dc7"
instance_type = "t2.micro"
depends_on = [ "aws_instance.backend" ]
timeouts {
  create = "5m"
  delete = "5m"
  }
lifecycle {
  prevent_destroy = false
```

#### PROVIDERS
- Providers are generally IaaS or SaaS services and responsible for understanding API interactions and exposing resources.  
- https://www.terraform.io/docs/providers/index.html  
- `terraform init` command download/provides latest version of provider by descriptions in your code.  
- Multiple providers possible to use.  

Config: terraform.tf
```
# Default Provider
provider "aws" {
access_key = "ACCESS_KEY"
secret_key = "SECRET_KEY"
region = "eu-central-1"
}

# Additional provider
provider "aws" {
access_key = "ACCESS_KEY"
secret_key = "SECRET_KEY"
region = "eu-west-3"
alias = "Paris"
}

resource "aws_instance" "backend" {
count = 2
ami = "ami-086a09d5b9fa35dc7"
instance_type= "t2.micro"
timeouts {
  create = "5m"
  delete = "5m"
  }
lifecycle {
  create_before_destroy = true
  }
}

resource "aws_instance" "frontend" {
# Provider selection in resource
provider = "aws.Paris"
count = 1
#ami = "ami-086a09d5b9fa35dc7"
# Paris Region different image.
ami = "ami-075b44448d2276521"
instance_type = "t2.micro"
depends_on = [ "aws_instance.backend" ]
timeouts {
  create = "5m"
  delete = "5m"
  }
lifecycle {
  prevent_destroy = false
  }
}

```
###### ZIP PROVIDER
```
provider "archive" {}

data "archive_file" "zip" {
  type        = "zip"
  source_file = "hello_lambda.py"
  output_path = "hello_lambda.zip"
}
```

#### VARIABLES
- Strings, lists, maps and booleans.  
###### VARIABLES IN TF CONFIGURATION
```
variable "key" {
type = "string"
default = "value"
description = "for what it's used, explanation"
}

variable "key" {
type = "list"
default = [ "admin", "ubuntu"]
description = "for what it's used, explanation"
}

variable "images" {
  type = "map"
  default = {
    "us-east-1" = "image-1234"
    "us-west-2" = "image-4567"
  }
}
```
###### VARIABLES IN VAR FILE
- var file example;  
Config: var_file_example.tfvars
```
zones = ["eu-central-1a", "eu-central-1b"]
```
Run:
```
terraform apply -var-file=var_file_example.tfvars -var-file=bar.tfvars
```

###### VARIABLES IN COMMAND LINE PIPED / EXPORTED
```
TF_VAR_image=test terraform apply
```
OR
```
export TF_VAR_zones='[ "eu-central-1a", "eu-central-1b" ]'
terraform init
terraform plan
terraform apply
```

###### EXAMPLE: CONFIG WITH VARIABLES  
- This will create each instance according in 2 different zones.  
Config: terraform.tf  
```
provider "aws" {
  access_key = "ACCESS_KEY"
  secret_key = "SECRET_KEY"
  region = "eu-central-1"
}

variable "zones" {
  default = [ "eu-central-1a", "eu-central-1b" ]
}

resource "aws_instance" "demo" {
  count = 2
  ami = "ami-086a09d5b9fa35dc7"
  instance_type = "t2.micro"
  availability_zone = "${var.zones[count.index]}"
}
```

#### OUTPUT ATTRIBUTES
- Outputing attriput provides getting information from our deployment, like private ip, public ip, dns name of an instance that we create and use as variable for another issue.  
- `'sensitive'[bbolean]`(boolean) - Indicates sensitive material within the block; set attribute to `"=true"`.  
- `'depends_on'`[ list of strings] - creates dependencies before the output is value is processed.
- `'description'` - to set a description.
```
output "name" {
  value = "getvalue"
} 
```
Example:
```
provider "aws" {
  access_key = "ACCESS_KEY"
  secret_key = "SECRET_KEY"
  region = "eu-central-1"
}

variable "zones" {
  default = [ "eu-central-1a", "eu-central-1b" ]
}

resource "aws_instance" "single" {
ami= "ami-086a09d5b9fa35dc7"
instance_type = "t2.micro"
}

resource "aws_instance" "demo" {
  count = 2
  ami = "ami-086a09d5b9fa35dc7"
  instance_type = "t2.micro"
  availability_zone = "${var.zones[count.index]}"
}


output "demo_public_ips" {
  value = "${aws_instance.single.public_ip}"
}

output "demo_private_ips" {
  value = "${aws_instance.demo.*.private_ip}"
}
```

###### EXERCISE WITH THE KNOWLEDGE TILL HERE
- Create a 2 backend and 2 frontend servers in 2 different availability zones and regions in AWS.  
- Set frontend servers as create before destroy and backend servers prevent destroy in lifecycle.  
- Set creation and deletetion timeouts 5 minutes.    
- Use variables for availability zones.  
- output public and private ip address of all servers.  
- frontend servers should depend on backend servers of it's availability zone.  
```
provider "aws" {
access_key = "ACCESS_KEY"
secret_key = "SECRET_KEY"
region = "eu-central-1"
}

provider "aws" {
alias = "Paris"
access_key = "ACCESS_KEY"
secret_key = "SECRET_KEY"
region = "eu-west-3"
}
variable "zones" {
  default = ["eu-central-1a", "eu-central-1b"]
}

variable "zones_paris" {
  default = [ "eu-west-3a", "eu-west-3b" ]
}

resource "aws_instance" "frontend" {
  count = 2
  depends_on = [ "aws_instance.backend" ]
  availability_zone = "${var.zones[count.index]}"
  ami = "ami-086a09d5b9fa35dc7"
  instance_type = "t2.micro"
  lifecycle {
    create_before_destroy = true
  }
  timeouts {
    create = "5m"
    delete = "5m"
  }
}

resource "aws_instance" "backend" {
  count = 2
  availability_zone = "${var.zones[count.index]}"
  ami = "ami-086a09d5b9fa35dc7"
  instance_type = "t2.micro"
  lifecycle {
    prevent_destroy = true
  }
  timeouts {
    create = "5m"
    delete = "5m"
  }
}

resource "aws_instance" "frontend-paris" {
  count = 2
  depends_on = [ "aws_instance.backend_paris" ]
  provider = "aws.Paris"
  availability_zone = "${var.zones_paris[count.index]}"
  ami = "ami-075b44448d2276521"
  instance_type = "t2.micro"
  lifecycle {
    create_before_destroy = true
  }
  timeouts {
    create = "5m"
    delete = "5m"
  }
}

resource "aws_instance" "backend_paris" {
  count = 2
  provider = "aws.Paris"
  availability_zone = "${var.zones_paris[count.index]}"
  ami = "ami-075b44448d2276521"
  instance_type = "t2.micro"
  lifecycle {
    prevent_destroy = true
  }
  timeouts {
    create = "5m"
    delete = "5m"
  }
}

output "all_public_ips" {
  value = "${aws_instance.backend.*.public_ip}"
}

output "all_private_ips" {
  value = "${aws_instance.frontend.*.private_ip}"
}
```

### BEYOND THE BASISCS
- Interpolation Expressions  
- Data Sources  
- Locals  
- Modules  
- Backends and Remote States  
- Workspaces  
- Software Provisioning and Provisioners  

###### Interpolation Expressions  
Interpolation Expressions Are;  
- Variable references  
- Conditionals  
- Built-in Functions  
- Console Command  
- Below some interpolated structures:  
```
## User String Variables
${var.foo}
## User Map Variables
${var.amis["eu-central-1"]}
## User List Variables
${var.sublents}
#OR
${var.subnets[idx]}
```
- Some more examples:  
```
${self.private_ip}
${aws_instance.web.id}
${aws_instance.web.*.id}.
```

###### EXPRESSIONS
- It can be used inside interpolated structures.  
- CONDITION ? TRUEVALUE : FALSEVALUE
```
resource "aws_instance" "vpn" {
  count = "${var.something ? 1 : 0}"
}
```

###### BUILT-IN FUNCTIONS: concat, join, contains, merge, length, replace
- `concat` : Blends 2 lists into one.  
```
concat(aws_instance,db,*,tags.Name, aws_instance.web.*.tags.Name)
```
- `join` : Joins the delimiter to the list to make a new string.  
```
join(",", aws_instance.foo.*.id)
join(",", var.ami_list)
```
- `contains` : Returns true if a list contains an element.  
```
contains(var.list_of_strings, "an_element")
```
- `merge` : Used for merging `maps`.  
```
${merge(map("a", "b"), map("c", "d"))}
```
Returns
```
{ "a": "b", "c": "d"}
```
- `length` : Number of characters or values.  
```
${length(split(",", "a,b,c"))} = 3
${length("a,b,c")} = 5
${length(map("key", "val"))} = 1
```
- Replace(string, search, replace)

###### TERRAFORM CONSOLE COMMAND
- It's a command to perform experimantal math operations.  
- terraform console[options][dirs]  
- Use this console to test out your interpolations before using them within configurations.  
```
echo "2 + 2" | terraform console
```
- You can test all the interpolation expressions in terraform console by typing;  
```
terraform console
```
Example Code: 
```
provider "aws" {
  access_key = "ACCESS_KEY"
  secret_key = "SECRET_KEY"
  region     = "us-east-1"
}

provider "aws" {
  alias      = "us-west-1"
  access_key = "ACCESS_KEY"
  secret_key = "SECRET_KEY"
  region     = "us-west-1"
}

variable "us-east-zones" {
  default = ["us-east-1a", "us-east-1b"]
}

variable "us-west-zones" {
  default = ["us-west-1c", "us-west-1b"]
}

variable "multi-region-deployment" {
  default = true
}

variable "environment-name" {
  default = "Terraform-demo"
}

resource "aws_instance" "frontend" {
  tags = {
    Name = "${join("-",list(var.environment-name, "frontend"))}"
  }

  depends_on        = ["aws_instance.backend"]
  availability_zone = "${var.us-east-zones[count.index]}"
  ami               = "ami-66506c1c"
  instance_type     = "t2.micro"
}

resource "aws_instance" "west_frontend" {
  tags = {
    Name = "${join("-",list(var.environment-name, "frontend"))}"
  }

  count             = "${var.multi-region-deployment ? 1 : 0}"
  depends_on        = ["aws_instance.west_backend"]
  provider          = "aws.us-west-1"
  ami               = "ami-07585467"
  availability_zone = "${var.us-west-zones[count.index]}"
  instance_type     = "t2.micro"
}

resource "aws_instance" "backend" {
  tags = {
    Name = "${join("-", list(var.environment-name, "backend"))}"
  }

  count             = 2
  availability_zone = "${var.us-east-zones[count.index]}"
  ami               = "ami-66506c1c"
  instance_type     = "t2.micro"
}

resource "aws_instance" "west_backend" {
  tags = {
    Name = "${join("-", list(var.environment-name, "backend"))}"
  }

  provider          = "aws.us-west-1"
  ami               = "ami-07585467"
  count             = "${var.multi-region-deployment ? 2 : 0}"
  availability_zone = "${var.us-west-zones[count.index]}"
  instance_type     = "t2.micro"
}

output "frontend_ip" {
  value = "${aws_instance.frontend.public_ip}"
}

output "backend_ips" {
  value = "${aws_instance.backend.*.public_ip}"
}

output "west_frontend_ip" {
  value = "${aws_instance.west_frontend.*.public_ip}"
}

output "west_backend_ips" {
  value = "${aws_instance.west_backend.*.public_ip}"
}
```

###### LOCALS
- Assigned name expressions.  
- Use locals instead of variable when you need to use expressions and don't want to write again and again a lot of variables, by expressions it will have a logic to generate.  
Example:  
```
locals {
  instance_ids = "${concat(aws_instance.blue.*id), aws_instance.green.*.id}"
}

locals { 
  default_name_prefix = "${var.project_name}-web"
  name_prefix = "${var.name_prefix != "" ? var.name_prefix : local.default_name_prefix}"
}

resource "aws_s3_bucket" "files" {
bucket = "${local.name_prefix}-files"
}
```
Demo:
```
provider "aws"  {
  access_key = "ACCESS_KEY"
  secret_key = "SECRET_KEY"
  region = "eu-central-1"
}

provider "aws"  {
  alias = "paris"
  access_key = "ACCESS_KEY"
  secret_key = "SECRET_KEY"
  region = "eu-west-3"
}

variable "multi-site" {
default = false
}

variable "def-reg-az" {
  default = [ "eu-central-1a", "eu-central-1b" ]
}

variable "paris-reg-az" {
  default = [ "eu-west-1a", "eu-west-1b" ]
}

variable "tag-name" {
  default = "demo-server"
}

locals {
  backend-tag = "${join("-",list(var.tag-name, "backend"))}"
  frontend-tag = "${join("-",list(var.tag-name, "frontend"))}"
}

resource "aws_instance" "frontend" {
  count = 2
  tags {
    Name = "${local.frontend-tag}"
  }
  ami = "ami-086a09d5b9fa35dc7"
  instance_type = "t2.micro"
  availability_zone = "${var.def-reg-az[count.index]}"
  root_block_device {
    volume_type = "gp2"
    volume_size = "8"
  }
  depends_on = ["aws_instance.backend"]
  timeouts {
  create = "5m"
  delete = "5m"
  }
  lifecycle {
  create_before_destroy = true
  }
}

resource "aws_instance" "backend" {
  count = 2
  tags {
    Name = "${local.backend-tag}"
  }
  ami = "ami-086a09d5b9fa35dc7"
  instance_type = "t2.micro"
  availability_zone = "${var.def-reg-az[count.index]}"
  timeouts {
  create = "5m"
  delete = "5m"
  }
  lifecycle {
  prevent_destroy = true
  }
}

resource "aws_instance" "frontend-paris" {
  count = "${var.multi-site ? 2 : 0}"
  provider = "aws.paris"
  tags {
    Name = "${local.frontend-tag}"
  }
  ami = "ami-075b44448d2276521"
  instance_type = "t2.micro"
  availability_zone = "${var.paris-reg-az[count.index]}"
  depends_on = ["aws_instance.backend-paris"]
  timeouts {
  create = "5m"
  delete = "5m"
  }
  lifecycle {
  create_before_destroy = true
  }
}

resource "aws_instance" "backend-paris" {
  count = "${var.multi-site ? 2 : 0}"
  provider = "aws.paris"
  tags {
    Name = "${local.backend-tag}"
  }
  ami = "ami-075b44448d2276521"
  instance_type = "t2.micro"
  availability_zone = "${var.paris-reg-az[count.index]}"
  timeouts {
  create = "5m"
  delete = "5m"
  }
  lifecycle {
  prevent_destroy = true
  }
}

output "frontend" {
  value = "${aws_instance.frontend.*.public_ip}"
}

output "backend" {
  value = "${aws_instance.backend.*.public_ip}"
}

output "frontend-paris" {
  value = "${aws_instance.frontend-paris.*.public_ip}"
}

output "backend-paris" {
  value = "${aws_instance.backend-paris.*.public_ip}"
}
```

###### DATA SOURCES: data
- Read only information outside of Terraform.  
- Example: Get list of ip ranges of ec2 service filter by 2 regions. Later you can use this data for firewall rules etc.    
```
data "aws_ip_ranges" "european-ec2" {
  regions = ["eu-central-1", "eu-west-3"]
  services = ["ec2"]
}
```

- Getting the data from a template file.  
- Teamplate file interpolation syntax is same.  
Defining:  
```
data "template_file" "setup" {
  template = "${file("config/setup.sh")}"
}
```
Using:  
- (TYPE.NAME.ATTR)  
```
resource "aws_instance" "web" {
ami = "${data.aws_ami.web.id}"
instance_type = "t2.micro"
}
```

- Meta-parameters & Multiplr Provider Instances:  
  - Maintain the same meta-marameters of resources except lifecycle configuration block.  
  - Can also be used with multiple aliased instances of the same provider.
- Example:  
```
data "aws_ami" "web" {
provider = "aws.west"
}
```
- Demo:
```
provider "aws" {
access_key = "ACCESS_KEY"
secret_key = "SECRET_KEY"
region = "eu-central-1"
}

data "aws_availability_zones" "us-central-1" {}

provider "aws" {
access_key = "ACCESS_KEY"
secret_key = "SECRET_KEY"
region = "eu-west-3"
alias = "paris"
}

data "aws_ip_ranges" "european-ec2" {
  regions = ["eu-central-1", "eu-west-3"]
  services = ["ec2"]
}

data "aws_availability_zones" "eu-central-1" {}

data "aws_availability_zones" "paris-region" {
  provider = "aws.paris"
}

resource "aws_instance" "frontend" {
  count             = 2
  availability_zone = "${data.aws_availability_zones.eu-central-1.names[count.index]}"
  ami               = "ami-086a09d5b9fa35dc7"
  instance_type     = "t2.micro"
}

resource "aws_instance" "frontend-paris" {
  count             = 2
  provider          = "aws.paris"
  ami               = "ami-075b44448d2276521"
  availability_zone = "${data.aws_availability_zones.paris-region.names[count.index]}"
  instance_type     = "t2.micro"
}

output "frankfurt-az" {
value = "${data.aws_availability_zones.us-central-1.names}"
}

output "paris-az" {
value = "${data.aws_availability_zones.paris-region.names}"
}

output "zone-ips" {
  value = ["${data.aws_ip_ranges.european-ec2.cidr_blocks}"]
}
```

#### MODULES
- Self-contained packages of configurations used to create reusable Terraform elements to help your code organized and also managing as a group.  
- Usage: You can use third party modules or local modules.  
- When using modules, you define a source and name.  
- Third party modules are installed from Terraform and installed by `terraform init` command.  
- https://registry.terraform.com/  
- Important to check modules that are verified by `HashiCorp`  

Source for modules:
- Mericurial repositories. `hg::` in the beginning.    
- HTTP URLs  
- S3 buckets  
- Git repository  

Including a module:
```
module "child" {
source = "./child"
}
```

Tree of a complete-module:  
- README.md  
- variables.tf  
- outputs.tf  
  - modules/
     - nestedA/
       - README.md
       - LICENSE.md
       - variables.tf
       - main.tf
       - outputs.tf
     - nestedB
     - ...../
  - examples/
    - exampleA/
      - main.tf
    - exampleB/
      - .../

###### MODULE EXAMPLE
Config: modules.tf
```
module "frontend" {
  source = "./aws_instances"
  aws_access_key = "${var.aws_access_key}"
  aws_secret_key = "${var.aws_secret_key}"
}

module "backend" {
  source = "./aws_instances"
  total_instances = 2
  aws_access_key = "${var.aws_access_key}"
  aws_secret_key = "${var.aws_secret_key}"
}

output "frontnend_ips" {
  value = "${module.frontend.ips}"
}

output "backend_ips" {
  value = "${module.backend.ips}"
}
```
Config: variables.tf
```
variable "aws_access_key" {}
variable "aws_secret_key" {}
```
Config: aws_instances/main.tf
```
data "aws_availability_zones" "available" {}

resource "aws_instance" "instance" {
  count                 = "${var.total_instances}"
  ami                   = "${var.amis[var.region]}"
  instance_type         = "t2.micro"
  availability_zone     = "${data.aws_availability_zones.available.names[count.index]}"
}
```
Config: outputs.tf
```
output "ips" {
  value = "${aws_instance.instance.*.public_ip}"
}
```
Config: aws_instances/providers.tf
```
provider "aws" {
  access_key = "${var.aws_access_key}"
  secret_key = "${var.aws_secret_key}"
  region     = "${var.region}"
}
```
Config: aws_instances/variables.tf
```
variable "amis" {
  type = "map"
  default = {
    us-east-1 = "ami-66506c1c"
    us-west-1 = "ami-07585467"
  }
}

variable "region" {
  default="us-east-1"
}

variable "total_instances" {
  default=1
}

variable "aws_access_key" {}
variable "aws_secret_key" {}
```

#### Backends and Remote State, Proviles and Credentials  
- `Backend` is local for large teams, you need to use SCM and put to git.   
- To keep terraform state `terraform.tfstate` need to set up a remote backend instead of local for collaboration, CI and CD.  
- Backend Types:  artifactory, azurem, consul, etcd, etcdv3, gcs, http, manta, swift, terraform enterprise.  
Setting up `Backend` at `S3`:  
- First create a S3 bucket.   
```
terraform {
  backend "s3" }
  bucket = "my-terraform-bucket"
  key = "network/terraform.tfstate"
  region = "eu-central-1"
  }
}
```
Setting up `Backend` at `Consul`:  
- https://blog.codeship.com/terraform-remote-state-with-consul-backend/  
```
terraform {
  backend "consul" {
    address = "demo.consul.io"
    scheme  = "https"
    path    = "example_app/terraform_state"
  }
}
```
###### Profiles and Credentials
- Inside the terraform module dir that we are working.  
```
mkdir .aws
```
Config: .aws/config
```
[default]
region = us-east-1

[profile caylentProd]
region = us-east-1
```
Config: .aws/credentials
```
[caylentProd]
aws_access_key_id = my_access_key
aws_secret_access_key = my_secret_key

[default]
aws_access_key_id = my_access_key
```
```
export AWS_PROFILE="caylentProd"
terraform init
```

#### WORKSPACES
- A named container to keep states and rename state file meant to keep in a shared resource.  
- Multiple workspace are available with these backends: AzureRM, Consul, GCS, Local, Manta.  
- Example you can create workspace for `production`, `staging` etc.  The first one is default.  
- You cannot delete wotkspaces.  
- Creating a workspace  
- Use workspaces to deal with small differences between development, staging, production.  
- Make it manageable and safe to split a large configuration into smaller ones.  
```
terraform workspace new staging
terraform workspace list
```
- Will create a folder `terraform.tfstate.d/staging`
- Interpolation in `tf` files:  ${terraform_remote_state}
Demo: Use workspaces and change tags according to namespaces  
```
provider "aws" {
  region = "us-east-1"
}

locals {
  default_name = "${join("-", list(terraform.workspace, "example"))}"
}

resource "aws_instance" "example" {
  tags = {
    Name = "${local.default_name}"
  }

  ami           = "ami-2757f631"
  instance_type = "t2.micro"
}
```

#### SOFTWARE PROVISIONING WITH PROVISIONERS
- Software Provisioner: help executing scripts on a local or remote machine.  Many provisioners require access to the remote resource `SSH` port `22` or `WINRM` port `5985`. Any connection information used in a resource applies to all provisioners.  
  - Userdata  
  - Remote exec
  - Local exec  
  - Ansible, Chef, Puppet like `CM` tools  
- Connection arguments:  
  - Private_key  
  - User  
  - Password  
  - Host  
  - Port  
  - Timeout  
- File Provisioner: Is a type of provisioner to transfer files to the servers by same protocols.  
- When you create a `local-exec` or `remote-exec` provisioner. You will use `command` or `inline` arguement. `command` is one line of code. `inline` is for a block of a code in list format. `local-exec` uses `command` and don't know `inline`. `remote-exec` knows `inline`.  

DEMO:  
- Create some files to execute on remote and configure. The first one a php script.    
```
mkdir frontend
echo "HELLOWORLD" > frontend/index.php
echo "<?php phpinfo(); ?>" > frontend/index.php
# Create Script
vi frontend/run_frontend.sh
```
Script To Run: frontend/run_frontend.sh  
```
#!/bin/bash

apt update
apt install -y php php-fpm nginx
systemctl restart php7.0-fpm.service
systemctl restart nginx.service
mkdir /var/www/frontend/
cp frontend.conf /etc/nginx/conf.d/frontend.conf
rm -rf /etc/nginx/sites-available/default
rm -rf /etc/nginx/sites-enabled/default
rm -rf /etc/nginx/conf.d/default
cp -a ./ /var/www/frontend/
nginx -s reload
```
```
vi frontend/frontend.conf
```
NGINX Config: frontend/frontend.conf  
```
server {
    listen         80 default_server;
    listen         [::]:80 default_server;
    server_name    localhost;
    root           /var/www/frontend;
    index          index.php;

  location ~* \.php$ {
    fastcgi_pass unix:/run/php/php7.0-fpm.sock;
    include         fastcgi_params;
    fastcgi_param   SCRIPT_FILENAME    $document_root$fastcgi_script_name;
    fastcgi_param   SCRIPT_NAME        $fastcgi_script_name;
  }
}
```
Terraform : terraform.tf
```
provider "aws" {
access_key = "ACCESS_KEY"
secret_key = "SECRET_KEY"
region = "eu-central-1"
}

variable "pvt_key" {
  default = "~/.ssh/test_key.pem"
}

resource "aws_security_group" "for_nginx" {
  name = "Nginx ssh/http"
  description = "nginx ssh and http access"
  ingress {
    from_port = "80"
    to_port = "80"
    protocol = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }
  ingress {
    from_port = "22"
    to_port = "22"
    protocol = "tcp"
    cidr_blocks  = ["0.0.0.0/0"]
  }
  egress {
    protocol = "-1"
    from_port = "0"
    to_port = "0"
    cidr_blocks = ["0.0.0.0/0"]
  }
  tags {
    Name = "Nginx ssh/http"
  }
}

resource "aws_instance" "nginx" {
  count = "1"
  depends_on = ["aws_security_group.for_nginx"]
  ami = "ami-086a09d5b9fa35dc7"
  vpc_security_group_ids = ["${aws_security_group.for_nginx.*.id}"]
  instance_type = "t2.micro"
  key_name = "JenkinsOnUbuntu"
  tags {
  Name = "NGINX"
  }

  connection {
    user = "ubuntu"
    type = "ssh"
    private_key = "${file(var.pvt_key)}"
    }

  provisioner "file" {
  source = "./frontend"
  destination = "~/"
  }

  provisioner "remote-exec" {
    inline = [
      "chmod +x frontend/run_frontend.sh",
      "cd frontend",
      "sudo ~/frontend/run_frontend.sh",
    ]
  }
}


output "sg_id" {
  value = "${aws_security_group.for_nginx.*.vpc_id}"
}

output "vpc_id" {
  value = ["${aws_security_group.for_nginx.*.id}"]
}

output "pub_ip" {
  value = "${aws_instance.nginx.*.public_ip}"
}

```
Run:  
```
terraform init
terraform validate
terraform show
```
- Get the ip and browse to the IP and check your site.  

#### SOFTWARE PROVISIONING WITH ANSIBLE
- Need to learn ansible as it's a different `CM` tools.  
- Terraform will build the servers and ansible will configure.  
- Let's do the same we did by `remote-exec` with ansible and `local-exec`.    
- We use ubuntu. Create a `sh` file that installs ansible.   
File: scripts/install_ansible_git.sh  
```
#!/bin/bash
sudo apt-get update
sudo apt-get install software-properties-common
sudo apt-add-repository ppa:ansible/ansible
sudo apt-get update
sudo apt-get -y install ansible git
```
Source Code File: frontend/index.php
```
mkdir frontend
echo "HELLOWORLD" > frontend/index.php
echo "<?php phpinfo(); ?>" > frontend/index.php
```
File: ansible-configure.yml
```
---
- hosts: all
  remote_user: ubuntu
  gather_facts: no
  become: true

  tasks:
    - name: update cache
      raw:  apt-get update
    - name: install python
      raw:  apt-get -y install python-simplejson
    - name: Install required packages update_cache=yes
      apt: name={{item}} state=present
      with_items:
        - php
        - php-fpm
        - nginx
    - name: start/enable php service
      service: name=php7.0-fpm state=started enabled=yes
    - name: start/enable nginx service
      service: name=nginx state=started enabled=yes
    - name: copy source
      copy: src=/home/ubuntu/frontend dest=/var/www/
      notify: restart nginx
    - name: delete deafult available site
      file: path=/etc/nginx/sites-available/default state=absent
      notify: restart nginx
    - name: delete default enabled site
      file: path=/etc/nginx/sites-enabled/default state=absent
      notify: restart nginx

  handlers:
    - name: restart nginx
      service: name=nginx state=restarted
```
Config: terraform.tf
```
provider "aws" {
access_key = "ACCESS_KEY"
secret_key = "SECRET_KEY"
region = "eu-central-1"
}

variable "ssh_key" {
  default = "/home/ubuntu/.ssh/test_key.pem"
}

data "aws_availability_zones" "available_az" {}

resource "aws_security_group" "nginx-sg" {
  name = "nginx-name"
  description = "ssh http inbound and all out bound for nginx server"
  ingress {
    from_port = "22"
    to_port = "22"
    protocol = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }
  ingress {
  from_port = "80"
  to_port = "80"
  protocol = "tcp"
  cidr_blocks = ["0.0.0.0/0"]
  }
  egress {
  from_port = "0"
  to_port = "0"
  protocol = "-1"
  cidr_blocks = ["0.0.0.0/0"]
  }
}

resource "aws_instance" "frontend" {
  count = 1
  ami = "ami-086a09d5b9fa35dc7"
  instance_type = "t2.micro"
  vpc_security_group_ids = ["${aws_security_group.nginx-sg.*.id}"]
  availability_zone = "${data.aws_availability_zones.available_az.names[count.index]}"
  key_name = "JenkinsOnUbuntu"

  tags {
    Name = "NGINX"
  }

  connection {
    user        = "ubuntu"
    type        = "ssh"
    private_key = "${file(var.ssh_key)}"
  }

  provisioner "remote-exec" {
    # The connection will use the local SSH agent for authentication
    inline = ["echo Successfully connected"]

  }
}

resource "null_resource" "assign_run_permission" {
  provisioner "local-exec" {
    command = "chmod +x scripts/install_ansible_git.sh"
  }
}

resource "null_resource" "install_ansible" {
  provisioner "local-exec" {
    command = "scripts/install_ansible_git.sh"
  }
}

resource "null_resource" "ansible-provision" {
  depends_on = [ "aws_instance.frontend" ]
  provisioner "local-exec" {
    command = "ansible-playbook --private-key=${var.ssh_key}  --ssh-common-args='-o StrictHostKeyChecking=no' -i '${aws_instance.frontend.public_ip},' ansible-configure.yml"
  }
}

output "frontend_ip" {
  value = "${aws_instance.frontend.public_ip}"
}

```

#### AWS ON TERRAFORM:  
- Terraform can deploy and configure any service that `AWS` has.  
- In this section the services below will be discussed.  
  - VPC  
  - EBS  
  - IAM  
  - Route53  
  - Autoscaling  
  - AWS RDS  
  - ELB  
  - Lambda  
  - Real-Life Examples  

###### VPC  
VPC:  VPCs are isolated virtual private networks in cloud. VPCs are used and configured by adjusting IP range, subnets, route tables, network gateways and security settings for isolating instance on a network level. It can be public or private.  
Some components:  
- `Internet Gateways`:  For providing inbound internet access.  
- `NAT Gateways`:  For providing outbound traffic to private subnets. Possible to attach elastic IP.  
- `Routes and Route Tables`: Route tables connected to your public subnet, as well as custom route tables allows usage of Internet Gateways and NAT gateways for inbound and outbound traffic.  
- `NACLs`:  Osi 2 layered, stateless, order listed, network layer security solution. It can do also deny.  
- `Security Groups`:  For allowing traffic. Object firewall. Osi 3+, firewall for EC2 instances.  There is no deny, only allow.    
- `Subnets`:  VPC subnets must and can be linked to one route table. You can link multiple subnets to the same route table.  
Example:  
```
Creating one VPC with a cidr block, 2 public subnets for webservers separated each in a different availability zone and 2 private subnets separated each in a different availability zone and setting up an internet gateway for internet access.
```

- Create a VPC by using `module` in code.  
- Modulel Usage Page: https://registry.terraform.io/modules/terraform-aws-modules/vpc/aws/1.46.0  
- If you need to read the codes and get mode information, you can read the source code by clicking the link in the module page.  Also it may be a good place to learn how to do by plugin and write your own code or module.  
- https://github.com/terraform-aws-modules/terraform-aws-vpc  
```
provider "aws" {
access_key = "ACCESS_KEY"
secret_key = "SECRET_KEY"
region = "eu-central-1"
region = "eu-central-1"
}

resource "aws_volume_attachment" "ebs_att_test" {
  device_name = "/dev/sdh"
  volume_id = "${aws_ebs_volume.test.id}"
  instance_id = "${aws_instance.web.id}"
}

resource "aws_instance" "web" {
  availability_zone = "eu-central-1a"
  ami = "ami-086a09d5b9fa35dc7"
  instance_type = "t2.micro"
  tags {
  Name = "Web"
  }
}

resource "aws_ebs_volume" "test" {
  availability_zone = "eu-central-1a"
  size = 7
  type = "gp2"
  tags {
  Name = "test-volume"
  }
}
```

###### EBS
- Every instance has a root EBS volume. Elastic block storage. Good practice is keeping your data in an additional EBS volume, so if your instance will be deleted, your data is kept if you set not to delete with the instance option. You can also take snapshots to EBS's and use in clusters.  

```
provider "aws" {
access_key = "ACCESS_KEY"
secret_key = "SECRET_KET"
region = "eu-central-1"
}

resource "aws_ebs_volume" "test" {
  availability_zone = "eu-central-1a"
  size = 7
  type = "gp2"
  tags {
  Name = "test-volume"
  }
}
```
 
###### IAM
- Identity & Access Management: A directory service designed for trackins system users, provide identity management and access permit and deny action on resources. Can configure shared access by delegating permissions. Multi factor authentication can be setup. Cloud security can be centralized and standardized. Temporary access, control the type of operations, access across accounts, all possible.   
  - IAM Roles:  A pack of policies.  
  - IAM Policies: Predefined permissions.  
  - IAM Groups:  A pack of identities like users.  
- You can assign policies to groups, users and roles.  
Demo: Create a user,group and policy attached together.  
File: variables.tf
```
variable "path" {
description = "Path in which to create the user."
default = "/"
}


variable "force_destroy" {
description = "Delete non-terraform managed IAM keys, login profiles and MFA."
default = true
}
```
Config: terraform.tf
```
provider "aws" {
  access_key = "ACCESS_KEY"
  secret_key = "SECRET_KEY"
  region = "eu-central-1"
}


resource "aws_iam_user" "user" {
  name          = "stefan-tf-demo"
  path          = "${var.path}"
  force_destroy = "${var.force_destroy}"
}

resource "aws_iam_group_membership" "team" {
  name = "tf-testing-group-membership"

  users = [
    "${aws_iam_user.user.name}",
  ]

  group = "${aws_iam_group.group.name}"
}

resource "aws_iam_group" "group" {
  name = "test-group"
}

resource "aws_iam_group_policy" "policy" {
  name  = "policy"
  group = "${aws_iam_group.group.id}"

  policy = <<EOF
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Action": [
        "ec2:Describe*"
      ],
      "Effect": "Allow",
      "Resource": "*"
    }
  ]
}
EOF
}

#resource "aws_iam_group_policy_attachment" "test-attach" {
#  group      = "${aws_iam_group.group.name}"
#  policy_arn = "${aws_iam_group_policy.policy.arn}"
#}
```

###### ROUTE53
- Highly available and scalable Domain Name System (DNS). Allows to map domain names to instances and S3. Aliases are different than CNAMEs and extends DNS functionality like you can map to a loadbalancer.
Demo: Creating a zone and adding a record.  
Config: Terraform    
```
provider "aws" {
  access_key = "ACCESS_KEY"
  secret_key = "SECRET_KEY"
  region = "eu-central-1"
}

resource "aws_route53_zone" "primary" {
name = "kops.lonynamer.com"
}

resource "aws_route53_record" "www" {
  zone_id = "${aws_route53_zone.primary.zone_id}"
  name = "www.kops.lonynamer.com"
  type = "A"
  records = ["8.8.8.8"]
  ttl = "300"
}

```

###### AUTOSCALING
- Build automatic scaling plans for infrastructure resources holding a collections of similar characteristics instances to build resilient  application infrastructure.  
- You need to define 2 resources;  
  - Launch config: propertices of instances.  
  - Autoscaling group and plicies by cloud watch alarms.  - Reacts well to load changes and scales, cost saving and supports work loads.  

- You can use spot instances as well.  
- Manual launch configuration is like creating an instance.  
- Lifecycle hooks: Tou run, notify or call something when autoscaling works.  
Demo: Create a autoscaling group, launch configuration including and instance installed nginx with loadbalancer.  
Config: terraform.tf  
```
provider "aws" {
  access_key = "ACCESS_KEY"
  secret_key = "SECRET_KEY"
  region = "eu-central-1"
}

# Data sources to get VPC, subnets and security group details
data "aws_vpc" "default" {
  default = true
}

data "aws_subnet_ids" "all" {
  vpc_id = "${data.aws_vpc.default.id}"
}

data "aws_security_group" "default" {
  vpc_id = "${data.aws_vpc.default.id}"
  name = "default"
}

data "aws_ami" "amazon_linux" {
  most_recent = true

  filter {
    name = "name"
    values = [
      "amzn-ami-hvm-*-x86_64-gp2",
    ]
  }

  filter {
    name = "owner-alias"
    values = [
      "amazon",
    ]
  }
}


module "example_asg" {
  source = "terraform-aws-modules/autoscaling/aws"
  name = "example-with-elb"

# Launch configuration
# create_lc = false  # disables the creation of lc, so you use the created.
  lc_name = "example-lc"
  image_id = "${data.aws_ami.amazon_linux.id}"
  instance_type = "t2.micro"
  key_name = "JenkinsOnUbuntu"
  user_data = <<EOF
    #!/bin/bash
    yum -y install nginx
    service nginx start
    chkconfig nginx on
  EOF
  security_groups = ["${data.aws_security_group.default.id}"]
  load_balancers = ["${module.elb.this_elb_id}"]

  # Block Devices
  ebs_block_device = [
    {
      device_name = "/dev/xvdz"
      volume_type = "gp2"
      volume_size = "8"
      delete_on_termination = true
    },
  ]

  root_block_device = [
    {
      volume_size = "50"
      volume_type = "gp2"
    },
  ]

# Auto scaling group
  asg_name = "example-asg"
  vpc_zone_identifier = ["${data.aws_subnet_ids.all.ids}"]
  health_check_type = "EC2"
  min_size = 1
  max_size = 3
  desired_capacity = 1
  wait_for_capacity_timeout = 0
  tags = [
    {
      key = "Environment"
      value = "Production"
      propagate_at_launch = true
    },
    {
      key = "Team"
      value = "Infra"
      propagate_at_launch = true
    },
  ]
}

# Load Balancer
  module "elb" {
    source = "terraform-aws-modules/elb/aws"
    name = "elb-example"
    subnets = ["${data.aws_subnet_ids.all.ids}"]
    security_groups = ["${data.aws_security_group.default.id}"]
    internal = false

    listener = [
      {
        instance_port = "80"
        instance_protocol = "HTTP"
        lb_port = "80"
        lb_protocol = "HTTP"
      },
    ]

    health_check = [
      {
        target = "HTTP:80/"
        interval = 30
        healthy_threshold = 2
        unhealthy_threshold = 2
        timeout = 5
      },
    ]

   tags = [
     {
       key = "Owner"
       valuer = "Lony Namer"
     },
   ]
  }
```

###### RDS
- RDS helps to build, run and scale databases through the AWS managed services. Allows you to spin up multiple relational database engines in the cloud at the same time.  
  - Aurora: Mysql or Postgresql type  
  - Mysql  
  - MariaDB  
  - Oracle  
  - Microsoft SQL  
  - PostgreSQL  
- RDS maintains many standard database management tasks for you.  
- Cost effective and resizable.  
- Helps predict impending issues or capacity constrains before performance or availability is affected.  

Upgrade Options:
- Upgrades can be done automatically and there are 2 types.  
  - Minor Versions:  
  - Major Versions:  
- Some resources:  
- https://www.terraform.io/docs/providers/aws/d/db_instance.html  
- https://github.com/terraform-aws-modules/terraform-aws-rds  
- https://registry.terraform.io/modules/terraform-aws-modules/rds/aws/1.0.2  

Maintenance Windows:  
- A chance to control when modifications and patching occur.  
- DB instances and DB clusters have weekly maintenance windows for system changes.  
- If needed maintanence windows are completed in 30 minute slots which you assign or are slotted in at random.  

High-Availability Options:
- HA provides failover support through multi-Availability Zone deployment.  
- Running DB instances with HA, enhances availability during planned system maintenance. Safeguards your databases against DB instance failure and AZ disruption.  
- Database size is resizable, as much as space you have more IOPS you will have, in a real production environment start with minimum 200 GB due to this reason.  
- AWS offer recovery resolution to every 1 minute in time, best practise use 35 days retention period.  

Demo: Creating a database by plugin `aws_db_instance`  
Config: terraform.tf  
```
provider "aws" {
  access_key = "ACCESS_KEY"
  secret_key = "SECRET_KEY"
  region = "eu-central-1"
}

data "aws_vpc" "default" {
  default = true
}

data "aws_subnet_ids" "all" {
  vpc_id = "${data.aws_vpc.default.id}"
}

resource "aws_db_subnet_group" "default" {
  name        = "default-mysql5-7"
  description = "default-mysql"
  subnet_ids  = ["${data.aws_subnet_ids.all.ids}"]

  tags {
    Name = "My DB subnet group"
  }
  lifecycle {
    create_before_destroy = true
  }
}

resource "aws_db_parameter_group" "default" {
  count = 1
  name = "default-mysql5-7"
#  name_prefix = "mysql"
  description = "Default parameter group for mysql5.7"
  family      = "mysql5.7"
  parameter = [
      {
        name = "character_set_client"
        value = "utf8"
      },
      {
        name = "character_set_server"
        value = "utf8"
      }
    ]
  tags = {
    Name = "My DB parameter group"
  }
  lifecycle {
    create_before_destroy = true
  }
}

resource "aws_db_option_group" "this" {
  count = 1
  name = "default-mysql5-7"
  option_group_description = "Default option group for mysql 5.7"
  engine_name              = "MySQL"
  major_engine_version     = "5.7"
  #option = ["${var.options}"]
  lifecycle {
    create_before_destroy = true
  }
}

resource "aws_security_group" "mysql" {
  name = "mysql"
  description = "aurora expose 3306"
  ingress {
    from_port = "3306"
    to_port = "3306"
    protocol = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }
  egress {
  from_port = "0"
  to_port = "0"
  protocol = "-1"
  cidr_blocks = ["0.0.0.0/0"]
  }
}

resource "aws_db_instance" "default" {
  identifier                = "demodb"
  engine                    = "mysql"
  engine_version            = "5.7.23"
  instance_class            = "db.t2.micro"
#  publicly accessible       = true
  storage_type              = "gp2"
  allocated_storage         = 20
  storage_encrypted         = false
# kms_key_id                = "arm:aws:kms:<region>:<accound id>:key/<kms key id>"
  name                      = "demodb"
  username                  = "user"
  password                  = "pass-1234"
  port                      = "3306"
  vpc_security_group_ids    = ["${aws_security_group.mysql.id}"]
  maintenance_window        = "Mon:00:00-Mon:03:00"
  backup_window             = "03:00-06:00"
# disable backups to create DB faster
  backup_retention_period   = 35
  parameter_group_name      = "default.mysql5.7"
  skip_final_snapshot       = true
# Snapshot name upon DB deletion
  final_snapshot_identifier = "demodb"
# DB subnet group
  db_subnet_group_name      = "default-mysql5-7"
# DB parameter group
  parameter_group_name      = "default-mysql5-7"
# DB option group
  option_group_name      = "default-mysql5-7"
  tags = {
    Owner       = "user"
    Environment = "dev"
  }
}
```

Demo: Creating a database by plugin `module rds`  
Config: terraform.tf  
```
provider "aws" {
  region = "eu-west-1"
}

##############################################################
# Data sources to get VPC, subnets and security group details
##############################################################
data "aws_vpc" "default" {
  default = true
}

data "aws_subnet_ids" "all" {
  vpc_id = "${data.aws_vpc.default.id}"
}

data "aws_security_group" "default" {
  vpc_id = "${data.aws_vpc.default.id}"
  name   = "default"
}

#####
# DB
#####
module "db" {
  source = "terraform-aws-modules/rds/aws"

  identifier = "demodb"

  # All available versions: http://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/CHAP_MySQL.html#MySQL.Concepts.VersionMgmt
  engine            = "mysql"
  engine_version    = "5.7.19"
  instance_class    = "db.t2.micro"
  allocated_storage = 5
  storage_encrypted = false

  # kms_key_id        = "arm:aws:kms:<region>:<accound id>:key/<kms key id>"
  name     = "demodb"
  username = "user"
  password = "YourPwdShouldBeLongAndSecure!"
  port     = "3306"

  vpc_security_group_ids = ["${data.aws_security_group.default.id}"]

  maintenance_window = "Mon:00:00-Mon:03:00"
  backup_window      = "03:00-06:00"

  # disable backups to create DB faster
  backup_retention_period = 35

  tags = {
    Owner       = "user"
    Environment = "dev"
  }

  # DB subnet group
  subnet_ids = ["${data.aws_subnet_ids.all.ids}"]

  # DB parameter group
  family = "mysql5.7"

  # Snapshot name upon DB deletion
  final_snapshot_identifier = "demodb"
}
```

###### LOADBALANCING
- Balancing, dstributing incoming traffic and scales resources to meet traffic demands.  Layer 3 (network) or Layer 7 (application) balancing possible, detects health.  
- You can assign security group and attach "SSL" certs for your site adding HTTPS.  
- You can configure health check.  
- Type:  
  - Application LBs:  Application layer and features for applications. Also provides more monitoring. You need to use target groups.  
  - Classic LBs:  Network layer and multi AZs.  
  - Network LBs:  
- Attachments: Attaching instances or resources.  
Demo: Creating an instance with classic load balancer and attaching the instance.  
Config: terraform.tf  
```
provider "aws" {
  access_key = "ACCESS_KEY"
  secret_key = "SECRET_KEY"
  region = "eu-central-1"
}

variable "number_of_instances" {
  description = "Number of instances to create and attach to elb."
  default = 1
}

data "aws_vpc" "default" {
  default = true
}

data "aws_subnet_ids" "all" {
  vpc_id = "${data.aws_vpc.default.id}"
}

resource "aws_security_group" "webservice" {
  name = "webservice"
  description = "expose 80"
  ingress {
    from_port = "80"
    to_port = "80"
    protocol = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }
  ingress {
    from_port = "8080"
    to_port = "8080"
    protocol = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }
  ingress {
    from_port = "22"
    to_port = "22"
    protocol = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }
  egress {
  from_port = "0"
  to_port = "0"
  protocol = "-1"
  cidr_blocks = ["0.0.0.0/0"]
  }
}

# Classic ELB by module
module "elb" {
  source = "terraform-aws-modules/elb/aws"
  name = "elb-example"
  subnets = ["${data.aws_subnet_ids.all.ids}"]
  security_groups = [ "${aws_security_group.webservice.id}" ]
  internal = false
  listener = [
    {
      instance_port = "80"
      instance_protocol = "HTTP"
      lb_port = "80"
      lb_protocol = "HTTP"
    },
    {
      instance_port = "8080"
      instance_protocol = "HTTP"
      lb_port = "8080"
      lb_protocol = "HTTP"
    },
  ]
  health_check = [
    {
      target = "HTTP:80/"
      interval = 30
      healthy_threshold = 2
      unhealthy_threshold = 2
      timeout = 5
    },
  ]

# ELB ATTACHMENTS
  number_of_instances = "${var.number_of_instances}"
  instances = ["${module.ec2_instances.id}"]
  tags = {
    Owner = "user"
    Environment = "dev"
  }
}

# Instances
module "ec2_instances" {
  source = "terraform-aws-modules/ec2-instance/aws"
  name = "myapp"
  instance_count = "${var.number_of_instances}"
  instance_type = "t2.micro"
  ami = "ami-086a09d5b9fa35dc7"
  subnet_id = "${element(data.aws_subnet_ids.all.ids, 0)}"
  vpc_security_group_ids = ["${aws_security_group.webservice.id}"]
  key_name = "JenkinsOnUbuntu"
  associate_public_ip_address = true
  user_data = <<EOF
#!/bin/bash
apt-get update
apt-get -y install nginx
service nginx start

EOF
}
```

Demo: Creating an a`pplication loadbalancer`, `target group`, `attachment to target group`, `listener` and `instances` to attach to target group.  
Config: terraform.tf    
```
provider "aws" {
  access_key = "ACCESS_KEY"
  secret_key = "SECRET_KEY"
  region = "eu-central-1"
}

variable "number_of_instances" {
  description = "Number of instances to create and attach to elb."
  default = 1
}

data "aws_vpc" "default" {
  default = true
}

data "aws_subnet_ids" "all" {
  vpc_id = "${data.aws_vpc.default.id}"
}

resource "aws_security_group" "webservice" {
  name = "webservice"
  description = "expose 80"
  ingress {
    from_port = "80"
    to_port = "80"
    protocol = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }
  ingress {
    from_port = "8080"
    to_port = "8080"
    protocol = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }
  ingress {
    from_port = "22"
    to_port = "22"
    protocol = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }
  egress {
  from_port = "0"
  to_port = "0"
  protocol = "-1"
  cidr_blocks = ["0.0.0.0/0"]
  }
}

# Instances
resource "aws_instance" "myapp" {
  count = "${var.number_of_instances}"
  instance_type = "t2.micro"
  ami = "ami-086a09d5b9fa35dc7"
  subnet_id = "${element(data.aws_subnet_ids.all.ids, 0)}"
  vpc_security_group_ids = ["${aws_security_group.webservice.id}"]
  key_name = "JenkinsOnUbuntu"
  associate_public_ip_address = true
  user_data = <<EOF
#!/bin/bash
apt-get update
apt-get -y install nginx
service nginx start
EOF
  tags {
    name = "myapp"
  }
}

#ELB
resource "aws_lb" "front_end" {
  name = "front-end-lb"
  load_balancer_type = "application"
  security_groups = ["${aws_security_group.webservice.id}"]
  subnets = ["${data.aws_subnet_ids.all.ids}"]
  enable_deletion_protection = false
  tags {
    Name = "front_end-elb"
    Environment = "production"
  }
}

# ELB TARGET GROUP
resource "aws_lb_target_group" "front_end" {
  name = "front-end-lb-tg"
  port = 80
  protocol = "HTTP"
  vpc_id = "${data.aws_vpc.default.id}"
}

# ELB TARGET GROUP ATTACHMENT
resource "aws_lb_target_group_attachment" "front_end" {
  count = "${var.number_of_instances}"
  target_group_arn = "${aws_lb_target_group.front_end.arn}"
  target_id = "${element(aws_instance.myapp.*.id, count.index)}"
}

# ELB_LISTENER
resource "aws_lb_listener" "front_end" {
  load_balancer_arn = "${aws_lb.front_end.arn}"
  port = 80
  protocol = "HTTP"

  default_action {
    target_group_arn = "${aws_lb_target_group.front_end.arn}"
    type = "forward"
  }
}

output "all" {
  value = ["${aws_instance.myapp.*.id}"]
}

output "external_address" {
  value = "${aws_lb.front_end.dns_name}"
}
```


###### LAMBDA
- It's a serverless compute service that runs code in response to events without provisioning or managing servers. Lambda automatically manages the underlying compute resources for you.  

- Usage and Benefots of Lambda  
  - Smaller on-demand applications that are responsive to events and new information.  
  - Event-driven execute code only when needed.  
  - Scales automatically from a few requests a day to thousands per second. Only pay for time consumed.  
  - Supports versioning.  
- Languages Supported: C# (.Net Core), Java, NodeJS, Go, Phython, Go.  
Demo: Zip a python 3.6 file and upload to lambda during creation, get some policies and create an iam role for lambda function, create the lambda function.  
Python 3.6 code: hello_lambda.py
```
import os

def lambda_handler(event, context):
    return "{} from Lambda!".format(os.environ['greeting'])
```
Config: terraform.tf
```provider "aws" {
  access_key = "ACCESS_KEY"
  secret_key = "SECRET_KEY"
  region = "eu-central-1"
}

provider "archive" {}

data "archive_file" "zip" {
  type        = "zip"
  source_file = "hello_lambda.py"
  output_path = "hello_lambda.zip"
}

data "aws_iam_policy_document" "policy" {
  statement {
    sid    = ""
    effect = "Allow"

    principals {
      identifiers = ["lambda.amazonaws.com"]
      type        = "Service"
    }

    actions = ["sts:AssumeRole"]
  }
}

resource "aws_iam_role" "iam_for_lambda" {
  name               = "iam_for_lambda"
  assume_role_policy = "${data.aws_iam_policy_document.policy.json}"
}

resource "aws_lambda_function" "lambda" {
  function_name = "hello_lambda"

  filename         = "${data.archive_file.zip.output_path}"
  source_code_hash = "${data.archive_file.zip.output_sha}"

  role    = "${aws_iam_role.iam_for_lambda.arn}"
  handler = "hello_lambda.lambda_handler"
  runtime = "python3.6"

  environment {
    variables = {
      greeting = "Hello"
    }
  }
}
```

#### REAL LIFE EXAMPLE STACK



#### TERRAFORM USAGE WITH KUBERNETES IN GOOGLE COLUD (GCP)
- Kubernetes: Kubernetes is an open-source platform designed to automate deploying, scaling and operating application containers developed by Google. Nickname K8S and it's goal to foster an ecosystem of components and tools that releive the burden og running applications. Can orchestrate Dockers and Rocket containers.  
- Kubernetes, runs everywhere, extenible, open source and self-healing.  Leader, has massive community and market standard.  
- It covers all issues to run an application and micro services; Load balancing, storage, secrets, scaling, rolling updates, debugging, logging, monitoring.  
- `GKE` : GCP managed kubernetes services.  
- Nodepools:  Subset of machines defining local SSDs, CPU platform, Preemptible VMs, Node images, instance sizes, machine sizes. Custom nodeppols are great, if you need more resources in comparison to others.  

###### INSTALLING K8S ON GCP  
Prerequisities:  
- When you first register to GCP with your gmail, you will get 300 $ credit for testing.  
- Gcloud SDK installed locally and authenticated  
- Gcloud project and billing setup.  
- GKE api enabled. Enabling API means relating your project which you will create during GCP SDK installation, to a billing account.  Please do it after GCP SDK installation, Kubernetes Engines, select project from top and associate. This will enable the GKE API.  

- Creating a cluster K8S cluster https://github.com/lonynamer/Knowledge/tree/master/Terraform/terraform-tutorial/5c_Deploy_cluster  

File : install-gcp-skd.sh
```
# Create environment variable for correct distribution
export CLOUD_SDK_REPO="cloud-sdk-$(lsb_release -c -s)"

# Add the Cloud SDK distribution URI as a package source
echo "deb http://packages.cloud.google.com/apt $CLOUD_SDK_REPO main" | sudo tee -a /etc/apt/sources.list.d/google-cloud-sdk.list

# Import the Google Cloud Platform public key
curl https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -

# Update the package list and install the Cloud SDK
sudo apt-get update && sudo apt-get install google-cloud-sdk
```
```
chmod +x install-gcp-sdk.sh
sudo install-gcp-sdk.sh
```
Authenticate
```
gcloud init
# Follow the steps and create project with unique name.
gcloud projects list
```
```
Additionally create a service account with key and download the key as `account.json` one folder down where you run terraform codes.
```
```
mkdir -p kubernetes/k8s-module
cd kubernetes
```
Config: kube.tf  
```
module "kubernetes" {
  source = "./k8s-module"
  region = "us-east1"

  project_id_map = {
    default = "lony-terraform-demo"
  }
}

output "connection-command" {
  value = "${module.kubernetes.connect-string}"

```
Config: k8s-module/providers.tf  
```
provider "google" {
  project = "${var.project_id_map[terraform.workspace]}"
  region  = "${var.region}"
  credentials = "${file("../account.json")}"
  version = "1.8.0"
}
```
Config: k8s-module/variables.tf  
```
variable "project_id_map" {
  default = {
    default = "lony-terraform-lony"
  }

  type = "map"
}

variable "region" {}

variable "oauth_scopes" {
  type = "list"

  default = [
    "https://www.googleapis.com/auth/compute",
    "https://www.googleapis.com/auth/devstorage.read_only",
    "https://www.googleapis.com/auth/logging.write",
    "https://www.googleapis.com/auth/monitoring",
    "https://www.googleapis.com/auth/service.management",
    "https://www.googleapis.com/auth/servicecontrol",
  ]
}

variable "instance_size" {
  type = "map"

  default = {
    default = "n1-standard-1"
  }
}

variable "disk_size" {
  type = "map"

  default = {
    default = 20
  }
}

variable "node_version" {
  default = "1.9.7-gke.6"
}

variable "dependency_id" {
  description = "This variable is unused. It is here simple as a work around to enforce module dependancy. Github issue 10462, in the provider repo"
  default     = ""
}
```
Config: k8s-module/main.tf  
```
data "google_compute_zones" "available" {}

data "google_container_engine_versions" "available" {
  zone = "${local.primary_zone}"
}

locals {
  default_name = "${var.project_id_map[terraform.workspace]}-cluster"
  name         = "${local.default_name}"
  primary_zone = "${data.google_compute_zones.available.names[0]}"
  project      = "${var.project_id_map[terraform.workspace]}"

  pre_calc_additional_zones = [
    "${data.google_compute_zones.available.names[1]}",
    "${data.google_compute_zones.available.names[2]}",
  ]

  additional_zones = "${compact(local.pre_calc_additional_zones)}"
}

resource "null_resource" "start" {
  triggers {
    depends_id = "${var.dependency_id}"
  }
}

resource "google_container_cluster" "primary" {
  name               = "${local.name}"
  project            = "${local.project}"
  zone               = "${local.primary_zone}"
  node_version       = "${var.node_version}"
  min_master_version = "${var.node_version}"
  description        = "Kubernetes cluster"

  additional_zones = ["${local.additional_zones}"]

  lifecycle {
    ignore_changes = ["node_pool"]
  }

  # Empty node pool
  node_pool {
    name = "unused-default-pool"
  }

  depends_on = ["null_resource.start"]
}

resource "google_container_node_pool" "default" {
  name               = "${local.name}-pool"
  zone               = "${local.primary_zone}"
  cluster            = "${google_container_cluster.primary.name}"
  initial_node_count = 1

  management = {
    auto_repair  = true
    auto_upgrade = false
  }

  autoscaling = {
    min_node_count = 1
    max_node_count = 10
  }

  node_config {
    # Optimal cost performance I/O
    disk_size_gb = "${var.disk_size[terraform.workspace]}"
    machine_type = "${var.instance_size[terraform.workspace]}"
    oauth_scopes = "${var.oauth_scopes}"
  }

  depends_on = ["google_container_cluster.primary"]
}

# The following outputs allow authentication and connectivity to the GKE Cluster.
output "connect-string" {
  value = "${join(
    " ",
    list(
      "gcloud container clusters get-credentials",
      local.name,
      "--zone",
      local.primary_zone,
      "--project",
      local.project
    )
  )}"
}

output "primary_zone" {
  value = "${google_container_cluster.primary.*.zone}"
}

output "kubernetes-version" {
  value = "${data.google_container_engine_versions.available.latest_node_version}"
}

```
- You will get connection string output which you will run to get connection string for kubectl.  
Like:  
```
Apply complete! Resources: 0 added, 0 changed, 0 destroyed.

Outputs:

connection-command = gcloud container clusters get-credentials lony-terraform-demo-cluster --zone us-east1-b --project lony-terraform-demo
```
Run:  
```
sudo gcloud components install kubectl
# OR
sudo apt-get install kubectl
# VALIDATE
kubectl version
kubectl get nodes
```

###### KUBERNETES PODS, SERVICES, DEPLOYMENTS
- PODS : Containers runs inside pods and uses shared storage and networking services which are attached to pods. Pod is the smallest automic unit of kubernetes. Pods have ephemeral IP addresses and they are mortal.  
- SERVICES:  Offers stable IP address and links to pods by label selectors.  
  - Type:  
    - ClusterIP: Internal Cluster IP  
    - NodePort: Mapping node external IP  
    - LoadBalancer: External load balancing for exposing  
    - ExternalName: CNAME mapping  
- DEPLOYMENTS: Wraps pods,replicasets and replication policies.  

- Create a replication controller (like deployment) with nginx and a service with load balancer exposing outside.  
- By kubernetes plugin deployment is not supported use module.  

Config: kubernetes/deployment-service.tf  
```
provider "kubernetes" {
config_path = "~/.kube/config"
#  # leave blank to pickup config from kubectl config of local system
}


resource "kubernetes_replication_controller" "nginx" {
  metadata {
    name = "terraform-example"
    labels {
      app = "nginx"
    }
  }

  spec {
    selector {
      app = "nginx"
    }
    template {
      container {
        image = "nginx"
        name  = "app"

        resources{
          limits{
            cpu    = "0.5"
            memory = "512Mi"
          }
          requests{
            cpu    = "250m"
            memory = "50Mi"
          }
        }
      }
    }
  }
}

resource "kubernetes_service" "example" {
  metadata {
    name = "terraform-nginx-example"
  }

  spec {
    selector {
      app = "nginx"
    }

    session_affinity = "ClientIP"

    port {
      port = 80
    }

    type = "LoadBalancer"
  }
}

output "lb_ip" {
  value = "${kubernetes_service.example.load_balancer_ingress.0.ip}"
}
```  

###### HELM CHART ON KUBERNETES  
- Helm is like apckage management tool for kubernetes.  
- It consists of 3 components.  
  - `helm` command tool.  
  - `tiller` service runs inside kubernetes services.  
  - `Charts` are yaml format `helm` bundled packages from a repository.  
    - There are 2 types of charts.  
      - Official and Unoffial.  
  - You can find, get from repository, customize, manage, manifest, store and install charts by helm.  

Installing `helm`:  
```
curl https://raw.githubusercontent.com/helm/helm/master/scripts/get > get_helm.sh
chmod 700 get_helm.sh
sudo ./get_helm.sh
helm init
```

###### CREATE A MONGODB CLUSTER   

Config: kubernetes/helm.tf  
```
provider "helm" {}

resource "helm_release" "mongo" {
  name  = "mongo"
  chart = "stable/mongodb-replicaset"

  set {
    name  = "auth.adminUser"
    value = "admin"
  }

  set {
    name  = "auth.adminPassword"
    value = "password"
  }
}
```

- Destroy everything you did.  
```
terraform destroy
```



QUESTIONS  
- Check how count.index works.  
- Count, availability zone olayi. anla  
- what happens, if you delete tfstate file.  
- how to secure passwords and keys.  
- How to download/install edit modules.    
- What is consul to keep the state.  
- turn back to 27 learn taint  
- turn back to 28 learn workspace  
- REAL LIFE EXAMPLE DO AGAIN  
