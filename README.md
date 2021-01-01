
Terraform :Terraform init ->initialize the direactory with terraform
Terraform refresh : to get the current architechture 
Terraform show : a comprehensive overview of the resources used
Terraform state list :-> gives detailed state of the each resouce used in the infrastructure
Terraform validate :=> before applying the config just to validate the syntaxes

 1.INPUT VARIABLES:

This has two phases to deal with 
One defining the variable and passing the values in the input variables
The main.tf file can define the list of variables we have to load for the resources 
These phase of terraform just acts like the traditional programmin languages

Input varibales can have
	1. data type
	2. data structure
	3. Conditions
    4. variable types
### Data types
String , number, bool, list , map(dict) and tuple ,  object

Lets see some example of using all the data types in one go
'''
Main.tf 
Variable "image_id"{
Type = string
}
Variable "zone"{
Type =list(string)
default =["us-east-1","us-east-2"]
}
Variable "docker_ports"{
Type =  list(object(
{
Internal = number
External = number
Protocol =string 
}
))
Default = [
{
Internal =8020
External = 8350
Protocol ="tcp"
}
]
}
}
'''

Types of the values:
Default :  KEYWORD WHICG HOLDS DEEFAULT VALUE WHEN ARGUMENTTS ARE NOT PASSED


 Type :  defines the data type of the variable

Description :  we can add description of the variable  using this keyword

Validation :   validation gives us space to perform conditions for checking or conditions to be applied on the variables

Error_message = this generates error description when the condition is failed
In case  we can also use can() function to minimize errors

Sensitive :  sensitive = true or false restricts the apply or validation functions to not to display the resuources under sensitive
A small example on the above cases

In main.tf add the variables section 
'''
variable "image_id" {
  type        = string
  description = "The id of the machine image (AMI) to use for the server."
validation {
    # regex(...) fails if it cannot find a match
    condition     = can(regex("^ami-", var.image_id))
    error_message = "The image_id value must be a valid AMI id, starting with \"ami-\"."
  }
}
With sensitivity

variable "user_information" {
  type = object({
    name    = string
    address = string
  })
  sensitive = true
}
resource "some_resource" "a" {
  name    = var.user_information.name
  address = var.user_information.address
}'''
Output :
See when tf apply is done we can not see the param values 
Terraform will perform the following actions:
 some_resource.a will be created
  + resource "some_resource" "a" {
      + name    = (sensitive)
      + address = (sensitive)
    }
PHASE 2: UTILIZING THE INPUT VALUES :
We can refer the variables using var and . Opertator to the resourse definitions
Example ami = var.image_id
---------------------------------------------------------------------------------------------------
METHODS OF ASSIGING THE VALUES TO THE VARIABLES :

To be direct terraform can load values 
	1. Cloud ui
	2. .tfvars
	3. .tfvars.json
	4. Any auto.tfvars or auto.tfvars.json
	5. Var and - var-file 
6 . By exporting the env as TF_VAR_variablename=value
 let us see the basic example of all the methods

By env variable 
$ export TF_VAR_image_id=ami-abc123
This kind doesn’t fit for automation as cli intrerupts for many times
 By cli var command :
Tf apply - var =  " ami_id = ami-oo7 "
Terraform apply -var = ' image_id_map ={"us-east-2":"ami-oo1"}'
	- Var option can be used n no of times
We can save the values in the tf file and can refere them using - var-file command 

Example 
'''
image_id = "ami-abc123"
availability_zone_names = [
  "us-east-1a",
  "us-west-1c",
]
''''
Save them as test.tfvars
Command : terraform apply -var-file = test.tfvars
order of terraform loading the input variables
	 1.Environment variables
	 
	 2.The terraform.tfvars file, if present.
	 
	 3.The terraform.tfvars.json file, if present.
	 4. Any *.auto.tfvars or *.auto.tfvars.json files, processed in lexical order of their filenames.
	 
Any -var and -var-file options on the command line, in the order they are provided. (This includes variables set by a Terraform Cloud workspace.)
===========================================================

####  Local variables:
If we have any repeating values to be substituted in the architechture
We can use local variables
locals {
   Ids for multiple sets of EC2 instances, merged together
  instance_ids = concat(aws_instance.blue.*.id, aws_instance.green.*.id)
}
locals {
   Common tags to be assigned to all resources
  common_tags = {
    Service = local.service_name
    Owner   = local.owner
  }
}
Applying the local variables :
resource "aws_instance" "example" {
  
tags = local.common_tags
}

========================================
#### OUTPUT VARIABLES AND USES OF THEM:

As terraform has thousands of input variables
We might have a usecase of fast fetching some important parameters like load balancers,vpc ids etc in such cases we do use output variables


Terraform outputs the variabless when applied the infra

Lets us assume we have an ip parameter in the main,tf

