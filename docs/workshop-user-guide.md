---
title: Predictive AIOps Workshop User Guide
subtitle: Predictive AIOps Workshop 
author: 
- Anthony Angelo, Netser Heruty, Daniel Smith
date: 10 Nov 2021
---

# Introduction

This document provides prescriptive guidance for generating Chaos in the Predictive AIOps Workshop environment to simulate anomalies in the Pet Clinic application. 

![Figure 1](workshop-chaos-architecture.png)

# Log into your NOW Instance

1. Login to your NOW Instance with your given credentials
1. Set your `Profile Time Zone` accordingly (e.g. `US\Eastern`)

   > NOTE: Log out and back in to make sure your `Profile Time Zone` is set correctly. Failure to do so will adversly affect the workshop and using basic HLA functions like searching and finding log entries.

# Trigger Chaos using the Chaos Catalog

Your Workshop instance has been provisioned with an _easy-to-use_ Chaos Catalog in order to help participants quicly dispatch chaos to different servers throughout the Workshop architecture. Access to the Chaos Catalog is as follows:

1. Navigate to `https://YOUR INSTANCE.service-now.com/chaos` 
1. You should see the following Chaos Catalog Page:

   ![chaos-catalog](chaos-catalog.png)

## Trigger Application Chaos

The Workshop Application Chaos leverages the `Chaos Monkey for Spring Boot` framework, which is an open-source project for Java Spring based applications used to simulate & control service degradations, random errors & stack traces. Please refer to the Chaos Monkey Project https://codecentric.github.io/chaos-monkey-spring-boot for more information. The Spring Boot application layer has been built and injected with the Chaos Monkey libraries which in turn, trigger service degradations and random errors, otherwise known as chaos. This simulated chaos is eventually detected by the HLA system as anomalies in any of the Spring Servers.

Use the Chaos Catalog as follows to create Application Chaos:

1. Select **Home> Predictive AIOps Workshop Error Generation > Application Chaos**

   ![chaos-application](chaos-application.png)

1. Select one or more servers in the `Select a server or multiple servers on which to trigger application chaos` field

1. Select an Exception Type in the `Select the type of Java Runtime exception you want Chaos Monkey to generate` field

1. Enter a random error message in the `Enter a random application error message` field

1. Press `Submit` to process request

## Trigger Infrastructure Chaos using the UNIX Logger API

The Workshop Application Chaos leverages the UNIX `logger` API which allows privileged users to simulate any log messages so that they are processed by the SYSLOG system and eventually broacasted to the `/var/log/messages` log files with the correct timestamp and log levels. This simulated chaos is eventually detected by the HLA system as anomalies in any of the Spring Servers. Use the Chaos Catalog as follows to trigger Infrastructure Chaos using the UNIX Logger API:

1. Select **Home> Predictive AIOps Workshop Error Generation > Infrastructure Chaos**

   ![chaos-infrastructure-logger](chaos-infrastructure-logger.png)

1. Select one or more servers in the `Select a server or multiple servers on which to generate infrastructure errors` field

1. Enter a random error message in the `Enter a random error message you want added to /var/log/messages on the target server` field

   >NOTE: You will need to include keywords in your error message associated with infrastructure failures such as `crash`, `failure`, `error`, `issue`, `warning`, etc.

1. Press `Submit` to process request

## Trigger Infrastructure Chaos using the Stress API

The Workshop Application Chaos leverages the UNIX `stress` API which allows privileged users to simulate memory and CPU load. These metrics (along with many others) are constantly being streamed to NOW for processing. This simulated chaos is eventually detected by the Alert Intelligence system as anomalies in any of the Spring Servers. Use the Chaos Catalog as follows to trigger Infrastructure Chaos using the Stress API:

1. Select **Home> Predictive AIOps Workshop Error Generation > Infrastructure Chaos**

   ![chaos-infrastructure-stress](chaos-infrastructure-stress.png)

1. Select one or more servers in the `Select a server or multiple servers on which to generate infrastructure errors` field

1. Enter a random error message in the `Enter a random error message you want added to /var/log/messages on the target server` field

   >NOTE: You will need to include keywords in your error message associated with infrastructure failures such as `crash`, `failure`, `error`, `issue`, `warning`, etc.

1. Press `Submit` to process request

# APPENDIX A - Trigger Application Chaos using REST

Following is an example scenario to trigger application chaos for the Predictive AIOps Workshop using the Chaos Monkey REST API which will ultimately be detected as an anomaly in ServiceNow.

## Chaos Monkey API Explained

The Chaos Monkey assaults API is accessible via REST both via HTTP GET from a browser to simply query for status, as well as HTTP PUT to configure new assaults. Chaos Monkey is initially deployed in a disabled state to minimize impact to the initial Predictive AIOps Workshop deployment and scenarios.  The Chaos Monkey APIs are as follows:

| HTTP Method | REST API | Description |
| ----------- | -------- | ----------- |
| GET	| /actuator/chaosmonkey/status |	Get status of the assaults |
| POST | /actuator/chaosmonkey/enable | Enable assaults |
| POST | /actuator/chaosmonkey/disable | Disable assaults | 
| GET	| /actuator/chaosmonkey/assaults | Get assaults payload |
| POST | /actuator/chaosmonkey/assaults | Set assaults payload |

## Trigger Application Chaos using REST

Following is an example scenario to trigger infrastructure chaos for the Predictive AIOps Workshop which will ultimately be detected as an anomaly in ServiceNow

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

# APPENDIX B - Trigger Infrastructure Chaos

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

   > NOTE: Use an HLA Lexical Keyword so that HLA can notice the log faster (e.g., error, failed, fatal, corrupt, etc.). For more information please refer to the [HLA Lexical Keyword](https://docs.servicenow.com/bundle/quebec-it-operations-management/page/product/health-log-analytics-admin/task/hla-lexical-keywords-admin.html) documentation

# APPENDIX C - Trigger Infrastructure Stress Chaos

Following is an example scenario to trigger `stress` chaos using a `privileged account` which will ultimately be detected as an anomaly in ServiceNow.

## Prerequisites

* Service Account & SSH Private Key
* Install `stress` as follows:

   ```
   sudo amazon-linux-extras install epel -y
   sudo yum install stress -y
   ```

## Trigger Infrastructure Stress Chaos and Remediation

1. SSH To MySQL Server

   ```
   ssh ec2-user@$HOST PUBLIC IP -i ~/.ssh/YOUR SSH KEY
   ```

1. Get Number of processors

   ```
   getconf _NPROCESSORS_ONLN
   ```

   >NOTE: To fully stress the CPU, this should be the total number of CPU cores or double that if the CPU supports hyper-threading

1. Run CPU Stress (5 mins)

   ```
   stress -v -c $(getconf _NPROCESSORS_ONLN) -t 5m
   ```

1. Run memory stress (5 mins)

   ```
   CPUS=$(getconf _NPROCESSORS_ONLN); stress -m $CPUS --vm-bytes $(awk -v cpus=$CPUS '/MemAvailable/{printf "%d\n", $2 / cpus;}' < /proc/meminfo)k --vm-keep  -t 5m
   ```

# APPENDIX D - Database Chaos and Remediation

Following is an example scenario to trigger database chaos using a `privileged account` which will ultimately be detected as an anomaly in ServiceNow.

## Prerequisites

* Service Account & SSH Private Key

## Trigger Database Chaos and Remediation
> Requires SSH access - ask your SN-rep to do this with you during a touchpoint meeting!

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
