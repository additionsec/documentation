---
Addition Security Message Gateway Operations Guide
---

Overview
========

The Messaging Gateway is a standalone Java 8-based application designed to receive messages from the Addition Security MobileAwareness SDK, transform the messages into a desired data representation format (e.g. CEF, LEEF, JSON, etc.) and forward the messages to a data storage or processing node (e.g. forward via syslog, save to a local file, upload to AWS S3, etc.).

This guide relates to Addition Security Messaging Gateway version 1.4.


Features
========

The Messaging Gateway can receive messages utilizing the Addition Security Operational Intelligence message format, used by all Addition Security products.

Once a message is received, it can be transformed into a selected output format:

-   **KVP (Key-Value Pair)** - a simple “key=value” representation that is suitable for transporting over syslog, importing into SIEMs, or providing a one-line-per-message output

-   **CSV (Comma Separated Value)** - a common database and spreadsheet format that is more concise than KVP since only the values are supplied (keys are specified in the initial header)

-   **HP ArcSight CEF (Common Event Format)** - a specified format used by the HP ArcSight SIEM, suitable for transport over syslog

-   **IBM QRadar LEEF (Log Event Extended Format)** - a specified format used by the IBM QRadar SIEM, suitable for transport over syslog

-   **JSON (JavaScript Object Notation)** - a common format used for data transit

After transformation, the messages are forwarded via a selected output method:

-   **Saved to a local file** - suitable for testing, saving data to a network file share, or providing the data to a secondary process on the system that is monitoring file output

-   **Displayed as console output** - suitable for testing or stdout pipe processing to a second receiving process/script

-   **Forwarded via syslog** - the industry standard to send logging data; suitable for consumption by SIEMs and log aggregation/consumption tools

-   **Batched uploaded to AWS S3** - messages are aggregated and sent at batch intervals to an Amazon hosted S3 bucket; the files can then be imported from S3 into data analytics tools (AWS EMR, Apache Hadoop) for analysis, or processed real-time via AWS Lambda functions

The Messaging Gateway is designed to be internally stateless and asynchronous in operation (based on Java Netty), allowing it to be highly scalable in vertical (single system w/ multiple cores) and horizontal (multiple systems in parallel) with simplicity. When combined with a preceding load balancer, a set of Messaging Gateways can easily meet any amount of incoming traffic demands.

To further simplify deployment, the Messaging Gateway distribution has been tailored for easy deployment via AWS Elastic Beanstalk and Docker, providing a simple approach to immediately scalable hosting of the Messaging Gateway in the cloud with no infrastructure overhead.

Prerequisites
=============

System Specifications
---------------------

The Messaging Gateway runs on Java 8, and as such, can conceivably be hosted on any system that supports Java 8. Beyond that, the memory requirements are moderate and the application is relatively CPU-bound due to the data transformations and network-bound to receive and forward the resulting data (or I/O bound if writing output to a local disk file). The application is multi-core and multi-thread aware, and will take advantage of multiple available cores by default.

### Required

-   Java JRE or JDK 8

Tested Java versions:

-   64-bit Oracle Java 8u65 (1.8.0\_65)

-   64-bit OpenJDK 1.8.0\_71 \[AWS Elastic Beanstalk\]

### Recommended For Production

-   64-bit multi-core CPU

-   Linux or WIndows 64-bit OS

-   64-bit Java 8 JRE (Oracle or OpenJDK)

-   1GB or more of RAM usable by the application

-   8GB of temporary disk storage

<div class="alert alert-info">NOTE: if using the file storage output configuration, then you will need appropriate local disk space for storing the output</div>

### Consideration: Temporary Files

The Message Gateway uses the Java standard `java.io.tmpdir` configuration property to indicate a valid, writable directory for creating temporary files. You can change the location of temporary directory by invoking the gateway with the `-Djava.io.tmpdir=(location)` startup parameter.

Temporary files are used in a few ways:

-   Large or slow uploads may need to be temporarily spooled to disk until processed

-   The `s3` output module batches output messages to disk in the event of an S3 accessibility error/network outage


Application Components
======================

The Messaging Gateway application distribution (.zip) file contains the following components:


