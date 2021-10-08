
# Workshop guide: **Step 2** Use Case 1: ***Terraform***



## Pre-Reqs

  Step 1 outcome and will include:
- Conjur Instance up and running
- Client successfully up and running
- Install Terraform: [More details here](https://learn.hashicorp.com/tutorials/terraform/install-cli?in=terraform/aws-get-started)
 - Install the following:
 ```Bash
sudo yum install -y yum-utils epel-release
sudo yum-config-manager --add-repo https://rpm.releases.hashicorp.com/RHEL/hashicorp.repo
sudo yum -y install terraform jq
```

## Terraform Conjur Provider

For this install we will try two approaches, the first one will use Environment Variables, and the second the Provider attributes.

**Creat 2 directories where we will put the Terraform files**
```Bash
mkdir -p $HOME/terraform-conjur/{envvars,attributes}
 ```

**Get the certificate from Conjur server**
 ```Bash
openssl s_client -showcerts -connect localhost:8443  < /dev/null | sed -ne '/-BEGIN CERTIFICATE-/,/-END CERTIFICATE-/p' > /etc/conjur.pem
 ```

### Using Environment Variables:

 ```Bash
cd $HOME/terraform-conjur/envvars
 ```

**1. Export the environment Variables**

 ```Bash
export CONJUR_APPLIANCE_URL="https://localhost:8443"  
export CONJUR_ACCOUNT="myConjurAccount"  
export CONJUR_AUTHN_LOGIN="host/BotApp/myDemoApp"  
export CONJUR_AUTHN_API_KEY="<BotApp API Key>"  
export CONJUR_CERT_FILE="/etc/conjur.pem"  
 ```

**2. Create your main.tf plan**
 ```Bash
cat << 'EOF' > main.tf  

terraform {  
  required_providers {  
    conjur = {  
      source  = "cyberark/conjur"  
    }  
  }  
}  

provider "conjur" {}  

data "conjur_secret" "botapp_secretvar" {  
  name = "BotApp/secretVar"  
}  

output "dbpass_output" {  
  value = "${data.conjur_secret.botapp_secretvar.value}"  

  # Must mark this output value as sensitive for Terraform v0.15+,  
  # because it's derived from a Conjur variable value that is declared  
  # as sensitive.  
  sensitive = true  
}  

>EOF

 ```

**3. Install the provider**
 ```Bash
terraform init
 ```
**4. Run the Plan**
 ```Bash
terraform plan
terraform plan -out=tfplan
terraform show -json tfplan
 ```

### Using Terraform Attributes:


**1. Create another main.tf plan**
 ```Bash
cd $HOME/terraform-conjur/attributes

cat << 'EOF' > main.tf  

variable "conjur_api_key" {}  
variable "conjur_ssl_cert_path" {}  

terraform {  
  required_providers {  
    conjur = {  
      source  = "cyberark/conjur"  
    }  
  }  
}  

provider "conjur" {  
  appliance_url = "https://localhost:8443"  
  ssl_cert_path = var.conjur_ssl_cert_path  
  account = "myConjurAccount"  
  login = "host/BotApp/myDemoApp"  
  api_key = var.conjur_api_key  
}  

data "conjur_secret" "botapp_secretvar" {  
  name = "BotApp/secretVar"  
}  

output "dbpass_output" {  
  value = "${data.conjur_secret.botapp_secretvar.value}"  

  # Must mark this output value as sensitive for Terraform v0.15+,  
  # because it's derived from a Conjur variable value that is declared  
  # as sensitive.  
  sensitive = true  
}
>EOF  
 ```

**3. Install the provider**
 ```Bash
terraform init
 ```
**4. Run the Plan**
 ```Bash
terraform plan
terraform plan -out=tfplan
terraform show -json tfplan
 ```

