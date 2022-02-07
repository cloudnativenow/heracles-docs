---
title: Predictive AIOps Workshop Installation Guide
subtitle: Predictive AIOps Workshop
author: 
- Anthony Angelo, Netser Heruty, Daniel Smith, Joe Steinfeld, Mike Gallagher
date: 1 Feb 2022
---
# Introduction

This document provides prescriptive guidance for deploying the Predictive AIOps Workshop infrastructure and ServiceNow instance configuration. 

# Prerequisites

This document assumes a basic level of competency and familiarity with the tools listed as prerequisites. See the Appendix section for some basic guidance of the use of these tools. Following is a list of prerequisite tools and accesses needed to perform a full Predictive AIOps Workshop installation:

* Access to an AWS Account with full admin privileges
* AWS CLI
* Bash Terminal Access (e.g., WSL for Windows, MacOS Terminal or another Linux)
* Terraform v0.12.31
* Python v3.7.10
* Ansible v4.5.0

# Deploy your NOW Instance

## Request a new NOW Instance

1. Navigate to [NOW HI](https://support.servicenow.com/now)
1. Search for "new internal instance request"
1. Request a new instance as follows, asking for the latest available **Rome** application version (e.g. `Rome Patch 6`):

   ![New Internal Instance Request](new-internal-instance-request.png)

## Upgrade your NOW Instance to latest Rome version

1. Navigate to [NOW HI](https://support.servicenow.com/now)
1. Select your Instance from the Instances Dashboard
1. Upgrade your instance to latest Rome version & patch level
> NOTE: **Minimal** Family Release version for this workshop is **Rome Patch 6 (RP6)**

## Install the HLA stack for your NOW Instance

1. Follow [KB0998946](https://support.servicenow.com/kb?id=kb_article_view&sysparm_article=KB0998946) for the latest installation steps

   > NOTE: Please read and follow all the steps carefully as instructed in the HLA Installation Guide KB, as it is updated frequently by the HLA Dev team

## Check HLA Services Status for your NOW Instance

1. Login to your NOW Instance as an Administrator
1. Set your `Profile Time Zone` accordingly (e.g. `US\Eastern`)

   > NOTE: Log out and back in to make sure your `Profile Time Zone` is set correctly. Failure to do so will adversly affect the workshop and using basic HLA functions like searching and finding log entries.

1. In your browser add the following to your instance URL: `xmlstats.do?include=services_status`
1. Check **Services Status** are as follows:


   | Name          | Status |
   | ------------- | ------ |
   | MetricBase    | green  |
   | Occultus      | green  |
   | ElasticSearch | green  |

## Check HLA Package Versions your NOW Instance

1. Login to your NOW Instance as an Administrator
1. In the Filter Navigator enter `sn_occ_stats.do`
1. Note the **Health Log Analytics Package Dependencies & Versions** as follows:


   | Dependency                  | Version   |
   | --------------------------- | --------- |
   | Health Log Analytics        | 22.0.12   |
   | Health Log Analytics Viewer | 21.0.0    |
   | Alert Intelligence          | 19.4.7    |
   | Occultus Version            | 1.14.0.34 |
   | Metrics Base Version        | 1.14.0.13 |
   | ElasticSearch Version       | 7.3.2     |


   > NOTE: Confirm your Occultus version matches your HLA version using this matrix: [KB1002197](https://support.servicenow.com/kb?id=kb_article_view&sysparm_article=KB1002197). If it doesn't, comment on your HLA installation CHG request asking to fix this.

## Install the required ITOM plugins for the Workshop

1. HOP to your NOW Instance as Administrator
1. Navigate to the **System Definition > Plugins** and install or activate the following mandatory plugins:


   | Plugin Name                                                  | Plugin ID                               |
   | ------------------------------------------------------------ | --------------------------------------- |
   | Agent Client Collector Log Analytics                         | sn_accl                                 |
   | Service Mapping                                              | com.snc.service-mapping                 |
   | Discovery and Service Mapping Patterns                       | sn_itom_pattern                         |
   | CMDB CI Class Models                                         | sn_cmdb_ci_class                        |
   | Certificate Inventory and Management                         | sn_disco_certmgmt                       |
   | Performance Analytics - Premium                              | com.snc.pa.premium                      |
   | ServiceNow IntegrationHub Professional Pack Installer [^1]   | com.glide.hub.integrations.professional |

   [^1]: Requires installation using `maint` role after hopping in with full access

## Execute the [PA HLA] Historic Data Collection Job to catch up the HLA Overview Dashboard

1. Login to your NOW Instance as an Administrator
1. Set the Application Scope to `Health Log Analytics`
1. Navigate to the **Performance Analytics > Data Collector > Jobs** and Execute the `[PA HLA] Historic Data Collection` by using right-click-mouse to `Execute Now`

## Fix Service Mapping Glitches

1. Login to your NOW Instance as an Administrator
1. In the Filter Navigator enter `sys_properties_list.do`
1. Set the following property:


   | Property Name                 | Value | Type    | Application |
   | ----------------------------- | ----- | ------- | ----------- |
   | com.glide.request.max_waiters | 30    | integer | Global      |

## Optimize your NOW Instance for the Workshop

1. Login to your NOW Instance as an Administrator
1. Navigate to **Health Log Analytics > Health Log Analytics Administration > System Properties** and set the following properties:


   | Property Name                                            | Value | Default Value |
   | -------------------------------------------------------- | ----- | ------------- |
   | aggregator.window_size_seconds                           | 300   | 1800          |
   | rules.filter_detections_with_low_current_value.threshold | 1.0   | 5.0           |
   | incidents.cooldown_period_minutes                        | 2     | 5             |


   > NOTE: Request an Occultus restart for properties that require it

1. Navigate to **Health Log Analytics > Health Log Analytics Administration > Features** and set the following features:


   | Name                                              | State | Default Value |
   | ------------------------------------------------- | ----- | ------------- |
   | Disable  Warm Up Time Rule                        | ON    | OFF           |
   | Disable Setup Time Rule                           | ON    | OFF           |
   | Disable All Events Metric Anomaly Detections Rule | ON    | OFF           |

# Deploy the Workshop AWS Environment

## Create the AWS Environment using Terraform

1. Clone Git Project

   ```
   $ git clone git@github.com:pangealab/heracles.git
   ```

   > NOTE: If you don't have an SSH Key setup, use the HTTPS URI instead to clone (e.g. https://github.com/pangealab/heracles.git)

1. Change to heracles folder

   ```
   $ cd heracles/
   ```

   > NOTE: Remain in this folder for the remainder of this installation. All files referenced therein are located in this folder.

1. Configure AWS Profile

   ```
   $ export AWS_PROFILE=YOUR PROFILE; printenv AWS_PROFILE
   ```

1. Standardize on using one AWS Region. The automation scripts provided assume you are deploying in one region (e.g. `us-east-2`). To change to another region, edit the `backend.tf`,`providers.tf` and `variables.tf` files and replace with your specific region.

1. Create Terraform State Bucket using your CLUSTER ID (e.g., hlawork1)

   ```
   $ aws s3 mb s3://YOUR CLUSTER ID-terraform-backend --profile YOUR PROFILE
   ```
1. Create SSH Key (e.g., heracles)

   ```
   $ ssh-keygen -t rsa -b 4096 -C "heracles@noreply.com" -f $HOME/.ssh/heracles -m PEM
   ```
1. Set your backend bucket property in the `backend.tf` file as follows:

   ```
   # Save Terraform State to S3 Bucket
   terraform {
   backend "s3" {
       bucket = "YOURCLUSTERID-terraform-backend"
       key    = "terraform.tfstate"
       region = "us-east-2"
   }
   }
   ```
1. Initialize Terraform

   ```
   $ terraform init
   ```
1. Create Infrastructure

   ```
   $ terraform apply -auto-approve -var instance_count=3 -var cluster_name=YOUR CLUSTER ID
   ```
1. Safeguard the Terraform output of server public and private IPs

1. Safeguard the generated Ansible Inventory file (e.g., inventory-hlawork1.cfg)

## Install the Pet Clinic Software stack using Ansible

1. Set SSH Agent

   ```
   $ eval `ssh-agent -s`
   $ ssh-add ~/.ssh/YOUR SSH KEY 
   ```
1. Run the Install Pet Clinic Playbook

   ```
   $ ansible-playbook -i YOUR INVENTORY FILE.cfg ansible/install-petclinic.yml \
   -e "github_token=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" \
   -e "mysql_host=YOUR MYSQL PRIVATE IP" \
   -e "servers='SPACE SEPARATED LIST OF YOUR SPRING SERVERS PRIVATE IPS'" \
   -e "frontend_addr=YOUR NGINX PUBLIC IP:8080"
   ```

# Deploy the MID Server and pre-configure ACC Access

## Configure NOW MID Access

1. Login to your NOW Instance as Administrator
1. In your browser add the following to your instance URL: `/nav_to.do?uri=%2F$mid_server_user.do`
1. **Create MID Server User**

   ![Create MID User](create-mid-user.png)

   > Safeguard the Credentials for the next step
   >

## Install MID Server Software using Ansible

1. Run the Install MID Server Playbook

   ```
   $ ansible-playbook -i YOUR INVENTORY FILE.cfg ansible/install-midserver.yml \
   -e "instance_url=https://YOUR NOW URL" \
   -e "mid_username=YOUR MID SERVER USER ID" \
   -e "mid_password=YOUR MID SERVER USER PASSWORD"
   ```

## Configure Discovery Credentials

1. Login to your NOW Instance as Administrator
1. Navigate to **Discovery > Credentials**
1. Add a **SSH Private Key Credentials** Credential named `heracles` as follows:

   ![SSH Credentials](heracles-credential.png)
1. Test Credential with any of your server private IP addresses (e.g. mysql, spring, etc.)

## Validate MID Server

1. Login to your NOW Instance as Administrator
1. Navigate to **MID Server > Servers**
1. Click on your MID Server
1. Click on the **Validate** link
1. Set the **MID Initial Selection Criteria** as follows:

   ![Validate MID Server](validate-mid.png)

## Setup ACC Monitoring:

In the MID Server record:
1. Click on the **Setup ACC Monitoring** link
1. Set the MID Web Server Port to `8085`
1. Safeguard your ACC Websocket Endpoint URL address (e.g., `wss://15.0.1.210:8085/ws/events` - to later be used as "acc_mid")

## Collect the MID Server API Key

1. Navigate to **MID Server > Extensions > MID Web Server API Key**
2. Copy and safeguard your `MID Web Server API Key` - to later be used as "acc_api_key"

## Setup ACC Log Analytics:
In the MID Server record:
1. Click on the **Setup ACC Log Analytics** link
1. Set the ACC Data Input Port to `5044`

## Validate ACC-L Extension Contexts

1. Cick on **Extension Context** Tab and check extensions are as follows:

   ![Extension Contexts](extension-contexts.png)

   > NOTE: Make sure form is in **Advanced View** for Tabs to be visible

## Configure Agent Client Collector Policies

1. Navigate to **Agent Client Collector > Configuration > Policies**
1. Activate the following Policies:


   | Name                | Active |
   | ------------------- | ------ |
   | Linux OS Events     | true   |
   | Linux OS Metrics    | true   |
   | MySQL DB Events     | true   |
   | MySQL DB Metrics    | true   |

## Create your Application Service

1. Navigate to **Service Mapping > Services > Application Services**
1. Create a **New** Application Service `Pet Clinic`
1. Add a **Web Application** Entry Point as follows:

   | Field                           | Value                                       |
   | ------------------------------- | ------------------------------------------- |
   | Discoverable by Service Mapping | true                                        |
   | URL                             | http:// NGINX AWS PRIVATE IP DNS NAME :8080 |
   | Host Name                       | NGINX AWS PRIVATE IP DNS NAME               |

1. Under **Additional Info**, set the **Operational Status** to `Operational` and Update
1. Press the **View Map** Button
1. If Discovery isn't already running, press the **Run Discovery** button

## Create your Application Service Database Relatonships

1. Navigate to **Service Mapping > Services > Application Services**
1. Select the `Pet Clinic` Application
1. Press the **View Map** Button to display the Service Map as follows:

   ![Service Map without the Database](petclinic-service-map-without-db.png)

1. On the top right, press the  **Suggestions > Connection Suggestion** Button
1. Select All suggestions in the list and press the **Add** button as follows:

   ![Connection Suggestions for Database](petclinic-connection-suggestions-db.png)

1. Your Service Map should now look as follows:

   ![Service Map with the Database](petclinic-service-map-with-db.png)

# Configure ACC Log Policies

1. Navigate to **System Update Sets > Retrieved Update Sets > Import Update Set from XML**
1. Select the `acc-l-policies-update-set.xml` Update Set from the `heracles/servicenow/` local folder
3. Navigate to **System Update Sets > Retrieved Update Sets > Import Update Set from XML**
1. Select the `Workshop custom ACC log policies` and press `Preview Update Set`
1. Check all records under the "Preview Problems for Batch" and select `Accept remote update` for **all Errors** listed
1. Press `Commit Update Set`

> NOTE: to see the changes, navigate to **ACC Log Analytics > ACC Log Policies**, clear the filter, and confirm the following policies exist:
   ![custom acc log policies](acc-log-policies-workshop.png)

# Install the ACC Software using Ansible

1. Run the Install Agents Playbook

   ```
   $ ansible-playbook -i YOUR INVENTORY FILE ansible/install-agents.yml \
   -e "acc_mid=YOUR MID ACC WEBSOCKET ENDPOINT URL" \
   -e "acc_api_key=YOUR MID WEB SERVER API KEY"
   ```

# Configure HLA Data Parsing

## Configure your Source Type Structures

1. Login to your NOW Instance as Administrator
1. Navigate to **Health Log Analytics > Mapping > Source Type Structures**
1. For each Source Type Structure, set the **Custom JS** Function using the scripts located in the cloned Git Project /servicenow folder (e.g., `source-type-structures-mariadb-error.js`)

   > NOTE: Source Type Structures cannot be updated if stil in `Learning` mode. Make sure you have at least 100 log entries before proceeding. In addition, Custom JS functions must be published to start working by clicking `Publish` after saving the form.

## Configure your Source Type Structures Key/Value Mappings

1. Login to your NOW Instance as Administrator
1. Navigate to **Health Log Analytics > Mapping > Source Type Structures**
1. For each Source Type Structure, set the Key/Value Mappings as follows

Syslog Logs

![syslog-messages](syslog-messages-kvm.png)

Spring App Logs

![spring-app](spring-app-kvm.png)

Spring Access Logs

![spring-access](spring-access-kvm.png)

NGINX Error Logs

![nginx-error](nginx-error-kvm.png)

NGINX Access Logs

![nginx-access](nginx-access-kvm.png)

MariaDB SQL Dump

![mariadb-sql](mariadb-sql-kvm.png)

MariaDB Error Logs

![mariadb-error](mariadb-error-kvm.png)

# Configure your NOW HLA instance Chaos Catalog

## Login to your NOW Instance

1. Login to your NOW Instance as Administrator

## Install the Chaos Catalog Global Update Set

1. Navigate to **System Update Sets > Retrieved Update Sets > Import Update Set from XML**
1. Select the `chaos-catalog-global-update-set.xml` Update Set from the `heracles/servicenow/` local folder
1. Select the `HLA WorkShop Global Updates` Loaded Update Set and press `Preview Update Set`
1. Press `Commit Update Set`

   > NOTE: Select `Accept remote update` for any Errors listed and commit update set
## Install the Chaos Catalog Service Portal Update Set

1. Navigate to **System Update Sets > Retrieved Update Sets > Import Update Set from XML**
1. Select the `chaos-catalog-portal-update-set.xml` Update Set from the `heracles/servicenow/` local folder
1. Select the `Predictive AIOps Workshop Service Portal` Loaded Update Set and press `Preview Update Set`
1. Press `Commit Update Set`

   > NOTE: Select `Accept remote update` for any Errors listed and commit update set
## Create Support Group

1. Navigate to **User Administration > Groups**
1. Create a Group called `Application Support`

## Map Linux Servers to Support Group

1. Navigate to **Configuration > Servers > Linux**
1. Add the `Support group` field to the list
1. Map each of your Spring Application Servers `Support group` to `Application Support`
1. Filter list to `Show Matching` only items mapped to the `Applicaton Support` group
1. Select `Copy query` using *right-click* on the `All>Support group=Application Support` fiter breadcrumb

## Configure the Generate Infrastructure Errors Catalog Item List Collector

1. Navigate to **Service Catalog > Catalog Definitions > My Items**
1. Filter list to `Show Matching` only items mapped to the `Predictive AIOps Workshop Error Generation` Catalogs as follows:

   | Name                           | Category             |
   | ------------------------------ | -------------------- |
   | Generate Application Errors    | Application Chaos    |
   | Generate Infrastructure Errors | Infrastructure Chaos |
   | Stress Servers                 | Infrastructure Chaos  |

1. For each item in this select the `List Collector` variable

   ![list-collector](list-collector.png)
1. Set the `Reference qualifier` field to the query copied earlier

   ![reference-qualifier](reference-qualifier.png)

## Configure the Chaos Catalog Credentials

1. Navigate to **Connections & Credentials > Connections & Credential Aliases**
1. Click on the `hla_workshop_creds` Credential
1. Add a **SSH Private Key Credentials** Credential named `heracles` as follows:
   ![SSH Credentials](heracles-credential.png)
1. Test Credential with any of your server IP addresses (e.g. mysql, spring, etc.)

## Test the Chaos Catalog

1. Navigate to `https://YOUR INSTANCE.service-now.com/chaos`
1. You should see the following Chaos Catalog Page:

   ![chaos-catalog](chaos-catalog.png)

# Grant User Access to your NOW Instance

1. Login to your NOW Instance as Administrator
1. Navigate to **User Admininstration > Users**
1. Create a user named `hlauser1` and grant the `hla_workshop_user` Role
1. Create a user named `hladmin1` and grant the `hla_admin_read_only` Role

# Secure your Workshop AWS Environment

## Identify your Crowdstrike CID

1. Navigate to [NOW SURF](https://surf.service-now.com/)
1. Search for `KB0051390` Knowledge Article
1. Note the Crowdstrike CID for the `Cloud (Commercial)` Environment

## Install the Crowdstrike Falcon Agent using Ansible

1. Download the Crowdstrike Falcon Agent RPM from HI for RHEL 8 (e.g. 6.32.0-12904)

   ```
   wget "https://surf.service-now.com/sys_attachment.do?sys_id=6e589ffcdb800d10ae878fd3399619f8" -qO falcon-sensor.rpm
   ```
1. Run the Install Falcon Playbook

   ```
   $ ansible-playbook -i YOUR INVENTORY FILE.cfg ansible/install-falcon.yml \
   -e "falcon_rpm=YOUR FALCON RPM FILE" \
   -e "falcon_cid=YOUR FALCON CID" 
   ```
# Appendix A – WSL Ubuntu Prerequisites Installation

Instructions for installing the lab buildout pre-requisites on WSL Ubuntu.

## Install Terraform

1. Start a Bash Shell
1. Install Terraform CLI (e.g., v0.12.31). See the [Terraform Docs](https://www.terraform.io/downloads.html) for more information.
   ```
   $ wget -qO- https://releases.hashicorp.com/terraform/0.12.31/terraform_0.12.31_linux_amd64.zip | busybox unzip -
   $ chmod 775 terraform
   $ sudo mv terraform /usr/local/bin/
   ```
1. Install pyenv
   ```
   curl https://pyenv.run | bash
   ```
1. Edit Bashrc
   ```
   export PYENV_ROOT="$HOME/.pyenv"
   export PATH="$PYENV_ROOT/bin:$PATH"
   eval "$(pyenv init --path)"
   ```

## Install Python

1. Start a Bash Shell
1. Install Python
   ```
   pyenv install 3.7.10
   ```
1. Get Versions
   ```
   pyenv versions
   ```
1. Use Version
   ```
   pyenv global 3.7.10
   ```

## Install Ansible

1. Start a Bash Shell
1. Install Ansible (e.g., v4.5.0). See the [Ansible Docs](https://docs.ansible.com/ansible/latest/installation_guide) for more information.

   ```
   $ pip install ansible==4.5.0
   ```
1. Edit Ansible Settings (e.g. vi ~/.ansible.cfg)

   ```
   [defaults]
   interpreter_python=auto_silent
   ideprecation_warnings=false
   ```
1. Install Prerequisites

   ```
   ansible-galaxy collection install community.mysql
   ```

##  Install AWS CLI

1. Start a Bash Shell
1. Install Venv

   ```
   $ sudo apt-get install -y python3-venv
   ```
1. Install the AWS CLI. See the [AWS Docs](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-welcome.html) for more information.

   ```
   $ curl "https://s3.amazonaws.com/aws-cli/awscli-bundle.zip" -o "awscli-bundle.zip"
   $ unzip awscli-bundle.zip
   $ sudo /usr/bin/python3 awscli-bundle/install -i \
   /usr/local/aws -b /usr/local/bin/aws
   ```
1. Configure your AWS CLI Profile. See the [AWS docs](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-configure.html) for more information.

   ```
   aws cli configure --profile=YOUR PROFILE
   ```
   > NOTE: Remember to configure your profile default region and credentials.

1. Validate AWS CLI Access

   ```
   aws ec2 describe-regions --profile=YOUR PROFILE
   ```

# Appendix B - MacOS Prerequisites Installation

Instructions for installing the lab buildout pre-requisites on a Mac.  

## Install Homebrew

1. Open a Terminal
1. Install [Homebrew](https://brew.sh/)

   ```
   /bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
   ```

## Install Terraform

1. Open a Terminal
1. Install Terraform. See the [Terraform Docs](https://www.terraform.io/docs) for more information

   ```
   brew install terraform@0.12
   ```

## Install Python

1. Open a Terminal
1. Install Python

   ```
   brew install python
   ```

## Install Ansible

1. Open a Terminal
1. Install Ansible. See the [Ansible Docs](https://docs.ansible.com/ansible/latest/installation_guide) for more information.

   ```
   brew install ansible
   ```
   
## Install AWS CLI

1. Configure your AWS CLI Profile. See the [AWS Docs](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-welcome.html) for more information. 

1. Open a Terminal
1. Install AWS CLI

   ```
   brew install awscli
   ```
1. Configure your AWS CLI Profile. See the [AWS docs](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-configure.html) for more information. 

   ```
   aws cli configure --profile=YOUR PROFILE
   ```
   > NOTE: Remember to configure your profile default region and credentials.

1. Validate AWS CLI Access

   ```
   aws ec2 describe-regions --profile=YOUR PROFILE
   ```