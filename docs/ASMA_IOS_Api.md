---
MobileAwareness SDK - IOS API Reference
---

Associated product: MobileAwareness SDK for IOS version 1.3

This document provides full API reference information for the MobileAwareness SDK for IOS.  For SDK overview information, compatibility information, typical usage scenarios, and example implementation guidance, please see the [IOS Developer Guide](ASMA_IOS_DeveloperGuide.md).



API Summary
=============

*Required use APIs*:

-   **AS\_Initialize()** - main function to initialize SDK operation


*Optional use APIs*:

-   **AS\_Register\_Identity()** - called after user authentication, to inform the SDK of the user identity for tracking and correlation purposes

-   **AS\_Login\_Status()** - called after successful & unsuccessful authentication attempts, for authentication tracking

-   **AS\_Send\_Message()** - send arbitrary data to the remote message gateway

-   **AS\_Network\_Reachability()** - inform the SDK of a change in network availability/reachability, triggering network security checks and message flush

-	**AS\_Heartbeat()** - request the SDK perform an on-demand security re-check

-	**AS\_Security\_Posture()** - retrieve a value representing the current security posture of the device and application

-   **AS\_Version()** - the version of the embedded SDK

AS\_Inititialize
================

```
Objective-C:
int AS_Initialize( const uuid_t deviceid, void (*callback)(int,int,CFDataRef,CFDataRef) )

Swift 2/3:
int AS_Initialize( const uuid_t deviceid, void (*callback)(Int32,Int32,CFData?,CFData?) )
```

  Inputs    | Description
  -------  | -----------
  deviceid | A device identifier value (see Notes)
  callback | **Optional**; a function to receive event callbacks; may be null/nil
 
  Returns    | Description
  -------    | -----------
  AS\_INIT\_SUCCESS | Successful Initialization
  AS\_INIT\_ERR\_LICENSE | The configuration file does not contain a valid license for the this application
  AS\_INIT\_ERR\_INTEGRITY | A security-related integrity failure event occurred during initialization;<br>**NOTE:** this should be treated as a security indicator of application tampering
  AS\_INIT\_ERR\_OLDCONFIG | The as.conf configuration format is out of date; re-generate a new as.conf file to bundle with the application
  AS\_INIT\_ERR\_ALREADYINIT | The AS_Initialize() call was already successfully performed;<br>**NOTE:** the provided callback is still used, allowing updates to the callback after first initialization
  AS\_INIT\_ERR\_GENERAL | The library failed to initialize and is not usable
  

This is the primary initialization function of the library, and a successful call is required for proper library operation.

***Note on DeviceID***

The library uses a provided `uuid_t` as the device identifier in all messages sent to the remote network message gateway. It is recommended to use the IOS *“Identifier For Vendors”* (IDFV) for the DeviceID, but an alternate identifier (IDFA, UDID, etc.) can be specified. The value should be persistent across application launches, and unique (minimally) per device.

*Objective-C Example:*
```
uuid_t deviceid;
[[UIDevice currentDevice].identifierForVendor.getUUIDBytes:deviceid];
int result = AS_Initialize( deviceid, … );
```

*Objective-C Example 2:*
```
AS_UUID_DEFAULT_IDFV(deviceid);
int result = AS_Initialize( deviceid, … );
```

*Swift 2 Example:*
```
var deviceid : [UInt8] = [UInt8](count:16, repeatedValue:0)
Device.currentDevice().identifierForVendor!.getUUIDBytes(&deviceid)
let result : CInt = AS_Initialize( deviceid, … );
```

*Swift 3 Example:*
```
var uuidBytes : uuid_t = UIDevice.current.identifierForVendor!.uuid
let deviceid : UnsafePointer<UInt8> = Data(bytes:&uuidBytes, count:16).withUnsafeBytes { UnsafePointer<UInt8>($0) }
let result : CInt = AS_Initialize( deviceid, … );
```

***Note on Callback***

Messages can be sent to an internal specified function callback, separate from being sent to a remote network message gateway. This allows the application to monitor and respond to the ongoing messages/observations.  You can change the callback (or disable it) by calling AS_Initialize() again with a new callback value (or NULL/nil); the call will return AS_INIT_ERR_ALREADYINIT but will utilize the newly provided callback.



AS\_UUID\_DEFAULT\_IDFV Macro (Objective-C Only)
=========

```
AS_UUID_DEFAULT_IDFV(variable_name)
```

   Inputs    | Description
  -------  | -----------
  variable_name | The name of a `uuid_t` variable to define and populate with the IOS *"Identify For Vendor"* value
  
For convenience, Addition Security provides the `AS_UUID_DEFAULT_IDFV` macro to simplify the usage of the IDFV value with an `AS_Initialize()` call.


AS\_CALLBACK Macro (Objective-C Only)
=============

```
AS_CALLBACK(callback_name)
```
   Inputs    | Description
  -------  | -----------
  callback_name | The name to provide the callback function

For convenience, Addition Security provides the `AS_CALLBACK` macro to simplify callback definition.  The resulting macro execution produces a method declaration as:

