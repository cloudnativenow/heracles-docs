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

![Figure 1](workshop-chaos-architecture.png)

# Application Chaos

# Infrastructure Chaos

1. SSH To Server

   ```
   ssh ec2-user@$HOST PUBLIC IP -i ~/.ssh/YOUR SSH KEY
   ```

1. Simulate 1 Error to /var/log/messages

   ```
   logger -i -t "chaos" -p user.notice "YOUR RANDOM ERROR MESSAGE HERE" -s
   ```

* Simulate 10 Errors to /var/log/messages

   ```
   for (( i=1; i<=10; i++ ))
   do  
      sleep .$[($RANDOM % 10)+1]s
      logger -i -t "chaos" -p user.notice "YOUR RANDOM ERROR MESSAGE HERE"" -s
   done
   ```

# Database Chaos