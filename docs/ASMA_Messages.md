---
Mobile RASP SDK - Messages Reference
---

Associated product: Mobile RASP SDK version 2.0

The following chart overviews the various messages reported by the Mobile RASP SDK.

## Messages Table Legend / Definitions

Identifer | Description
----- | ------
SHA1 | SHA1 hash, 20 binary bytes
SHA256 | SHA256 hash, 32 binary bytes
X509 Subject | X.509 certificate subject
X509 Certificate | X.509 certificate data, DER encoding
SPKI Hash | SHA256 binary hash of X.509 certificate Subject Public Key Info (SPKI)

## Messages

Platform | MsgID | MsgSubID | Description | Data1 | Data2
----- | ----- | ----- | ----- | ----- | -----
A,I | 1 | 0 | System Hardware Info | Hardware model (string) | -
A,I | 2 | 0 | System Firmware Info | Firmware version (string) | -
A,I | 3 | 0 | System OS Info | OS version (string) | -
A,I | 4 | 0 | Application Info | Application version (string) | -
A,I | 8 | 0 | SDK Version Info | `as.def` version (binary) | `as.conf` timestamp (binary)
A,I | 50 | 0 | Initialization Complete | - | -
A,I | 55 | 0 | Background Monitoring Disabled | - | -
A,I | 150 | PISID | Known Malware Artifact Detected | Artifact string (string) | -
A,I | 151 | PISID | Public or Stolen Cert Signer Present | Application name (string) | SHA1 (binary)
A,I | 152 | PISID | Known Malware Signer Present | Application name (string) | SHA1 (binary)
A,I | 300 | PISID | Synthetic System (Emulator) | Artifact string (string) | -
A | 302 | PISID | Non-Production System | Artifact string (string) | -
A,I | 305 | PISID | Privilege-Providing Application Installed | Artifact string (string) | -
A,I | 307 | PISID | Hacking Tool Installed | Artifact string (string) | -
A,I | 308 | PISID | Security Subversion Tool Installed | Artifact string (string) | -
A,I | 309 | PISID | Application Tampering Tool Installed | Artifact string (string) | -
A,I | 310 | PISID | Game Cheat Tool Installed | Artifact string (string) | -
A,I | 311 | PISID | In-App Purchasing/Fraud Tool Installed | Artifact string (string) | -
A,I | 312 | PISID | Test/Automation Tool Installed | Artifact string (string) | -
A,I | 313 | PISID | Security Expectation Failure | Artifact string (string) | -
A,I | 314 | 0 | System is Rooted/Jailbroken | - | -
A,I | 315 | PISID | Security Hiding Tool Installed | Artifact string (string) | -
A | 316 | 0 | System Signer | SHA1 (binary) | -
A | 317 | 0 | System Unsigned | - | -
A | 318 | 0 | ADB Daemon Running | - | -
A,I | 400 | PISID | Debug/Instrumentation Artifact | Artifact string (string) | -
A,I | 401 | PISID | Application Tampering Detected | Artifact string (string) | -
I | 402 | 0 | Application Unencrypted | - | -
I | 403 | 0 | Application Encryption Disabled | - | -
I | 404 | 0 | Application Signer - Executable | SHA256 (binary) | X509 Subject (string)
A | 404 | 0 | Application Signer - APK | SHA256 (binary) | X509 Subject (string)
A,I | 405 | 0 | Application Unsigned | - | -
A,I | 406 | 0 | Stealth Callback Failure | - | -
I | 407 | 1 | Application Measurement - Executable file | SHA256 (binary) | File path (string)
A | 407 | 1 | Application Measurement - Shared library .so file | SHA256 (binary) | File path (string)
A | 407 | 10 | Application Measurement - APK file | SHA256 (binary) | APK file path (string)
I | 408 | 0 | Provisioning Signer | SHA256 (binary) | X509 Subject (string)
I | 409 | 0 | Provisioning Missing | - | -
A | 409 | 0 | Provisioning Missing | - | -
I | 410 | 0 | Provisioning Corrupted | - | -
A,I | 411 | PISID | Security Operation Failed | - | -
A,I | 412 | 0 | Stealth Callback Timeout | - | -
A | 413 | 0 | Application is Developer Signed | SHA1 (binary) | X509 Subject (string)
A | 414 | 0 | Debug Build | - | -
A | 415 | 0 | Provisioning Provider | Installer package name (string) | -
A,I | 416 | 0 | Heartbeat Failure | - | -
A | 417 | 0 | ARM Emulation on X86 Device | - | -
A,I | 500 | 0 | SSL Pin Violation | hostname (string) | SPKI Hash (binary)
A,I | 502 | 0 | SSL Pin Violation Certificate | hostname (string) | X509 Cert (binary)



