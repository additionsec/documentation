---
title: |
    <span id="h.gjdgxs" class="anchor"></span>AdditionSecurity\
    MobileAwareness SDK Overview
---

Purpose
=======

The MobileAwareness SDK is a security library that you embed into your applications, to provide operation and security detection & intelligence benefits. By embedding the MobileAwareness SDK, your applications receive immediate insight into application tampering, device concerns, and network attacks -- allowing your application to flexibly respond in an appropriate manner and consistent with your mobile application operation & experience.

Optionally, all of the information can be sent to a remote network gateway (the AdditionSecurity Messaging Gateway), where the messages are transformed into a message format of choice (JSON, CSV, CEF, LEEF, etc.) and forwarded to a SIEM or archived to a file or AWS S3 (for further possible import into an analytics pipeline). Please see the Messaging Gateway Overview for further details on the Messaging Gateway capabilities and benefits.

MobileAwareness Benefits
========================

By using the MobileAwareness SDK, you receive:

-   Application re-packing/re-signing detection

-   Runtime application tampering detection

-   Debugger, automation, and instrumentation detection

-   Emulator/simulator detection

-   Rooting and jailbreak detection

-   Development and non-production devices

-   Hacking, fraud, and game cheating tools installed on the device

-   Known malware detection

-   Integrity measurements for application executables and critical code sections

-   Application and provisioning profile signer reporting

-   Verification of application encryption present and enabled (IOS/Apple AppStore)

-   Full flexibility to decide how to respond to security events

-   Report existing application authentication info to SDK for unified event tracking

-   Security mechanisms to strongly link the SDK to your application

-   Ability to send messages to a remote network message gateway

-   Integration with existing Security Event & Information Management (SIEM) system

Application Integration
=======================

The MobileAwareness SDK is added to an application at development time. Minimum integration involves only a modest amount of application code changes (adding less than 5 lines of source code), while more complicated integrations -- receiving additional security benefits -- are optional.

The MobileAwareness SDK is particular to the mobile platform (Android, IOS, etc.). You download the SDK for the platform of choice, and follow the Developer Guide documentation for how to integrate the SDK. Support and instructions for use with the most common mobile application development IDEs is included. PhoneGap/Cordova and other “hybrid” mobile application platforms are also accommodated.

Application Impact
==================

Performance and size matter. That’s why the MobileAwareness SDK is written in optimized native code, to keep the performance fast and the speed small. An application experiences less than a 0.2 second delay in initializing the SDK library, and has full startup results within 0.5 seconds. All for under 500Kb added to the application, per supported hardware architecture.

Testing Friendly
================

Well-tested apps are important. The MobileAwareness SDK comes with simulator/emulator friendly versions, and provides flexibility to not disrupt how you choose to conduct your testing & QA -- whether it’s in-house or through testing services such as Amazon AWS Device Farm.

Next Steps
==========

Through the AdditionSecurity customer portal, you can download the platform-appropriate SDK and Developer Guides to get started.