| Component | Description |
| ----- | ----- |
| asgw.jar                            | The self-contained standalone Messaging Gateway application, distributed as a Java 8 JAR file
|  config.properties                   | The configuration file used by the Messaging Gateway application
| definitions.json                    | An (optional) meta-data file used to decode message IDs and SubIDs for more meaningful message titles when transforming received messages
| deploy\_awsbeanstalk/asgw\_eb.zip   | A base container file for deploying the Messaging Gateway to AWS Elastic Beanstalk; **NOTE: a config.properties file must be added to asgw\_eb.zip before it is suitable for deployment**
| deploy\_docker/Dockerfile           | A Docker image definition for use with Docker deployments

Networking
==========

The Messaging Gateway implements an HTTP 1.0/1.1 server for incoming GET and POST requests.

The following table lists the various URL endpoints supported by the Messaging Gateway:

| URL | Description |
| ----- | ----- |
|  /                |(GET requests) A simple URL for health checking (e.g. AWS ELB); always returns 200 OK response
|  /v1/msg          |(POST requests) Receive input message(s) from a remote system for processing; returns 200 on success, or 500 on processing error
|  /v1/msg          |(GET requests) Always returns a 200 OK response; used by MobileAwareness clients as a network accessibility check endpoint
| All other URLs   | Returns a 404 Not Found response


### HTTPS Support

The Messaging Gateway implements an HTTP server, that should be secured by the use of SSL/TLS.

For secure production envrionments, it is strongly recommended to use an HTTPS-terminating load balancer (e.g. AWS ELB) or HTTP forwarder (e.g. Nginx) in front of the Messaging Gateway. The Messaging Gateway is aware of X-Forwarded-For header semantics to determine the original message submitter information.

For low-volume and test environments, the Messaging Gateway includes optional built-in basic SSL support.


Message Transform/Output Formats
================================

The following overviews the particular message transform formats supported by the Message Gateway.

### CEF (HP ArcSight)

The Common Event Format is a particular format defined for use by the HP ArcSight SIEM product. CEF messages begin with a fixed header prefix that resembles:

`CEF:0|AdditionSecurity|ASGW|1.0|(EventID)|(EventTitle)|(Severity)|...`

