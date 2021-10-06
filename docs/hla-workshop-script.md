---
title: HLA Workshop
subtitle: Script
author: 
- Anthony Angelo, Netser Heruty, Daniel Smith
date: 6 Oct 2021
---
# Introduction

This document provides prescriptive guidance for running the HLA Workshop

# Overview (25 minutes)

## Objectives (5 minutes)

## Product Overview (10 minutes)

   1. First call deck (technical)

## Workshop Architecture (10 minutes)

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

# Product Deep Dive (60 minutes)

1. Hands-on Workshop  Section(20 minutes)
   1. Demonstrate 3 Chaos Engineering Use-Cases
      1. Trigger Application Chaos
         * This Use-Case uses the [Chaos Monkey REST API](https://codecentric.github.io/chaos-monkey-spring-boot/latest/#_http_endpoint) to setup and trigger Assaults
      1. Trigger Infrastructure Chaos
         * This Use-Case uses the [UNIX logger API](https://man7.org/linux/man-pages/man1/logger.1.html) to inject simulated errors in syslog log files
      1. Trigger Database Chaos
         * This Use-Case uses setting the **LimitNOFILE=64** MySQL property and bouncing the MySQL service to trigger a "Too many open files" errors in logs and makes database inaccessible from the Spring Apps
   1. Demonstrate Root Cause Analysis
      1. Change Request is automatically mapped to the Alert
   1. Demonstrate one Remediation
      1. Trigger Application Remediation by turning off assault
      1. Trigger Infrastructure Remediation by restarting the Spring Apps services
      1. Trigger Database Remediation by setting **LimitNOFILE=infinite** and bouncing the MySQL service
1. Product Deep Dive Section (aka. Butonology) (45 minutes)
   1. Show **Data Inputs** (20 minutes)
      * Show Data Collection options
   1. Show **Streaming Sources** and their validation (1 minute)
   1. Show **Data Input Mapping** (5 minutes)
      * Show why Data Input Mapping is important (i.e., to facilitate baselining of logs and break down log stream into respective logical components)
      * Show go to Map Data Inputs to relevant Source Types and talk about Parsers
   1. Show **Data Input Preprocessor** (2 minutes)
      * Modify data as necessary for data refinement pre ingestion
   1. Show **Source Type Structure** (15 minutes)
      * Show parsing log structures and how thy are used to extract and classify key value pairs, etc.
   1. Show **Log Sources**  (2 minutes)
      * Review Mapping Results
   1. Show **Log Viewer** 
      * Show Defined Alerts

# Workshop environment hand off
1. Disclose how to access the HLA Workshop Environment (5 minutes)
1. Describe Workshop usage
   1. Chaos Engineering
   1. Root Cause Analysis (RCA)
   1. Remediation

# Conclusion (5 mins)
   1. Q & A with Customer
   2. Schedule Touchpoint with Customer
