---
title: HLA Workshop User Guide
subtitle: Health Log Analytics 
author: 
- Anthony Angelo, Netser Heruty, Daniel Smith
date: 6 Oct 2021
---

# Introduction

This document provides prescriptive guidance for generating Chaos in the HLA Workshop environment to simulate anomalies in the Pet Clinic application. 

![Figure 1](workshop-chaos-architecture.png)

# Application Chaos

Following is an example scenario to trigger application chaos for the HLA Workshop which will ultimately be detected as an anomaly in ServiceNow.

## What is Chaos Monkey

Chaos Monkey for Spring Boot is an open-source project for Java Spring based applications used to simulate & control service degradations, random errors & stack traces.  Please refer to the Chaos Monkey Project https://codecentric.github.io/chaos-monkey-spring-boot for more information.

## Workshop Application Chaos Architecture

The Spring Boot application layer has been built and injected with the Chaos Monkey libraries which in turn, trigger service degradations and random errors, otherwise known as chaos. This simulated chaos is eventually detected by the HLA system as anomalies after receiving and processing the application telemetry, such as streaming logs from all layers of the application. The frequency and the nature of the errors are controlled using the assaults REST API and can also be altogether turned off using the disable and enable REST APIs. 

## Chaos Monkey API Explained

The Chaos Monkey assaults API is accessible via REST both via HTTP GET from a browser to simply query for status, as well as HTTP PUT to configure new assaults. Chaos Monkey is initially deployed in a disabled state to minimize impact to the initial HLA Workshop deployment and scenarios.  The Chaos Monkey APIs are as follows:

| HTTP Method | REST API | Description |
| ----------- | -------- | ----------- |
| GET	| /actuator/chaosmonkey/status |	Get status of the assaults |
| POST | /actuator/chaosmonkey/enable | Enable assaults |
| POST | /actuator/chaosmonkey/disable | Disable assaults | 
| GET	| /actuator/chaosmonkey/assaults | Get assaults payload |
| POST | /actuator/chaosmonkey/assaults | Set assaults payload |

## Trigger Application Chaos and Remediation

Following is an example scenario to trigger infrastructure chaos for the HLA Workshop which will ultimately be detected as an anomaly in ServiceNow

1. Start a Bash or Terminal session and make sure you have curl installed

   ```
   $ curl –version

   curl 7.68.0 (x86_64-pc-linux-gnu) libcurl/7.68.0 OpenSSL/1.1.1f zlib/1.2.11 brotli/1.0.7 libidn2/2.2.0 libpsl/0.21.0 (+libidn2/2.2.0) libssh/0.9.3/openssl/zlib nghttp2/1.40.0 librtmp/2.3
   Release-Date: 2020-01-08
   Protocols: dict file ftp ftps gopher http https imap imaps ldap ldaps pop3 pop3s rtmp rtsp scp sftp smb smbs smtp smtps telnet tftp
   Features: AsynchDNS brotli GSS-API HTTP2 HTTPS-proxy IDN IPv6 Kerberos Largefile libz NTLM NTLM_WB PSL SPNEGO SSL TLS-SRP UnixSockets
   ```

1. Set a TARGET environment variable corresponding to one of your Spring Boot nodes See example below:

   ```
   $ TARGET=http://YOUR SPRING APP PUBLIC IP:8080; export TARGET
   ```

1. Get the assault status from the target server. See example below:

   ```
   $ curl -sX GET $TARGET/actuator/chaosmonkey/status

   You switched me off!
   ```

1. Set the Attack Payload. See example below:

   ```
   $ curl -sX POST $TARGET/actuator/chaosmonkey/assaults \
   -H 'Content-Type: application/json' \
   -d \
   '{
   "level": 5,
   "latencyRangeStart": 2000,
   "latencyRangeEnd": 5000,
   "latencyActive": true,
   "exceptionsActive": true,
   "killApplicationActive": false,
   "exception": {
      "type": "java.lang.IllegalArgumentException",
      "arguments": [
         {
         "className": "java.lang.String",
         "value": "YOUR RANDOM ERROR MESSAGE HERE"
         }
      ]
   }
   }'
   ```

1.	Enable the Assault

      ```
      $ curl -X POST $TARGET/actuator/chaosmonkey/enable -H 'Content-Length:0'

      Chaos Monkey is enabled
      ```

      > NOTE: As long as the assault is enabled, the Spring Application will trigger random Java Exceptions and Traces in the /var/log/spring.log

1.	Disable the Assault. See Example below:

      ```
      $ curl -X POST $TARGET/actuator/chaosmonkey/disable -H 'Content-Length:0'

      Chaos Monkey is disabled
      ```

# APPENDIX A - Trigger Infrastructure Chaos and Remediation

Following is an example scenario to trigger server syslog chaos using a `privileged account` which will ultimately be detected as an anomaly in ServiceNow.

## Prerequisites

* Service Account & SSH Private Key

## Trigger Infrastructure Chaos and Remediation

1. SSH To any Server (e.g. nginx, spring or mysql)

   ```
   ssh ec2-user@$HOST PUBLIC IP -i ~/.ssh/YOUR SSH KEY
   ```

1. Simulate 1 Error to /var/log/messages

   ```
   logger -i -t "chaos" -p user.notice "YOUR RANDOM ERROR MESSAGE HERE" -s
   ```

1. Simulate 10 Errors to /var/log/messages

   ```
   for (( i=1; i<=10; i++ ))
   do  
      sleep .$[($RANDOM % 10)+1]s
      logger -i -t "chaos" -p user.notice "YOUR RANDOM ERROR MESSAGE HERE"" -s
   done
   ```

# APPENDIX B - Database Chaos and Remediation

Following is an example scenario to trigger database chaos using a `privileged account` which will ultimately be detected as an anomaly in ServiceNow.

## Prerequisites

* Service Account & SSH Private Key

## Trigger Database Chaos and Remediation

1. SSH To MySQL Server

   ```
   ssh ec2-user@$HOST PUBLIC IP -i ~/.ssh/YOUR SSH KEY
   ```

1. Set LimitNOFILE=64 to trigger chaos

   ```
   mkdir -p /etc/systemd/system/mariadb.service.d
   cat <<-EOF > /etc/systemd/system/mariadb.service.d/limits.conf  
   [Service]
   LimitNOFILE=64
   EOF
   ```

   > CAUTION: This triggers "[ERROR] Error in accept: Too many open files" errors and makes the database inaccesible for the Spring Apps

1. Restart Service

   ```
   systemctl daemon-reload && systemctl restart mariadb
   ```

1. Tail the Error Log to observe the chaos

   ```
   tail -f /var/log/mariadb/mariadb.log
   ```

1. Set LimitNOFILE=infinite to remediate

   ```
   mkdir -p /etc/systemd/system/mariadb.service.d
   cat <<-EOF > /etc/systemd/system/mariadb.service.d/limits.conf  
   [Service]
   LimitNOFILE=infinite
   EOF
   ```

1. Restart Service

   ```
   systemctl daemon-reload && systemctl restart mariadb
   ```

1. Check Limits (e.g. 962)

   ```
   $ mysql -u root -e "SHOW VARIABLES LIKE 'open_files_limit';"

   +------------------+-------+
   | Variable_name    | Value |
   +------------------+-------+
   | open_files_limit | 962   |
   +------------------+-------+
   ```