And contains optional extension key-value pairs using a defined naming convention.  A CEF format reference guide is available at [here](https://www.protect724.hpe.com/docs/DOC-1072).

Example full CEF message (single line):

```
CEF:0|AdditionSecurity|ASGW|1.0|1001|-|3|start=Apr 01 2018 09:04:06 dvc=10.9.0.10 
cs1Label=suser2 cs1=myuser@example.com cs2Label=deviceType cs2=Android deviceProcessName=com.example.app 
cn1Label=eventSubId cn1=2002 cs4Label=model cs4=phone cs5Label=file cs5=/dev/null cn2Label=number cn2=42
```

### LEEF (IBM QRadar)

The Log Event Extended Format is a particular format defined for use by the IBM QRadar SIEM product. LEEF messages begin with a fixed header prefix that resembles:

```
LEEF|1.0|AdditionSecurity|ASGW|1.0|(EventID)|...
```

And contains optional extension key-value pairs using a defined naming convention.  A LEEF format reference guide is available [here](https://www.ibm.com/developerworks/community/wikis/form/anonymous/api/wiki/9989d3d7-02c1-444e-92be-576b33d2f2be/page/3dc63f46-4a33-4e0b-98bf-4e55b74e556b/attachment/a19b9122-5940-4c89-ba3e-4b4fc25e2328/media/QRadar\_LEEF\_Format\_Guide.pdf).

Example full LEEF message (single line):
```
LEEF|1.0|AdditionSecurity|ASGW|1.0|1001|devTimeFormat=MMM dd yyyy HH:mm:ss 
devTime=Apr 01 2018 09:05:33 cat=SystemCharacteristics src=10.9.0.10 sev=3 
confidence=Medium title=- accountName2=myuser@example.com resourceType=Android 
application=com.example.app event2=2002 model=phone file=/dev/null number=42
```

### KVP (Key-Value Pair)

The key-value pair format is a simple representation that contains space-separated “key=value” pairs. Quotes may optionally surround the value.

Special characters are escaped as follows:

-   A “ (double quotes) character is escaped by a preceding backslash (i.e. \\”)

-   A = (equals) character is escaped by a preceding backslash (i.e. \\=)

-   A \\ (backslash) character is escaped by a preceding backslash (i.e. \\\\)

-   Non-printable ASCII characters are escaped by a leading \\x followed by two characters that represent the hexadecimal character value (i.e. \\x07)

Examples:
```
category=AppCategory data1=”some \\”custom\\” data”
eventId=123 filepath=”c:\\\\temp\\\\tmp27342.log”
date=2018-01-01T14:32:89 attribute=”key\\=value”
```

Example full KVP message (single line):

```
eventId=1001, title="-", eventSubId=2002, ts=2018-04-01T16:01:25Z, cat=SystemCharacteristics, 
recvIp=10.9.0.10, sev=3, conf=Medium, accountId2="myuser@example.com", systemType=Android, 
application="com.example.app", file="/dev/null", number=42
```


### CSV (Comma-Separated Values)

The CSV transform uses the standard comma-separated value format. Values are separated by commas, and complex values are enclosed in quotes; embedded quotes are escaped by a second quote.

Example:

`Firstvalue,”second value”,”third and “”final”” value”`

The CSV transform will optionally output a standard CSV “header” that indicates the field names. Each message only includes the values, in the matching order of the header field names.

<div class="alert alert-info">There are an optional number of extension value fields for a CSV message; see the discussion under <i>“Special Formatting Considerations”</i>.</div>

<div class="alert alert=info">CSV transformation is very performant, and produces the smallest output data (i.e. bandwidth-efficient) since it doesn’t include the overhead of key names within each message.</div>

### Special Formatting Considerations

Certain fields in a CSV message produced by the Message Gateway have special meaning. The first field is a “Format” field, which is currently defined as “2”. 

<div class="alert alert-info">CSV Format 1 is deprecated and no longer supported as of Message Gateway version 1.4</div>

A message can also have an arbitrary number of extensible “Data” fields, which are value fields of the form “key=value”. To be able to process the correct number of fields, use the “DataCount” value as an indicator of the total number of message fields.

For example, a CSV header and messages may look like:

```
Format,EventId,DataCount,Data
2,123,0
2,234,1,”key1=val1”
2,345,2,”key1=val1”,”key2=val2”
```

In the example, first we have a header field. Then a message with `EventId=123` and no Data.  Next is a message with `EventId=234`, and the extensible Data field value of `key1=val1` which is separately parseable as `key1` = `val1`; the `DataCount` field value reflects 1 total data value. The final message has `EventId=345`, and two extensible data field values of `key1` = `val1` and `key2` = `val2` (with extra escaped quotations); the `DataCount` field value reflects the 2 total data values.

This behavior will be consistent for any CSV message produced by the Message Gateway that has a `Format` field value of “2”. Any modifications of this format will use a different `Format` field value, signaling to any data processors that the format expectations differ from what is documented here as `Format` type “2”.

Example full CSV message (single line):

```
2,10.0.0.10,-,"myuser@example.com",Android,cd0f156a6859ee2,-,"com.example.app",400,0,"-",2016-05-24T18:47:40Z,SystemCharacteristics,Moderate,Medium,3,"model=""phone""","file=""/dev/null""","number=42"
```

### JSON (JavaScript Object Notation)

JSON is an industry standard to express data in complex structures. The JSON format produced for each message involves a top-level JSON object (aka dictionary, hash table, or map) of keys and values; there is also an “observableData” key that contains a further JSON object of keys and values.

For example (NOTE: spacing included for readability, but not included in the actual output):

```JSON
{
	“date”:”2018-01-01”,
	“eventId”:123,
	“observableData”:{
		“file”:”/tmp/tmp34234.log”,
		“data”:”\\x03\\xfaD\\x91”
	}
}
```

<div class="alert alert-warning">JSON transformation is a processing intensive format; other transform formats should be considered if possible. The output is also large and potentially multi-line, making it incompatible with syslog outputs.</div>

Example full JSON message:

```JSON
{
   "id":1001,
   "testTitle":"-",
   "subId":2002,
   "timestamp":"2018-04-01T16:09:05Z",
   "category":"SystemCharacteristics",
   "recvIp":"10.9.0.10",
   "impact":"Moderate",
   "confidence":"Medium",
   "accountId2":"myuser@example.com",
   "systemType":"Android",
   "application":"com.example.app",
   "observableData":{
      "model":"phone",
      "file":"/dev/null",
      "number":42
   }
}
```

Message Output
==============

Once messages are transformed, they are subject to forwarding or storage; this is referred to as an output.

### File Output

Messages are appended to a local file. File output can be very useful in testing, development, or low-volume situations; however, it is considered unsuitable for high-volume message environments due to limited file I/O write bandwidth.

### Sinkhole Output

The sinkhole output will discard all messages after transformation. It is exclusively used for network load and transform performance testing.

### Console Output

**Requires Messaging Gateway version 1.4 or later**

The console output will display all messages on the system console.  It should be used only for testing and debugging.


### UDP Syslog Output

Messages can be forwarded to a network syslog server over UDP. The UDP syslog output is high performance due to the minimal network overhead of UDP traffic, but messages are subject to the potential loss in an unreliable network situation.

<div class="alert alert-info">Certain multi-line and large message transform formats, such as JSON, are not compatible with syslog output. The Message Gateway will issue a configuration error upon startup if you attempt an incompatible combination/configuration.</div>

### AWS S3 Batch Upload Output

The Message Gateway can aggregate incoming messages into a batch/single file and then upload it into an Amazon AWS S3 bucket. The max batch data size and batch timing intervals are configuration. All AWS S3 regions (standard or region specific) are supported.

Files are uploaded to the “root” directory of the S3 bucket, with a filename format of:
`(random UUID) . (counter) . (extension)`

-   **Random UUID** - a random value generated by the Message Gateway at startup, and consistent through the lifetime of that single application runtime/execution

-   **Counter** - an incremental counter that starts at 1 and increases for each S3 PUT performed by the gateway

-   **Extension** - a file extension appropriate to the selected transform; e.g. CSV transform uses “.csv”, JSON transform uses “.json”, etc.

For certain transforms, such as CSV, a header line may be included at the beginning of each uploaded file.

For data safety purposes, all batched messages are also stored in a temporary file on disk. When successfully uploaded, the temporary file is deleted. In the event of a communication error with AWS S3, an error message is logged along with the path to the temporary cache file, and the cache file is not deleted -- you can retrieve those unsent messages from the undeleted cache file at a later time.


Configuration (config.properties)
=================================

Proper configuration is essential to the operation of the Addition Security Messaging Gateway. The included `config.properties` file contains all available configuration options and inline brief documentation.

The `config.properties` file contains four sections: global configuration, data input configuration, data transformation configuration, and data output configuration. The file uses the common Java properties file format, which utilizes a simple `key=value` approach.

Global Configuration Section
----------------------------

The following configuration values are found in the global configuration section. These options affect the entirety of Message Gateway operation.

### Port

|  Property Name:     |port
|  ----------------- |-------------------------
|  Default Value:     |5000
|  Possible Values:   |Integer between 1-65535

The `port` property defines the network TCP port the Messaging Gateway listens on for HTTP requests. By default, the Messaging Gateway listens on TCP port 5000, which is compatible with Amazon AWS Elastic Beanstalk.

<div class="alert alert-info">Changing the port to a value below 1024 may require the Messaging Gateway to run with elevated privileges on certain systems (e.g. Linux) in order to properly bind to the privileged port.</div>

### Max_size

  Property Name:     |max\_size
  ----------------- |-----------------------------------
  Default Value:     |65536
  Possible Values:   |Integer between 1 - 2,147,483,647

The maximum size allowed for incoming POST requests. This configuration value is meant to safeguard the Message Gateway from large erroneous POST requests. 

<div class="alert alert-info">Large requests must be temporarily saved to disk or stored in memory, so excessively large values may degrade performance.</div>

Generally speaking, the Addition Security products will not send single requests of more than 64K (65536) in size; if necessary, the Addition Security products will divide messages into multiple smaller successive requests.

<div class="alert alert-danger">Inappropriately low values (under 8192) may prevent proper operation.</div>

### Definitions

  Property Name:   |  definitions
  -----------------| -------------------------------
  Default Value:    |(Empty)
  Possible Values:   |Path to `definitions.json` file

Message title information and other message meta-data is stored in a separate JSON file, typically named `definitions.json`. A definitions file can provide more message context information into the transformed message.

The Message Gateway application distribution file includes a basic `definitions.json` file for use; updated definitions files may be available for download via the Addition Security customer portal.


### Save_ip

  Property Name:    | save\_ip
  ----------------- |-------------------
  Default Value:     |true
  Possible Values:   |“true” or “false”

The Message Gateway will normally extract the client IP address (possibly via an X-Forwarded-For header) and include it with the message, for tracking the source IP address that sent the incoming message(s). In some circumstances (e.g. privacy regulations), it may be desirable to not record this information. When `save_ip=false`, the Messaging Gateway will record all source IP addresses as “0.0.0.0”.

### Ssl_keystore

  Property Name:    | ssl\_keystore
  ----------------- |-------------------
  Default Value:     |(Empty)
  Possible Values:   |Path to Java KeyStore (.jks) file containing server SSL certificate

**Requires Messaging Gateway version 1.4 or later**

The Message Gateway will normally operate in HTTP mode.  By providing the `ssl_keystore` and associated `ssl_keystore_password` values, the server will operate in HTTPS mode using the SSL certificated contained in the keystore file referenced by the `ssl_keystore` value.

<div class="alert alert-info">A keystore password is required when the keystore property value is set</div>

### Ssl_keystore_password

  Property Name:    | ssl\_keystore\_password
  ----------------- |------------------
  Default Value:     |(Empty)
  Possible Values:   |Password string for associated java key store used in `ssl_keystore`

**Requires Messaging Gateway version 1.4 or later**

The Message Gateway will normally operate in HTTP mode.  By providing the `ssl_keystore` and associated `ssl_keystore_password` values, the server will operate in HTTPS mode using the SSL certificated contained in the keystore file referenced by the `ssl_keystore` value.



Data Input Section
------------------

The configuration properties in the data input section affect the processing of incoming POST data, prior to message transformation.

### Input

  Property Name:    | input
  -----------------| -------------------------------------
  Default Value:    | **NONE; a value must be specified**
  Possible Values:   |“protobuf”

The expected input message format. The Message Gateway currently only supports the Addition Security Operational Intelligence protobuf format, indicated by a “protobuf” value.

### Input.coalesce_disable

  Property Name:     |input.coalesce\_disable
  ----------------- |-------------------------
  Default Value:     |false
  Possible Values:   |“true” or “false”

By default the Messaging Gateway will use de-duplication heuristics to remove immediately redundant messages in incoming data transmissions. You can disable this default behavior by setting the disable property to “true”.

### Input.limit_org

  Property Name:     |input.limit\_org
  ----------------- |------------------------------------------
  Default Value:     |(Empty)
  Possible Values:   |Hexadecimal org ID value (40 characters)

By default the Messaging Gateway will allow all incoming messages. Included in each message is an organization (“org”) ID that relates to the customer configuration used by the client. Your org ID is listed in the Addition Security customer portal, with your account info.

The Message Gateway can filter incoming messages to just your organization ID by copying the hexadecimal value listed in the AdditionSecurity customer portal as the property value. For example:

`input.limit_org=bb54cfac59e73d8dae01b84bd476bbadde7d8747`


Data Transformation Section
---------------------------

The data transformation section contains the desired message transformation format and any transformation-specific configuration options.

### Transform

  Property Name:    | transform
  -----------------| -------------------------------------
  Default Value:     |**NONE; a value must be specified**
  Possible Values:   |“cef”, “leef”, “kvp”, “csv”, “json”

The desired message transform to apply. Details of each particular transform format are specified in the ***“Message Transform/Output Formats”*** section of this document.

### Transform.systemId2_string

  Property Name:     |transform.systemId2\_string
  ----------------- |-----------------------------
  Default Value:     |false
  Possible Values:   |“true” or “false”

By default (`transform.systemId2_string=false`), the `systemId2` message value, which signifies an application-provided system identifier, is transformed into messages as hexadecimal characters. Set the property value to “true” if you wish the `systemId2` message value to be decoded as an ASCII string.

<div class="alert alert-info">It must be known in advance whether the endpoint is sending a string-based “systemId2” identifier or a binary-based “systemId2” identifier.</div>

### Transform.accountName2_string

  Property Name:     |transform.accountName2\_string
  ----------------- |--------------------------------
  Default Value:     |false
  Possible Values:   |“true” or “false”

By default (`transform.accountName2_string=false`), the `accountName2` message value, which signifies an application-provided user/account identifier, is transformed into messages as hexadecimal characters. Set the property value to “true” if you wish the `accountName2` message value to be decoded as an ASCII string.

<div class="alert alert-info">It must be known in advance whether the endpoint is sending a string-based “accountName2” identifier or a binary-based “accountName2” identifier.</div>

<div class="alert alert-info">The value provided by the Addition Security MobileAwareness AS_Register_Identity() API call is typical string-based, and thus this property should be set to “true”.</div>

### Transform.include_title

  Property Name:     |transform.include\_title
  ----------------- |--------------------------
  Default Value:     |false
  Possible Values:   |“true” or “false”

<div class="alert alert-info">This property is effective only if the “definitions” property is set and the “transform” is set to “cef”, “leef”, “json”, or “kvp”.</div>

Human-readable message title information is not included in transformed messages by default, for performance and bandwidth reasons. Normally the eventID and eventSubID values can be post-processed to the equivalent meanings (e.g. eventID “401” is always the “ApplicationTamperingDetected” title). It may be desired to include human-readable title and subtitle information in each transformed message; if so, set the property value to “true”.

<div class="alert alert-info">Performance and bandwidth may be affected to look up, include, and forward the extra title information for each message.</div>

### Transform.include_organizationId

  Property Name:     |transform.include\_organizationId
  ----------------- |--------------------------
  Default Value:     |false
  Possible Values:   |“true” or “false”

**Requires Messaging Gateway version 1.4 or later**

Include the reported organization ID in the transformed messages.  This feature is only useful when using a single gateway to receive data from multiple separate organizations (multi-tenancy) or otherwise track the organization ID/license info of the reporter.


Data Output Section
-------------------

The data output section contains information on where transformed messages are sent or stored.

### Output

  Property Name:     |output
  ----------------- |---------------------------------------
  Default Value:     |**NONE; a value must be specified**
  Possible Values:   |“udpsyslog”, “file”, “s3”, “sinkhole”, "console"

The desired message output. Details of each particular output is specified in the ***“Message Outputs”*** section of this document.

**"console" requires Messaging Gateway version 1.4 or later**

### Output.file.path

  Property Name:     |output.file.path
  ----------------- |------------------------------------------------------
  Default Value:     |**NONE; a value must be specified if “output=file”**
  Possible Values:   |An absolute filesystem path of an output file

**This property is effective only if “output=file”**

The filesystem path of the file to write/append messages to. The parent directory must exist prior to Message Gateway startup, and the parent directory must be suitably writable by the Message Gateway process to create/write the file.

<div class="alert alert-info">The Message Gateway does not currently “rotate” or otherwise cease appending to the single file while running</div>

### Output.syslog.host

  Property Name:     |output.syslog.host
  ----------------- |-----------------------------------------------------------
  Default Value:     |**NONE; a value must be specified if “output=udpsyslog”**
  Possible Values:   |DNS hostname or IP address of syslog host


**This property is effective only if “output=udpsyslog”**

The specified hostname or IP address of the remote host to send syslog messages to.

<div class="alert alert-info">The Message Gateway currently does a DNS lookup of a provided hostname at startup and caches it during the lifetime of the application. Subsequent DNS changes of a hostname are not effectual without a Message Gateway software restart to receive the updated DNS resolution information.</div>

### Output.syslog.port

  Property Name:     |output.syslog.port
  ----------------- |------------------------------
  Default Value:     |514
  Possible Values:   |An integer between 1 - 65535

**This property is effective only if “output=udpsyslog”**

The network port the remote syslog host is listening on. The traditional syslog port (514) is the default.

### Output.syslog.facility

  Property Name:     |output.syslog.facility
  ----------------- |---------------------------
  Default Value:     |14 (aka “Local0”)
  Possible Values:   |An integer between 1 - 24

**This property is effective only if “output=udpsyslog”**

The syslog facility to use when sending syslog messages; the syslog “Local0” facility is the default. See the syslog protocol RFC for defined values.

### Output.syslog.severity

  Property Name:     |output.syslog.severity
  ----------------- |--------------------------
  Default Value:     |6 (aka “Informational”)
  Possible Values:   |An integer between 1 - 8

**This property is effective only if “output=udpsyslog”**

The syslog severity to use when sending syslog messages; the syslog “Information” severity is the default. See the syslog protocol RFC for defined values.

### Output.syslog.bom

  Property Name:     |output.syslog.bom
  ----------------- |-------------------
  Default Value:     |false
  Possible Values:   |“true” or “false”

**This property is effective only if “output=udpsyslog”**

The syslog BOM (Byte Order Marker) is a special indicator optionally sent in syslog messages when messages are using non-ASCII data formats. The Addition Security Message Gateway uses ASCII-appropriate encoding for syslog messages, but some receiving syslog servers may expect or desire a BOM marker for UTF-8 decoding, etc.

Normally a BOM marker is not sent in syslog messages (`output.syslog.bom=false`, the default).  You should change the property to “true” if you experience your syslog receiver incorrectly parsing incoming messages.

### Output.s3.endpoint

  Property Name:     |output.s3.endpoint
  ----------------- |----------------------------------------------------
  Default Value:     |**NONE; a value must be specified if “output=s3”**
  Possible Values:   |AWS S3 hostname without the bucket

**This property is effective only if “output=s3”**

The S3 endpoint is particular the AWS region hosting the S3 bucket, or the global “standard” region.

Example values:
```
output.s3.endpoint=s3-us-west-2.amazonaws.com // for us-west-2 S3 region
output.s3.endpoint=s3.amazonaws.com // for standard/global S3 region
```

### Output.s3.bucket

  Property Name:     |output.s3.bucket
  ----------------- |----------------------------------------------------
  Default Value:     |**NONE; a value must be specified if “output=s3”**
  Possible Values:   |AWS S3 bucket name

**This property is effective only if “output=s3”**

The AWS S3 bucket name. The bucket name is combined with the `output.s3.endpoint` property value to construct the public hostname for the S3 bucket.

### Output.s3.access_key

  Property Name:     |output.s3.access\_key
  ----------------- |----------------------------------------------------
  Default Value:     |**NONE; a value must be specified if “output=s3”**
  Possible Values:   |AWS IAM access key

**This property is effective only if “output=s3”**

An AWS IAM access key with sufficient S3 write permissions.

<div class="alert alert-info">A value must be explicitly specified; the Message Gateway does not currently retrieve implicit IAM credentials when deployed within an AWS environment</div>

### Output.s3.secret_key

  Property Name:     |output.s3.secret\_key
  ----------------- |----------------------------------------------------
  Default Value:     |**NONE; a value must be specified if “output=s3”**
  Possible Values:   |AWS IAM secret key

**This property is effective only if “output=s3”**

An AWS IAM secret key with sufficient S3 write permissions.

<div class="alert alert-info">A value must be explicitly specified; the Message Gateway does not currently retrieve implicit IAM credentials when deployed within an AWS environment</div>

### Output.s3.memory_max

  Property Name:     |output.s3.memory\_max
  ----------------- |-------------------------------------------
  Default Value:     |8388608
  Possible Values:   |Integer (bytes) between 1 - 2,147,483,647

**This property is effective only if “output=s3”**

The maximum amount of memory (in bytes) to use for batch aggregation of messages before an S3 PUT is forced/performed. The default is 8MB. Note the memory consumption relates to raw data size, and not the total Java/JVM overhead -- consider the specified value to be approximate.

<div class="alert alert-warning">Inappropriately low values (under 1MB) may affect performance</div>

### Output.s3.interval

  Property Name:     |output.s3.interval
  ----------------- |------------------------------------
  Default Value:     |300
  Possible Values:   |Integer (seconds) between 1 - 3600

**This property is effective only if “output=s3”**

The time interval used for S3 batch flushing, which defaults to 5 minutes. Time-based intervals are used to ensure small amounts of batch queued messages are flushed to S3 if there are not enough messages present to hit the `output.s3.memory_max` flush trigger.

<div class="alert alert-info">The logic will ensure at least one message flush within the given interval range; given certain timing circumstances (i.e. a flush occurs within the first second of an interval), it may be nearly two intervals (the remainder of the first interval, which experienced a flush, and a subsequent interval, which didn’t experience a flush) before a flush is performed.</div>

Running the Gateway Application
===============================

First confirm your Java version by running the following command:

```
java -version
```

Your Java version should list “1.8.x” (a.k.a. Java 8). If your Java version indicates “1.7.x” or “1.6.x”, you will need to update to Java 8.

The gateway application is executed/started by a standard Java command:
```
java -jar asgw.jar path/to/config.properties
```

The `java -jar asgw.jar` executes the Java VM and launches the application contained in the Addition Security provided `asgw.jar` file. The config.properties path is provided as a parameter to the command/gateway application.

Typically, the `asgw.jar` and `config.properties` files (and optionally the `definitions.json` file) will be contained in the same folder/directory. Execution is then simply executing the command from that folder/directory:

```
java -jar asgw.jar config.properties
```

The gateway will print information to the console/standard out. An initial startup configuration will be displayed, for example:

```
ASGateway 1.1
- Upload/temp dir: /tmp
- Definitions: additionsecurity.com 20160401
- Input: AddSec Protobuf
- Transform: KVP; includeTimestamp=true; includeTitle=true
- Output: File; file=/tmp/cti.log
Ready to receive requests on port 5000
```

The configuration of the input, transform, and output is displayed, along with the loaded definitions information. The “Ready to receive requests” final line indicates the gateway is properly initialized and running.

Example Configurations
======================

### CEF & Syslog

An example configuration that will transform incoming messages on HTTP port 8080 into the CEF format, then forward them via syslog to a host at the IP address “10.0.1.1” on syslog port 514. Definitions will be loaded and titles will be enabled in the CEF messages.

```
port=8080
definitions=definitions.json
input=protobuf
transform=cef
transform.include_title=true
output=udpsyslog
output.syslog.host=10.0.1.1
output.syslog.port=514
```

### KVP & S3

An example configuration that will transform incoming messages on HTTP port 5000 into the KVP format, then batch upload the messages to AWS S3 bucket named “addsec-messages” in the US Standard S3 region. Titles will not be included.

```
port=5000
input=protobuf
transform=kvp
output=s3
output.s3.endpoint=s3.amazonaws.com
output.s3.bucket=addsec-messages
output.s3.access_key=AKI...AQ
output.s3.secret_key=QQ6t...QA
```

Common Field Mappings
=====================

The following chart indicates the key/field names used by each transform output.

<div class="alert alert-info">Addition Security Operational Intelligence messages have extensible data sections that can use various key names; the list below is for the typical/common keys only.</div>

  ------------------------------------------------------------------------------------------------------------
  **Key**                  |**CEF**                | **LEEF**       |**KVP**       |**CSV**       |**JSON**
  ------------------------ |----------------------- |-------------- |------------- |------------- |----------------
  ***eventId***            |CEF header              |LEEF header    |eventId       |EventId       |id
  ***eventSubId***         |cnX/cnXLabel=eventSubId              |event2         |eventSubId    |EventSubId    |subId                                             
  ***category***           |-                       |cat            |cat           |Category      |category
  ***title***              |CEF header              |title          |title         |EventTitle    |title
  ***severity***           |CEF header              |sev            |sev           |Impact        |impact
  ***confidence***         |-                       |confidence     |conf          |Confidence    |confidence
  ***timestamp***          |start                   |devTime        |ts            |Timestamp     |timestamp
  ***recvip***             |dvc                     |src            |recvIp        |RecvIp        |recvIp
  ***accountId***          |suser                   |accountName    |accountId     |AccountId     |accountId
  ***accountId2 \[2\]***   |csX/csXLabel=suser2                    |accountName2   |accountId2    |AccountId2    |accountId2                                             
  ***systemId \[1\]***     |deviceExternalId        |resource       |systemId      |SystemId      |systemId
  ***systemId2***          |csX/csXLabel=deviceExternalId2                    |resource2      |systemId2     |SystemId2     |systemId2                                                             
  ***systemType***         |csX/csXLabel=deviceType                    |resourceType   |systemType    |SystemType    |systemType                                                                          
  ***applicationId***      |deviceProcessName       |application    |application   |Application   |application
  ***organizationId\[3\]***   | cs1/cs1Label=org | org | org | Organization\[4\] | org 
  ***Extensible data***    |csX/cnX with labels   |(varies)       |(varies)      |Data          |observableData

\[1\] The “systemId” value is the required (device) identifier provided to the `AS_Initialize()` function in the Addition Security MobileAwareness SDK, which is in hexadecimal format; the value is intended to be unique across systems, but may not necessarily be the same value for applications on the same system. See the MobileAwareness SDK documentation for details.

\[2\] The “accountId2” value is the optional value provided to the `AS_Register_Identity()` function in the Addition Security MobileAwareness SDK, which is typically in string format

\[3\] The "organizationID" value is only reported when `transform.include_organizationId=true`

\[4\] The "Organization" field is only present when `transform.include_organizationId=true`