Declaring the output 

Output "ip"{
Value = aws_resourcename.value

}

Now this gets automaticaally displayed when tf apply is done

To be more specific once the configuration is done we can also do
Terraform output variable name 

            this ends the basics of terraform 
===============================================
Unit  2 :
### Managing infrastructure with terraform
 
#### Providers

Providers are the pluggins for the terraform to interact with cloud apis
For all the resource creations


Learning providers can be divided into three topics
	1. Provider requirements
	2. Provider configuration
	3. Dependency lock
	4. A simple example on how to define lock and version providers in terraform
	
	
	### 1.Provider requirements
	     provider requirements phase tells what is the cloud service you are expecting the infra and versions in that
	
	A simple example:
	terraform {
  required_providers {
    mycloud = {
      source  = "mycorp/mycloud"
      version = "~> 1.0"
    }
  }
}
	provider "mycloud" {
  # ...
}
	
	
	In place of my cloud we can assume aws/google etc
	As tf supports some parts of saas and paas we can also define
	Some of the http servers as the rewuired providers 
	
	So the required providers with source and version block is the main funda of the PROVIDER REQUIREMENTS
	
	
   ### 	2. PROVIDER CONFIGURATION:
                   This phase deals with configuring the options,aliases and multi cloud for the provider

say we have created the provider aws
example : defining the provider requirements
'''
terraform {
     required proviers {
         aws {
           source = "harshicorp/aws"
             version = ">=1.0"
            }
   provider "aws"{
      region="us-west-1"
      alias ="west"
      }
      
 provider "aws"{
  region ="us-east-1"
  alias ="eas1"
  }
  reasource "aws_instance" "my_ec2_machine"{
      provider = aws.west
      
  }
  
  '''
  
  alias for muti provider configuration definitions
  versions are depricated in provider block
  we must ensure the version+expresiion is defined in required providers block
  
  ##### 3.dependency file lock systems
  once we have listed the requirements and configuration ,hit terraform init
  init command specifies the terraform to dowload prescribed providers instead of big bundles
  init automatically creates the lock file for the configuration
  lock files have the information like infra detailes + recorded hashes at the time of the configurtation applied
  to populate or view the checksums and hashes use terraform lock command
  
  the following example gives a detailed view on locking and upgrading the terraform
  https://learn.hashicorp.com/tutorials/terraform/provider-versioning?in=terraform/configuration-language&utm_source=WEBSITE&utm_medium=WEB_IO&utm_offer=ARTICLE_PAGE&utm_content=DOCS
  
 the following
  create/open the versions and add the 
  required_version="value"
  do terraform init
  to apply and generate the new lock file .terraform.lock.
  to see the file it will like as follows
  '''
  provider "registry.terraform.io/hashicorp/aws" {
  version     = "2.50.0"
  constraints = ">= 2.0"
  hashes = [
    "h1:aKw4NLrMEAflsl1OXCCz6Ewo4ay9dpgSpkNHujRXXO8=",
    # ... truncated ...
    "zh:fdeaf059f86d0ab59cf68ece2e8cec522b506c47e2cfca7ba6125b1cd06b8680",
  ]
}

provider "registry.terraform.io/hashicorp/random" {
  version     = "3.0.0"
  constraints = "3.0.0"
  hashes = [
    "h1:yhHJpb4IfQQfuio7qjUXuUFTU/s+ensuEpm23A+VWz0=",
    # ... truncated ...
    "zh:fbdd0684e62563d3ac33425b0ac9439d543a3942465f4b26582bcfabcb149515",
  ]
}
  '''
  we can change any of the versions and constarints and re- init the configuration
  The lock file causes Terraform to always install the same provider version, ensuring that runs across your team or remote sessions will be consistent.
  #### terraform providers chapter summary :
  
  terraform required providers :
  to make  sure we are telling the terraform to download the latest and prescribed plugins with versions
  terraform provider configurations:
  using the defined providers and adding the confgurations
  
  terraform dependency lock file:
  to ensure that same versions are downloded and maintained across the team
  terraform lock 
  terraform init 
  terraform apply 
  constraints : to hold a conditional uphand on the versions 
  checks and validates the hash witg checksum mechanism and installs all the provides and versions
  
  ---------------end of the chapter terraform providers--------
  
  
  ## purpose of terraform state
  maps the infrastrucute to the real world
  it also tracks meta data to gather info about dependencies
  inoder to make sure when destroying the infra , it keeps the state file 
  and metadata to perform operations smoothly without destroying the resources used by other infra
  
  #### for performence :
  terraform stores and fetches the current state data for every plan command 
  so that it can sync with real world attributes
 
  for larger infra , it is recommended to you -refresh=false inorder to work around because some delay in syncing can cause effects
 
 use remote state and remote locking to make sure that two users dont audit the same terraform file at same instance
 
 we can use the terraform block for variaous reasons
 1. to define the version of the terraform
 2. to define the required providers
 3. to define the terraform backend
 a simple example
