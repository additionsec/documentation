---
MobileAwareness SDK - Android Java API Reference
---

Associated product: MobileAwareness SDK for Android version 1.3

This document provides full Java API reference information for the MobileAwareness SDK for Android.  For SDK overview information, compatibility information, typical usage scenarios, and example implementation guidance, please see the [Android Developer Guide](ASMA_Android_DeveloperGuide.md).

To use the Java API, the application must include the `asma-standalone.jar` or `asma-embedded.jar` Java library in the application.


API Summary
=============
Addition Security provides the com.additionsecurity.MobileAwareness class containing the following APIs:

*Required use APIs*:

-	**initialize()** - main function to initialize SDK operation for use

*Optional use APIs*:

-   **registerIdentity()** - called after user authentication, to inform the SDK of the user identity for tracking and correlation purposes

-   **loginStatus()** - called after successful & unsuccessful authentication attempts, for authentication tracking

-   **sendMessage()** - send arbitrary data to the remote message gateway

-   **networkEvent()** - inform the SDK of a change in network availability/reachability, triggering network security checks and message flush

-	**heartbeat()** - request the SDK perform an on-demand security re-check

-	**securityPosture()** - retrieve a value representing the current security posture of the device and application

-   **version()** - the version of the embedded SDK

Additionally, Addition Security provides various interfaces and exception definitions in the com.additionsecurity.\* package/namespace.


inititialize( Context, IMobileAwarenessCallback )
================

```
static void initialize( Context ctx, com.additionsecurity.IMobileAwarenessCallback callback )
```

  Inputs    | Description
  -------  | -----------
  ctx | A valid application context
  callback | **Optional**; a function to receive event callbacks; may be null
 
  Throws   | Description
  -------    | -----------
  LicenseException | The configuration file does not contain a valid license for the this application
  ConfigurationFileException | The `as.conf` file is missing or corrupted
  SecurityException | A security-related integrity failure event occurred during initialization;<br>**NOTE:** this should be treated as a security indicator of application tampering
  OperationException | The library failed to initialize and is not usable
  
This is the primary initialization function of the library, and a successful call is required for proper library operation.  This function initializes the library using the `ANDROID_ID` from the Android Secure Settings as an internal device indentifier.  The `as.conf` is retrieved from the applications `assets/` folder.


inititialize( Context, IMobileAwarenessCallback, byte[] )
================

```
static void initialize( Context ctx, com.additionsecurity.IMobileAwarenessCallback callback, byte[] id )
```

  Inputs    | Description
  -------  | -----------
  ctx | A valid application context
  callback | **Optional**; a function to receive event callbacks; may be null
  id | A binary identifier to use for this device 

  Throws   | Description
  -------    | -----------
  LicenseException | The configuration file does not contain a valid license for the this application
  ConfigurationFileException | The `as.conf` file is missing or corrupted
  SecurityException | A security-related integrity failure event occurred during initialization;<br>**NOTE:** this should be treated as a security indicator of application tampering
  OperationException | The library failed to initialize and is not usable
  
This is the primary initialization function of the library, and a successful call is required for proper library operation.  The `as.conf` is retrieved from the applications `assets/` folder.


inititialize( Context, IMobileAwarenessCallback, byte[], byte[] )
================

```
static void initialize( Context ctx, com.additionsecurity.IMobileAwarenessCallback callback, byte[] id, byte[] config )
```

  Inputs    | Description
  -------  | -----------
  ctx | A valid application context
  callback | **Optional**; a function to receive event callbacks; may be null
  id | A binary identifier to use for this device 
  config | Binary contents of an `as.conf` file

  Throws   | Description
  -------    | -----------
  LicenseException | The configuration file does not contain a valid license for the this application
  ConfigurationFileException | The `as.conf` file is missing or corrupted
  SecurityException | A security-related integrity failure event occurred during initialization;<br>**NOTE:** this should be treated as a security indicator of application tampering
  OperationException | The library failed to initialize and is not usable


This is the primary initialization function of the library, and a successful call is required for proper library operation.  The `config` represents the contents of an `as.conf` file passed as a byte[] object.


registerIdentity
===========

```
static void registerIdentity( String identity )
```


   Inputs    | Description
  -------  | -----------
  identity | A string of relevant user identity, e.g. email address
 
  Throws    | Description
  -------    | -----------
  OperationException | Internal operational failure
 
An optional API to associate an application-specific user identity with the existing application session. The registered user identity is used to associate the application instance with a confirmed user identity, for event correlation purposes. The provided identity will be sent in a `IdentityRegistration` message, and subsequently included all messages as `accountName2` to the remote network message gateway. This function can be called multiple times with the same or different values -- only the last value will be used. 

Further, the provided identity will be stored and retrieved on future application restarts automatically (unless the *“Disable persistent registered identity”* configuration option is specified in your `as.conf` configuration), which alleviates the need to call this function upon every application restart. E.g. the application remembers the user authentication and does not perform the authentication process upon successive restarts and thus doesn’t call the `registerIdentity()` function upon successive restarts.

Typical usage would be to call the `registerIdentity()` with a confirmed user login name/email address after the user successfully authenticates to the application, at the natural point when the application prompts the user for authentication.  Alternatively, you can call the function with a session token or other ephemeral identification token.

loginStatus
========

```
static void loginStatus( boolean status )
```

   Inputs    | Description
  -------  | -----------
  status | true = a successful login event; or<br>false = an unsuccessful login event
 