`void callback_name(int msgid, int msgsubid, CFDataRef data1, CFDataRef data2)`

Example Objective-C usage:

```
AS_CALLBACK(callback)
{
	// parameters are msgid, msgsubid, data1, data2
	// ...
}

// ...
AS_UUID_DEFAULT_IDFV(deviceid);
int result = AS_Initialize( deviceid, callback );
```



AS\_Register\_Identity
===========

```
Objective-C:
int AS_Register_Identity( const char *identity )

Swift 2/3:
AS_Register_Identity( identity:String ) -> CInt
```


   Inputs    | Description
  -------  | -----------
  identity | A NULL-terminated UTF-8 string of relevant user identity, e.g. email address
 
  Returns    | Description
  -------    | -----------
  AS\_SUCCESS | Successful registration
  AS\_ERR\_GENERAL | Internal operational failure
 
An optional API to associate an application-specific user identity with the existing application session. The registered user identity is used to associate the application instance with a confirmed user identity, for event correlation purposes. The provided identity will be sent in a `IdentityRegistration` message, and subsequently included all messages as `accountName2` to the remote network message gateway. This function can be called multiple times with the same or different values -- only the last value will be used. 

Further, the provided identity will be stored and retrieved on future application restarts automatically (unless the *“Disable persistent registered identity”* configuration option is specified in your `as.conf` configuration), which alleviates the need to call this function upon every application restart. E.g. the application remembers the user authentication and does not perform the authentication process upon successive restarts and thus doesn’t call the `AS_Register_Identity()` function upon successive restarts.

Typical usage would be to call the `AS_Register_Identity()` with a confirmed user login name/email address after the user successfully authenticates to the application, at the natural point when the application prompts the user for authentication.  Alternatively, you can call the function with a session token or other ephemeral identification token.

<div class="alert alert-danger">The remote message gateway expects the string to conform to UTF-8 encoding; alternate encodings may lead to decoding errors and corrupted values when transformed by the remote gateway.</div>


AS\_Login\_Status
========

```
Objective-C:
void AS_Login_Status( int status )

Swift 2/3:
AS_Login_Status( status:CInt ) -> Void
```

   Inputs    | Description
  -------  | -----------
  status | 1 = a successful login event; or<br>0 = an unsuccessful login event
 


An optional API to to provide authentication status messages to the remote network message gateway. The messages are received and processed by the gateway as `LoginSuccessful` and `LoginUnsuccessful` messages.

Typical usage would call `AS_Login_Status(0)` for each failed authentication attempt to the application, and `AS_Login_Status(1)` (potentially in tandem with `AS_Register_Identity`) upon successful authentication to the application.


AS\_Send\_Message
=========

```
Objective-C:
int AS_Send_Message( uint32_t msgid, const char *data )

Swift 2/3:
AS_Send_Message( msgid:UInt32, data:String ) -> CInt
```


   Inputs    | Description
  -------  | -----------
  msgid | An integer value between 0 and 0xffff0000
  data | A NULL-terminated UTF-8 string of data to send in the message, of less than 4096 bytes; zero length strings are allowable
 
  Returns    | Description
  -------    | -----------
  AS\_SUCCESS | Successful registration
  AS\_ERR\_GENERAL | Internal operational failure
 
 
An optional API to send arbitrary messages/data to the remote network message gateway. The messages are received and processed by the gateway in `CustomerMessage` messages where the message subID is set to the specified `msgid` input value, and the message string is set to the `data` input value.

**The maximum `data` size supported is 4096 bytes.**

Typical usage would allow the application to include arbitrary messages into the remote message data stream; message IDs (`msgid`) would be defined for certain meanings proprietary to the customer, and the data (`data`) would be relevant data for that defined message. Binary data can be either Base64 or ASCII hex-encoded before being provided to this API. Numeric data should be converted to a string before being provided to this API.

<div class="alert alert-danger">The remote message gateway expects the string to conform to UTF-8 encoding; alternate encodings may lead to decoding errors and corrupted values when transformed by the remote gateway.</div>



AS\_Network\_Reachability
=========

```
Objective-C:
void AS_Network_Reachability()

Swift 2/3:
AS_Network_Reachability() -> Void
```


An optional API to inform the library of a network availability change/event. A call will trigger the library to perform any network-particular operations (man-in-the-middle testing, etc.), and attempt to flush any queued messages to the remote network message gateway.

Typical usage would call `AS_Network_Reachability()` from any network availability/monitoring operations performed by the application.


AS\_Heartbeat
========

```
Objective-C:
long AS_Heartbeat( long input )

Swift 2/3:
AS_Heartbeat( input:CLong ) -> CLong
```

   Inputs    | Description
  -------  | -----------
  input | The `Input Nonce` value defined in the `as.conf` configuration
 

  Returns    | Description
  -------    | -----------
  output | The `Output Nonce` value defined in the `as.conf` configuration

**Available in MobileAwareness SDK v1.1 or later**

