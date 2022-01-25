---
title: Predictive AIOps Workshop Installation Guide
subtitle: Predictive AIOps Workshop
author: 
- Anthony Angelo, Netser Heruty, Daniel Smith, Joe Steinfeld
date: 10 Nov 2021
---

# Introduction

This document provides prescriptive guidance for deploying the Predictive AIOps Workshop infrastructure. The overall Predictive AIOps Workshop Architecture consists of several AWS EC2 servers conforming to a typical N-Tier web application, complete with a Load Generator and Chaos Simulator. The core application is the Spring Pet Clinic which is based on Java Spring Boot and requires the use of a Database for persistence. 

![Workshop Architecture](workshop-architecture.png)

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
1. Request a new instance as follows, using the latest available application version:

   ![New Internal Instance Request](new-internal-instance-request.png)

## Upgrade your NOW Instance to latest Rome version

1. Navigate to [NOW HI](https://support.servicenow.com/now)
1. Select your Instance from the Instances Dashboard
1. Upgrade your instance to latest Rome version & patch level as follows:

   ![Upgrade to Rome](upgrade-to-rome.png)

## Install the HLA stack for your NOW Instance

1. Follow the [HLA Installation Guide](hla-installation-guide.pdf) for the latest installation steps

   > NOTE: Please read and follow all the steps carefully as instructed in the "HLA Installation Guide" document as it us updated frequently by the HLA Development Team

## Check HLA Services Status for your NOW Instance

1. Login to your NOW Instance as an Administrator
1. Set your `Profile Time Zone` accordingly (e.g. `US\Eastern`)

   > NOTE: Log out and back in to make sure your `Profile Time Zone` is set correctly. Failure to do so will adversly affect the workshop and using basic HLA functions like searching and finding log entries.

1. In your browser add the following to your instance URL `xmlstats.do?include=services_status`
1. Check **Services Status** are as follows:


   | Name          | Status |
   | --------------- | -------- |
   | MetricBase    | green  |
   | Occultus      | green  |
   | ElasticSearch | green  |

## Check HLA Package Versions your NOW Instance

1. Login to your NOW Instance as an Administrator
1. In the Filter Navigator enter `sn_occ_stats.do`
1. Note the **Health Log Analytics Package Dependencies & Versions** as follows:


   | Dependency                  | Version   |
   | ----------------------------- | ----------- |
   | Health Log Analytics        | 21.0.1    |
   | Health Log Analytics Viewer | 20.1.4    |
   | Alert Intelligence          | 19.3.8    |
   | Occultus Version            | 1.13.2.25 |
   | Metrics Base Version        | 1.14.0.13 |
   | ElasticSearch Version       | 7.3.2     |

## Install the required ITOM plugins for the Workshop

1. HOP to your NOW Instance as Administrator
1. Navigate to the **System Definition > Plugins** and install or activate the following plugins:


   | Plugin Name |  Plugin ID   |
   | ----------- | ------------ |
   | Agent Client Collector Monitoring | sn_itmon|
   | Service Mapping | com.snc.service-mapping |
   | Discovery and Service Mapping Patterns | sn_itom_pattern |
   | CMDB CI Class Models | sn_cmdb_ci_class |
   | Certificate Inventory and Management | sn_disco_certmgmt |
   | Performance Analytics - Premium | com.snc.pa.premium |
   | ServiceNow IntegrationHub Professional  Pack Installer | com.glide.hub.integrations.professional | 

## Execute the [PA HLA] Historic Data Collection Job to catch up the HLA Overview Dashboard

1. Login to your NOW Instance as an Administrator
1. Set the Application Scope to `Health Log Analytics`
1. Navigate to the **Performance Analytics > Data Collector > Jobs** and Execute the `[PA HLA] Historic Data Collection` by using right-click-mouse to `Execute Now`

## Fix Service Mapping Glitches

1. Login to your NOW Instance as an Administrator
1. In the Filter Navigator enter `sys_properties_list.do`
1. Set the following property:


   | Property Name                 | Value | Type    | Application |
   | ------------------------------- | ------- | --------- | ------------- |
   | com.glide.request.max_waiters | 30    | integer | Global      |

## Optimize your NOW Instance for the Workshop

1. Login to your NOW Instance as an Administrator
1. Navigate to **Health Log Analytics > Health Log Analytics Administration > System Properties** and set the following properties:

   | Property Name                                            | Value | Default Value |
   | ---------------------------------------------------------- | ------- | --------------- |
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
   $ cd heracles/
   ```

   > NOTE: If you don't have an SSH Key setup, use the HTTPS URI instead to clone (e.g. https://github.com/pangealab/heracles.git)

1. Configure AWS Profile

   ```F
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
1. Set your backend bucket property in the *backend.tf* file as follows:

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

# Deploy the MID Server and pre-configure ACC, Filebeat Access

## Configure NOW MID Access

1. Login to your NOW Instance as Administrator
1. Navigate to **Guided Setup > ITOM Guided Setup**
1. Click on **MID Server**
1. Click on **Create MID User**

   ![Create MID User](create-mid-user.png)

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
1. Navigate to **MID Server > Server**
1. Click on your MID Server
1. Click on the **Validate** link
1. Set the **MID Initial Selection Criteria** as follows:

   ![Validate MID Server](validate-mid.png)
1. Click on the **Supported Applications** tab and check settings are as follows:


   | Field                       | Value           |
   | ----------------------------- | ----------------- |
   | Name                        | ALL             |
   | Default MID Server          | YOUR MID SERVER |
   | Included in application ALL | true            |
1. Click on the **Capabilities** tab and check settings are as follows:


   | Field                       | Value           |
   | ----------------------------- | ----------------- |
   | Name                        | ALL             |
   | Default MID Server          | YOUR MID SERVER |
   | Included in application ALL | true            |
1. Click on the **IP Ranges** tab and check settings are as follows:


   | Field | Value   |
   | ------- | --------- |
   | Name  | ALL     |
   | Type  | Include |
   | Range | 0.0.0.0 |

## Setup Metric Intelligence

1. Click on the **Setup Metric Intelligence** link

## Setup Agent Client Collector Listener

1. Click on the **Setup Agent Client Collector Listener** link
1. Set the MID Web Server Port to `8085`
1. Safeguard your Endpoint address (e.g., wss://15.0.1.107:8085/ws/events)

## Collect the MID Web Server API Key

1. Navigate to **MID Server > MID Web Server API Key**
1. Copy and safeguard your `MID Web Server API Key`

## Configure Agent Client Collector Policies

1. Navigate to **Agent Client Collector > Configuration > Policies**
1. Activate the following Policies:

   | Name                | Active |
   | --------------------- | -------- |
   | Linux OS Events     | true   |
   | Linux OS Metrics    | true   |
   | MySQL DB Events     | true   |
   | MySQL DB Metrics    | true   |
   | Self-Healing Events | true   |

## Create your Application Service

1. Navigate to **Service Mapping > Services > Application Services**
1. Create a **New** Application Service `Pet Clinic`
1. Add a **Web Application** Entry Point as follows:


   | Field                           | Value                                       |
   | --------------------------------- | --------------------------------------------- |
   | Discoverable by Service Mapping | true                                        |
   | URL                             | http:// NGINX AWS PRIVATE IP DNS NAME :8080 |
   | Host Name                       | NGINX AWS PRIVATE IP DNS NAME               |
1. Press the **View Map** Button
1. Set the **Operational Status** to `Operational`
1. Press the **Run Discovery** Button

## Create your Application Service  Database Relatonships

1. Navigate to **Service Mapping > Services > Application Services**
1. Select the `Pet Clinic` Application
1. Press the **View Map** Button
1. Hover over each `Tomcat` Server and click on the **Manually add a connection** link
1. Manually add a connection as follows:


   | Field                   | Operator                                   |
   | ------------------------- | -------------------------------------------- |
   | Select Entry Point Type | MySQL Server Endpoint                      |
   | Host                    | YOUR MYSQL PRIVATE IP (e.g. ip-15-0-1-241) |
   | Port                    | YOUR MYSQL PORT (e.g. 3306)                |

1. Press the **Add** Button

# Create your HLA Data Inputs

## NGINX Data Input

1. Login to your NOW Instance as Administrator
1. Navigate to **Health Log Analytics > Data Input**
1. Create a **Linux using Filebeat Data Input** as follows:

   ![NGINX Data Input](create-nginx-di.png)
1. Press **Submit** when done

   > NOTE: Do not download the ”filebeat.yml” as it is part of an Ansible Playbook already

## Spring Data Input

1. Login to your NOW Instance as Administrator
1. Navigate to **Health Log Analytics > Data Input**
1. Create a **Linux using Filebeat Data Input** as follows:

   ![Spring Data Input](create-spring-di.png)
1. Press **Submit** when done

   > NOTE: Do not download the ”filebeat.yml” as it is part of an Ansible Playbook already

## MySQL Data Input

1. Login to your NOW Instance as Administrator
1. Navigate to **Health Log Analytics > Data Input**
1. Create a **Linux using Filebeat Data Input** as follows:
1. Press the **Add** Button

   ![MySQL Data Input](create-mysql-di.png)
1. Press **Submit** when done

   > NOTE: Do not download the ”filebeat.yml” as it is part of an Ansible Playbook already

# Install the ACC & Filebeat Software using Ansible

1. Run the Install Agents Playbook

   ```
   $ ansible-playbook -i YOUR INVENTORY FILE ansible/install-agents.yml \
   -e "acc_mid=YOUR ACC MID URL" \
   -e "acc_api_key=YOUR ACC API KEY" \
   -e "nginx_logstash=YOUR MID SERVER PRIVATE IP:5040" \
   -e "spring_logstash=YOUR MID SERVER PRIVATE IP:5041" \
   -e "mysql_logstash=YOUR MID SERVER PRIVATE IP:5042"
   ```

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

   | Name    | Category |
   | ------- | ------------- |
   | Generate Application Errors | Application Chaos |
   | Generate Infrastructure Errors | Infrastructure Chaos |
   | Stress Servers | nfrastructure Chaos |

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
   
# Configure your NOW HLA instance for a new Workshop

## Configure your Source Type Structures

1. Login to your NOW Instance as Administrator
1. Navigate to **Health Log Analytics > Mapping > Source Type Structures**
1. For each Source Type Structure, set the **Custom JS** Function using the scripts located in the cloned Git Project /servicenow folder (e.g., `source-type-structures-mariadb-error.js`)

   >NOTE: Source Type Structures cannot be updated if stil in `Learning` mode. Make sure you have at least 100 log entries before proceeding. In addition, Custom JS functions must be published to start working by clicking `Publish` after saving the form. 

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

# Grant User Access to your NOW Instance

1. Login to your NOW Instance as Administrator
1. Navigate to **User Admininstration > Users**
1. Create a user named `hlauser1` and grant the `hla_workshop_user` Role
1. Create a user named `hlaadmin1` and grant the `hla_admin_read_only` Role

# Secure your Workshop AWS Environment

## Identify your Crowdstrike CID

1. Navigate to [NOW SURF](https://surf.service-now.com/)
1. Search for `KB0051390` Knowledge Article
1. Note the Crowdstrike CID for the `Cloud (Commercial)` Environment

## Install the Crowdstrike Falcon Agent using Ansible

1. Download the Crowdstrike Falcon Agent RPM from HI for RHEL 8 (e.g. 6.32.0-12904)

   ```
   wget https://surf.service-now.com/sys_attachment.do?sys_id=6e589ffcdb800d10ae878fd3399619f8 -qO falcon-sensor.rpm
   ```

1. Run the Install Falcon Playbook

   ```
   $ ansible-playbook -i YOUR INVENTORY FILE.cfg ansible/install-falcon.yml \
   -e "falcon_rpm=YOUR FALCON RPM FILE" \
   -e "falcon_cid=YOUR FALCON CID" 
   ```

# Appendix A – Terraform Installation

These instructions apply to a WSL Ubuntu workstation. Please refer to the Terraform Downloads page https://www.terraform.io/downloads.html for more information

1. Start a Bash Shell
1. Install Terraform CLI (e.g., v0.12.31)

   ```
   $ wget -qO- https://releases.hashicorp.com/terraform/0.12.31/terraform_0.12.31_linux_amd64.zip | busybox unzip -
   $ chmod 775 terraform
   $ sudo mv terraform /usr/local/bin/
   ```

# Appendix B – Python and Pyenv Installaton

These instructions apply to a WSL Ubuntu workstation and provide an opinionated approach to installing Python. There are many ways to install Python on a workstation and we found that using the `pyenv` utility provides the most consistent approach across several operating systems. 

1. Start a Bash Shell
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

# Appendix C – Ansible Installation

These instructions apply to a WSL Ubuntu workstation and Python PIP which provides broad cross-platform support regardless of the operating system. Please refer to the Ansible Installation Guide https://docs.ansible.com/ansible/latest/installation_guide for more information.

1. Start a Bash Shell
1. Install Ansible (e.g., v4.5.0)
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

# Appendix D – AWS CLI Installation

These instructions apply to a WSL Ubuntu workstation. Please refer to the AWS CLI Installation Guide https://docs.aws.amazon.com/cli/latest/userguide/install-linux.html for more information.

1. Start a Bash Shell
1. Install Venv

   ```
   $ sudo apt-get install -y python3-venv
   ```
1. Install the AWS CLI

   ```
   $ curl "https://s3.amazonaws.com/aws-cli/awscli-bundle.zip" -o "awscli-bundle.zip"
   $ unzip awscli-bundle.zip
   $ sudo /usr/bin/python3 awscli-bundle/install -i \
   /usr/local/aws -b /usr/local/bin/aws
   ```