'''
terraform {
  required_providers {
    aws = {
      version = ">= 2.7.0"
      source = "hashicorp/aws"
    }
  }
}
'''
 we can also explicity pass the metada in this block using provider_meta
to have a overview on the above topic please follow the exaple

use case : create an machine image with packer and terraform

#### packer: open source tool for create identical machine image platforms
as a sungle source of confg
machine image : os image or we can say like a single docker unit kind of
1.in aws its ami for ec2
2.vmdk/vmx for windows
benfits of packer:
packer is a kind of user data automation in ec2
template based ami creation 
this helps to reduce boot time when we are horizontally scaling the system

step1 :
create a basic packer image with all the softawares and configs to be done
step 2 :
write the shell script which does the following
 run the packer to build the images
 populate the ami image ids and store them to variable.tfrs
 link the image ids from variable.tfrs
 terraform init && terraform apply
 
### terraform provisioners
the above work shop comes under provisioning concepet
when we need to integrate with other devops tools
or to do operations like user data in aws ec2 we can go for provisioning
local exec : we can give an linux command to exucute in the current script

self : self is used to access other attributes in the current tf file

only runs when  when first creation of the resource 
or when we are destroying 

example 1:
            resource "aws_instance" "my_instance"{
                location = "us-east-1"
            }
               provisioner "local-exec"{
                   when = destroy
                   command = "echo  instance destroyed at $self.location "
               }

  on_failure :continue like continein programming ignores the errors
              fail : to raise an error
lets see the second example 
resouurce "aws_instance " "web"{
    provisioner "local-exec"{
        command = "echo the server operated ip is ${self.private_ip}"
        on_failure=contine/fail
        
    }
    
}
#### types of provisioners
generic [provisoners given by terraform enterprise]
vendor specific for chef,pupper,salt-masterless etc
lets see the examples for generic provisoners
#### file provisioners
to transfer content from sorce to destionation
directoray remote uploads is also possible 
but ssh connection should be created intially
the directory structe is automatic , if not terraform will create based on the path
arguments are 
source : source path
content : the string to be appended to the destination file 

 
 resource "aws_instance" "web" {


   Copies the myapp.conf file to /etc/myapp.conf
  provisioner "file" {
    source      = "conf/myapp.conf"
    destination = "/etc/myapp.conf"
  }

  Copies the string in content into /tmp/file.log
  provisioner "file" {
    content     = "ami used: ${self.ami}"
    destination = "/tmp/file.log"
  }

  Copies the configs.d folder to /etc/configs.d
  provisioner "file" {
    source      = "conf/configs.d"
    destination = "/etc"
  }

 #### provisioner overall example 
 
 resource "aws_instance" "my instance"{
     public_ip="some_ip"
     region ="us-west1"
we can use connection for ssh to the aws cli instance ,directly     
     connection{
         type ="ssh"
         host=self.public_ip
         user="ubuntu"
as ssh has pem files as password, we use file() to read the pem input         
         private_key= file('home/ubuntu/private_pemfile.pem')
}
end of connection
lets output the log to remote file using terraform provisioner
  provisioner "file"{
      content = "swaroop_loggein sir"
      destination = "/home/ununtu/loggenin_details.txt"
      
  }

this script copies the content string to the destination on the first 
execution of the terraform apply
or we can use source ="randomfile.txt"
then source file gets copied to the destination directory

 
  Copies all files and folders in apps/app1 to D:/IIS/webapp1
  provisioner "file" {
    source      = "apps/app1/"
    destination = "D:/IIS/webapp1"
  }
}

#### remote- exec
this provisioner helpts to run the scripts on the remote server 
works as an add on for the scripts and files given in the file provisioner 

contents :
inline = list of sequential commands
script : takes the script 
scripts : takes the list of the scripts 

#### example on the provisioner "remote-exec"

resource "aws_instance" "my_instance"{
    provisioner "file"{
this source is the file with script
        source = "script.sh"
        destination ="/tmp/script.sh"
    }
    provisioner "remote-exec"{
        inline =[ "chmod +x /tmp/script.sh", "/tmp/scrip.sh args"
        ]
    }
}

summary :
provisioner :
the main use of provisioner is to execute operations in the current file 
or in the remote backend
provisioner "file" for file management in source and destination
executes on the first time of the infra apply
later for destroy 
when = destroy
provisioner remote exe:
for executing commands on the given sorce and destination 
inline,script and scripts
=========end of chapter infrastructure provisioning and configuration management=========================
## UNIT-3  MASTERING THE WORKFLOW



      
      
      
      
  
  
   
         
 