An optional API to to provide authentication status messages to the remote network message gateway. The messages are received and processed by the gateway as `LoginSuccessful` and `LoginUnsuccessful` messages.

Typical usage would call `loginStatus(false)` for each failed authentication attempt to the application, and `loginStatus(true)` (potentially in tandem with `registerIdentity`) upon successful authentication to the application.


sendMessage
=========

```
static void sendMessage( long msgid, String data )
```


   Inputs    | Description
  -------  | -----------
  msgid | An integer value between 0 and 0xffff0000
  data | A string of data to send in the message, of less than 4096 UTF-8 bytes; zero length strings are allowable
 
  Throws    | Description
  -------    | -----------
  OperationException | Internal operational failure
 
 
An optional API to send arbitrary messages/data to the remote network message gateway. The messages are received and processed by the gateway in `CustomerMessage` messages where the message subID is set to the specified `msgid` input value, and the message string is set to the `data` input value.

**The maximum `data` size supported is 4096 UTF-8 bytes.**

Typical usage would allow the application to include arbitrary messages into the remote message data stream; message IDs (`msgid`) would be defined for certain meanings proprietary to the customer, and the data (`data`) would be relevant data for that defined message. Binary data can be either Base64 or ASCII hex-encoded before being provided to this API. Numeric data should be converted to a string before being provided to this API.


networkEvent
=========

```
static void networkEvent( Intent info )
```

   Inputs    | Description
  -------  | -----------
  info | **Optional**; the intent received during a CONNECTIVITY_CHANGED broadcast; may be null


An optional API to inform the library of a network availability change/event. A call will trigger the library to perform any network-particular operations (man-in-the-middle testing, etc.), and attempt to flush any queued messages to the remote network message gateway.

Typical usage would call `networkEvent()` from from a broadcast receiver registered for the CONNECTIVITY_CHANGED broadcast.

heartbeat
========

```
static long heartbeat( long input )
```

   Inputs    | Description
  -------  | -----------
  input | The `Input Nonce` value defined in the `as.conf` configuration
 

  Returns    | Description
  -------    | -----------
  output | The `Output Nonce` value defined in the `as.conf` configuration

**Available in MobileAwareness SDK v1.1 or later**

The optional `heartbeat()` API is used to request an on-demand security re-check by the MobileAwareness SDK.  Typically an application would periodically call `heartbeat()`, particularly before any security-sensitive operations.

The operation of the `input` and `output` values mirror the same operation as found in the MobileAwareness Stealth Callback feature.  Conceptually, you can think of `heartbeat()` as an inverted version of a stealth callback, where the application provides the SDK a specific input value (the `Input Nonce`), the SDK verifies it received the correct value, and returns the expected `Output Nonce`, where the application can verify it received the proper result.  This effectively creates a bidirectional handshake between the application and the SDK.



securityPosture
========

```
static SecurityPosture securityPosture()
```

  Returns    | Description
  -------    | -----------
  SecurityPosture | A SecurityPosture class instance containing various security posture flags

**Available in MobileAwareness SDK v1.2 or later**

An optional API to retrieve the calculated security posture of the device.  When requesting the security posture with `securityPosture()`, the MobileAwareness SDK will first internally perform a security re-check (identical to `heartbeat()`, then return the latest calculated security posture result.

The SecurityPosture class is currently a data container for various security posture flags:

```
    public static class SecurityPosture {
        public boolean completed; // Initialization is complete
        public boolean emulator; // Device is an emulator
        public boolean rooted; // Device is rooted
        public boolean nonProduction; // Device is not production qualified
        public boolean hackingTool; // A hacking tool is installed
        public boolean securityFailure; // Security operation/expectation failed
        public boolean debugger; // A debugger was detected
        public boolean tampering; // App tampering was detected
        public boolean network; // A network attack was detected
        public boolean malware; // Malware was detected
        public boolean cheatOrFraudTool; // A game cheat/fraud tool is installed
        public boolean devBuild; // Development-related app qualities
    }
```



version
========

```
static long version()
```

  Returns    | Description
  -------    | -----------
  Library version | An integer representing the MobileAwareness SDK build version


An optional API to retrieve the library version number.


Message Callback
================
The IMobileAwarenessCallback interface defines the onMessage method necessary for implementing a MobileAwareness SDK Java callback.

```
void onMessage(int msgId, int msgSubId, byte[] data1, byte[] data2)

```

   Inputs    | Description
  -------  | -----------
  msgId | A non-zero numerical identifier representing a specific event/message
  msgSubId| An optional non-zero numerical ID representing a secondary identifier for the given `msgId`; or zero if not used
  data1 | An optional data value related to the particular specified `msgId`, or NULL
  data2 | An optional data value related to the particular specified `msgId`, or NULL


The message callback mechanism is the primary method for the application to internally receive feedback and respond to events, messages, and observations produced by the MobileAwareness SDK.


***Notes on Data1/Data2***

The contents of `data1`/`data2` are particular to the `msgId`, and may be binary bytes or a UTF-8 encoded string.

<div class="alert alert-danger">Both data1 and data2 may be NULL; check values before using</div>



Stealth Callback
===================


```
static long callback(long input)

```

   Inputs    | Description
  -------  | -----------
  input | An incoming value from the SDK; it will match the `Input Nonce` value included in your `as.conf` configuration

   Returns    | Description
  -------  | -----------
  output | An appropriate return value; it must match the `Output Nonce` value included in your `as.conf` configuration
  


