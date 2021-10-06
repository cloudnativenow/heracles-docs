---
title: HLA Workshop
subtitle: Script
author: 
- Anthony Angelo, Netser Heruty, Daniel Smith
date: 6 Oct 2021
---
# Introduction

This document provides prescriptive guidance for running the HLA Workshop

# HLA Overview

## Objectives (5 minutes)
## Describe Product Overview (5 minutes)
   1. First call deck (technical)
## Describe Workshop Architecture (15 minutes)
   1. Show Application Architecture
      1. Show Application architecture diagram
      1. Show Pet Clinic (webapp)
   1. Show Overall Workshop Architecture
      1. Monitored Application Architecture & Logging (ACC, Filebeats)
      1. HLA Architecture (Talk through Performance & Objection Handling)
# Demonstration (20 minutes - OPTIONAL)
   1. Show Operator Workspace
   1. Show Service Map
   1. Show Alert Intelligence (List View)
   1. Show Event Management correlation (Group Alerts)
   1. Show Log Analytics Alerts
      1. Show Anomaly Example(s)
         1. Show an Identified Issue (Metric)
         1. Show an Anomaly Graph
      1. Show Meaningful Properties (RCA)
      1. Show Surrounding Logs
      1. Show Impacted Services/CIs
      1. Show Machine Learning Feedback
   1. Show Log Analytics Correlations
   1. Show Alert Actions
      1. Show the Agent Assist for an Alert
      1. Show a Remediation Workflow for an Alert

# HLA Deep Dive (60 minutes)

1. Hands-on Workshop  Section(20 minutes)
   1. Demonstrate 3 Chaos Engineering Use-Cases
      1. Trigger Application Chaos
         * This Use-Case uses the [Chaos Monkey REST API](https://codecentric.github.io/chaos-monkey-spring-boot/latest/#_http_endpoint) to setup and trigger Assaults
      1. Trigger Infrastructure Chaos
         * This Use-Case uses the [UNIX logger API](https://man7.org/linux/man-pages/man1/logger.1.html) to inject simulated errors in syslog log files
      1. Trigger Database Chaos
         * This Use-Case uses setting the **LimitNOFILE=64** MySQL property and bouncing the MySQL service to trigger a "Too many open files" errors in logs and makes database inaccessible from Spring Apps
   1. Demonstrate Root Cause Analysis
      1. Change Request is automatically mapped to the Alert
   1. Demonstrate one Remediation
      1. Trigger Application Remediation by turning off assault
      1. Trigger Database Remediation by setting **LimitNOFILE=infinite** and bouncing the MySQL service

iii.     Trigger Infrastructure Remediation by
restarting Spring Services

6. Product Deep Dive Section –
   Describe how the product works, i.e., buttonology) (45 minutes)

a.      Show
Data Inputs (20 minutes)

i.     data
collection options

b.
Show
Streaming Sources & Validation (1 minute)

c.       Show
Data Input Mapping (5 minutes)

i.     Why?
To facilitate baselining of logs…Break down log stream into respective logical
components

ii.     Map
to relevant source types (parsers)

d.
Show
Data input preprocessor (2 minutes)

i.     Modify data as necessary for data
refinement pre ingestion

e.      Source
type structure (15 minutes)

i.     Parsing
log structure to extract and classify key value pairs etc.

f.
Log
sources (2 minutes)

i.     Review mapping results

g.
Log
viewer

i.     Defined alerts

7. Accessing HLA workshop
   Environment (5 minutes)
8. Next steps (5 minutes)
   1. Q&A
   2. Set up touchpoint check in …
