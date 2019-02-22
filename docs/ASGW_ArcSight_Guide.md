---
subtitle: '<span id="h.30j0zll" class="anchor"></span>Version 204010101'
title: '<span id="h.gjdgxs" class="anchor"></span>AdditionSecurity Messaging Gateway Integration with HP ArcSight'
---

Overview
========

This document contains instructions in configuring the AdditionSecurity Messaging Gateway to forward messages to an HP ArcSight SIEM system.

Prerequisites
=============

It is assumed:

-   The reader has administrative & configuration understanding of HP ArcSight Logger

-   The reader has read the AdditionSecurity Messaging Gateway Operations Guide

-   The deployed Messaging Gateway(s) will be co-located or otherwise able to communicate over a network to the ArcSight system(s)

Configuration Summary
=====================

This guide will recommend an AdditionSecurity Messaging Gateway to send CEF-formatted messages over syslog to an HP ArcSight Logger system on UDP port 8514.

Configuring HP ArcSight Logger
==============================

1.  Log into the ArcSight Logger web console

2.  Click on the Configuration tab

3.  On the left, select Event Input

4.  Click Add

5.  Type a name for your receiver

6.  Select UDP Receiver | CEF UDP Receiver

7.  Click Next

8.  Select the IP addresses the ArcSight Logger will use for incoming traffic

9.  Enter port 8514

10. Under source type, select CEF

11. Click Save

12. Enable the receiver

The CEF syslog log source has been added to ArcSight Logger.

Configuring AdditionSecurity Message Gateway
============================================

The relevant configuration properties from the Message Gateway config.properties file are:

```
port=5000

definitions=definitions.json

input=protobuf

transform=cef

transform.systemId2\_string=true

transform.accountName2\_string=true

transform.include\_title=true

output=udpsyslog

output.syslog.host=(IP address or hostname of ArcSight Logger system)

output.syslog.port=8514
```

This instructs the gateway to transform messages into the CEF format, then forward the messages over UDP syslog to the ArcSight system on port 8514.