The optional `AS_Heartbeat()` API is used to request an on-demand security re-check by the MobileAwareness SDK.  Typically an application would periodically call `AS_Heartbeat()`, particularly before any security-sensitive operations.

The operation of the `input` and `output` values mirror the same operation as found in the MobileAwareness Stealth Callback feature.  Conceptually, you can think of `AS_Heartbeat()` as an inverted version of a stealth callback, where the application provides the SDK a specific input value (the `Input Nonce`), the SDK verifies it received the correct value, and returns the expected `Output Nonce`, where the application can verify it received the proper result.  This effectively creates a bidirectional handshake between the application and the SDK.



AS\_Security\_Posture
========

```
Objective-C:
uint32_t AS_Security_Posture()

Swift 2/3:
AS_Security_Posture() -> UInt32
```

  Returns    | Description
  -------    | -----------
  Security value | An integer representing the security posture of the device and application


**Available in MobileAwareness SDK v1.2 or later**

An optional API to retrieve the calculated security posture of the device.  The resulting value can be processed by the included `AS_SECURITY_*` macros to derive specific security meaning.

When requesting the security posture with `AS_Security_Posture()`, the MobileAwareness SDK will first internally perform a security re-check (identical to `AS_Heartbeat()`, then return the latest calculated security posture result.


AS\_SECURITY\_\* Macros
=======
```
Objective-C:
AS_SECURITY_INITCOMPLETED(security_value)
AS_SECURITY_JAILBROKEN(security_value)
AS_SECURITY_HACKINGTOOL(security_value)
AS_SECURITY_SECURITYVIOLATION(security_value)
AS_SECURITY_DEBUGGER(security_value)
AS_SECURITY_TAMPERING(security_value)
AS_SECURITY_NETWORK(security_value)
AS_SECURITY_MALWARE(security_value)
AS_SECURITY_GAMECHEATTOOL(security_value)
AS_SECURITY_DEVBUILD(security_value)

Swift 2/3:
(See note, below)
```

   Inputs    | Description
  -------  | -----------
  security_value | The result from `AS_Security_Posture()`

**Available in MobileAwareness SDK v1.2 or later**

The `AS_SECURITY_*` macros provide a convenient way to test the security value returned by `AS_Security_Posture()`.  Typically the `AS_SECURITY_*` macros would be used in conditional test code.  

*Objective-C example:*
```
uint32_t security_value = AS_Security_Posture();

if( AS_SECURITY_JAILBROKEN(security_value) 
	|| AS_SECURITY_DEBUGGER(security_value)
	|| AS_SECURITY_TAMPERING(security_value) ) {

	// ... react to a jailbroken, debugger, and/or tampering posture	
}
```

<div class="alert alert-info">Due to the lack of macro support in Swift, Swift developers will need to consult the macro values in the as_mobileawareness.h file to properly process the security_value value.</div>



AS\_Version
========

```
Objective-C:
uint32_t AS_Version()

Swift 2/3:
AS_Version() -> UInt32
```

  Returns    | Description
  -------    | -----------
  Library version | An integer representing the MobileAwareness SDK build version


An optional API to retrieve the library version number.


Message Callback
================

```
Objective-C:
void callback(int msgId, int msgSubId, CFDataRef data1, CFDataRef data2)

Swift 2/3:
callback(msgid:Int32, msgsubid:Int32, data1:CFData?, data2:CFData?) -> Void
```

   Inputs    | Description
  -------  | -----------
  msgId | A non-zero numerical identifier representing a specific event/message
  msgSubId| An optional non-zero numerical ID representing a secondary identifier for the given `msgId`; or zero if not used
  data1 | An optional data value related to the particular specified `msgId`, or NULL
  data2 | An optional data value related to the particular specified `msgId`, or NULL


The message callback mechanism is the primary method for the application to internally receive feedback and respond to events, messages, and observations produced by the MobileAwareness SDK.

<div class="alert alert-danger">Both data1 and data2 may be NULL; check values before using</div>

The contents of `data1`/`data2` are particular to the `msgId`, and may be binary bytes or a UTF-8 encoded string.

***Objective-C Notes on Data1/Data2***

The defined data type for `data1` and `data2` is `CFDataRef`. These values can be cast to the more familiar `NSData*` equivalents in an ARC-supportable manner via the following Objective-C code:

```
NSData *d1 = (__bridge NSData*)data1;
NSData *d2 = (__bridge NSData*)data2;
```


Stealth Callback
===================

```
Objective-C:
(uint32_t) callback:(uint32_t)input

Swift 2/3:
@objc static func callback(input:UInt32) -> UInt32
```

   Inputs    | Description
  -------  | -----------
  input | The expected input nonce value

Returns    | Description
  -------    | -----------
  output | The expected output nonce value

  
A stealth callback is optionally defined in your `as.conf` configuration file.  Also defined in your `as.conf` file are the expected `input` and `output` nonces.

<div class="alert alert-danger">Please see the IOS Developers Guide for information regarding stealth callback naming conventions within Swift applications.</div>
