---
title: Predictive AIOps Workshop Introduction
subtitle: Predictive AIOps Workshop
author: Anthony Angelo, Netser Heruty
date: 1 Feb 2022
---
# Introduction

Title | Subtitle | Author | Date
--- | --- | --- | --- 
Predictive AIOps Workshop 1.0 Installation Guide | Predictive AIOps Workshop - I am the I in AI | Anthony Angelo, Netser Heruty, Danny Smith | 10 Nov 2021
**Predictive AIOps Workshop 2.0 Installation Guide** | **Predictive AIOps Workshop - The Rise of the Unified Agent** | **Anthony Angelo, Netser Heruty** | **01 Feb 2022**
_Predictive AIOps Workshop 3.0 #Future Plan#_ | _Predictive AIOps Workshop - Feat. Grafana #No_Time_to_Glide_ | _Anthony Angelo, Netser Heruty, Karlis Peterson_ | _July 2022?_

# Introduction

The heracles-docs repo provides prescriptive guidance for deploying the Predictive AIOps Workshop environment (both the monitored app, and the ServiceNow instance). The overall Predictive AIOps Workshop Architecture (see diagram below) consists of several AWS EC2 servers conforming to a typical N-Tier web application, complete with a Load Generator and Chaos Simulator. The core application is the Spring Pet Clinic which is based on Java Spring Boot and requires the use of a Database for persistence.

![image](workshop-app-arch.png)

On the ServiceNow side, we provide the full **ITOM Health** experience leveraging the Agent Client Collector (ACC), Health Log Analytics (HLA), Metric Intelligence (MI), Event Managment (EM) as well as ITOM Visiblity with Discovery and Service Mapping (see diagram below). Each of the Configuration Items (CI's) in the environment is fully monitored by ACC which runs basic discovery and collects all the observability data from the apps.

![image](workshop-sn-arch.png)

The latest version of ACC unifies the legacy functions of our Monitoring Agent (ACC-M) together with log collection, providing a unified AIOps capability. ACC is built on the [Sensu.io](https://sensu.io) framework and comes installed with monitoring capabilities for servers, databases, application servers, and middleware. Sensu based checks and policies run in the ACC agent to retrieve the relevant data, which is transformed into events or metrics which are analyzed in real-time by the Metric Intelligence capability. In addition, this latest ACC version removes the need for a separate 3rd party agent to collect log files (e.g., Filebeat) simplifying the deployment. Log files from your application components and infrastructure are streamed into ServiceNow and analyzed in real-time by HLA looking for anomalies and alerting users when they are detected. Alerts are scanned by Event Management and correlated using temporal or lexical heuristics, or by leveraging relationships defined in the Configuration Management Database (CMDB).