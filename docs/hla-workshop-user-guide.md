---
title: HLA Workshop
subtitle: User Guide
author: 
- Anthony Angelo, Netser Heruty, Daniel Smith
date: 6 Oct 2021
---
# Introduction

This document provides prescriptive guidance for generating
Chaos in the HLA Workshop environment to simulate anomalies in the (a) application (b) database and (c) infrastructure tiers of the architecture.

# HLA Overview

## Objectives (5 minutes)

## Describe Product Overview (5 minutes)

1. First call deck (technical)

## Describe Workshop Architecture (15 minutes)

1. Show Application Architecture
   1. Show Application architecture diagram
   2. Show Pet Clinic (webapp)
2. Show Overall Workshop Architecture
   1. Monitored Application Architecture & Logging (ACC, Filebeats)
   2. HLA Architecture (Talk through Performance & Objection Handling)

# Demonstration (20 minutes - OPTIONAL)

1. Show Operator Workspace
2. Show Service Map
3. Show Alert Intelligence (List View)
4. Show Event Management correlation (Group Alerts)
5. Show Log Analytics Alerts
   1. Show Anomaly Example(s)
      1. Show an Identified Issue (Metric)
      2. Show an Anomaly Graph
   2. Show Meaningful Properties (RCA)
   3. Show Surrounding Logs
   4. Show Impacted Services/CIs
   5. Show Machine Learning Feedback
6. Show Log Analytics Correlations
7. Show Alert Actions
   1. Show the Agent Assist for an Alert
   2. Show a Remediation Workflow for an Alert

# HLA Deep Dive (60 minutes)

1. Hands-on Workshop  Section(20 minutes)
   1. Demonstrate 3 Chaos Engineering Use-Cases
      1. Trigger Application Chaos
         * This Use-Case uses the [Chaos Monkey REST API](https://codecentric.github.io/chaos-monkey-spring-boot/latest/#_http_endpoint) to setup and trigger Assaults
      2. Trigger Infrastructure Chaos
         * This Use-Case uses the [UNIX logger API](https://man7.org/linux/man-pages/man1/logger.1.html) to inject simulated errors in syslog log files
      3. Trigger Database Chaos
         * This Use-Case uses setting the **LimitNOFILE=64** MySQL property and bouncing the MySQL service to trigger a "Too many open files" errors in logs and makes database inaccessible from the Spring Apps
   2. Demonstrate Root Cause Analysis
      1. Change Request is automatically mapped to the Alert
   3. Demonstrate one Remediation
      1. Trigger Application Remediation by turning off assault
      2. Trigger Infrastructure Remediation by restarting the Spring Apps services
      3. Trigger Database Remediation by setting **LimitNOFILE=infinite** and bouncing the MySQL service
2. Product Deep Dive Section (aka. Butonology) (45 minutes)
   1. Show **Data Inputs** (20 minutes)
      * Show Data Collection options
   2. Show **Streaming Sources** and their validation (1 minute)
   3. Show **Data Input Mapping** (5 minutes)
      * Show why Data Input Mapping is important (i.e., to facilitate baselining of logs and break down log stream into respective logical components)
      * Show go to Map Data Inputs to relevant Source Types and talk about Parsers
   4. Show **Data Input Preprocessor** (2 minutes)
      * Modify data as necessary for data refinement pre ingestion
   5. Show **Source Type Structure** (15 minutes)
      * Show parsing log structures and how thy are used to extract and classify key value pairs, etc.
   6. Show **Log Sources**  (2 minutes)
      * Review Mapping Results
   7. Show **Log Viewer**
      * Show Defined Alerts
3. Disclose how to access the HLA Workshop Environment (5 minutes)
4. Discuss next steps (5 minutes)
   1. Q & A with Customer
   2. Schedule Touchpoint with Customer
