[![Build Status](https://travis-ci.org/addario-org/currencycloud-java-client.svg?branch=master)](https://travis-ci.org/github/addario-org/currencycloud-java-client)
[![Github Package](https://img.shields.io/badge/github%20package-v3.6.1--ea-brightgreen)](https://github.com/addario-org/currencycloud-java-client/packages/329443)
[![Maven Package](https://img.shields.io/badge/maven%20package-v3.6.1--ea-brightgreen)](https://search.maven.org/artifact/io.github.addario-org/currencycloud-java-client/3.6.1-ea/jar)
[![Issues](https://img.shields.io/github/issues/addario-org/currencycloud-java-client)](https://github.com/addario-org/currencycloud-java-client/issues)
[![License](https://img.shields.io/github/license/addario-org/currencycloud-java-client)](https://github.com/addario-org/currencycloud-java-client/blob/master/LICENSE.md)
[![Last Commit](https://img.shields.io/github/last-commit/addario-org/currencycloud-java-client)](https://github.com/addario-org/currencycloud-java-client/commits/master)


# Currencycloud API v2 Java client

## Version: 3.6.1-ea (FORK)
This is a **FORK** from the official [Java SDK for the Currencycloud API.][original] While it tries to keep in sync with the upstream version, it does also include enhancements and modifications ranging from bugfixes to usability features.

Additional documentation for each API endpoint can be found at [developer.currencycloud.com][docs].

If you have any queries please contact their development team at development@currencycloud.com Please quote your login Id in any correspondence as this allows us to locate your account and give you the support you need.

## Prerequisites
### 1. Maven (optional, but highly recommended)
CurrencyCloud-Java is a Maven project. We highly recommend using [Apache Maven][maven] 3 (or a compatible build tool like Gradle) 
to build your project. While using Maven is not strictly required 
it will simplify building the project and handling dependencies.

### 2. Oracle JDK 8 or equivalent JDK
CurrencyCloud-Java requires at least a Java version 8 compatible JDK.

### 3. A valid sandbox login id and api key on the Currencycloud sandbox API environment.
You can register for a demo API key at [developer.currencycloud.com][developer].

While we expose certain routes on the sandbox API without the requirement for authentication, we rate-limit these requests significantly to prevent abuse. Rate-limiting on authenticated requests is less restrictive.

## Installing the Currencycloud SDK
### 1. Using Maven
To use the Currencycloud SDK in a Maven project, add the following dependency to your project's `pom.xml`:
```xml
<dependency>
    <groupId>io.github.addario-org</groupId>
    <artifactId>currencycloud-java-client</artifactId>
    <version>3.6.1-ea</version>
</dependency>
```

### 2. Manually downloading the jars
Download the Currencycloud SDK jar:
1. Open https://github.com/addario-org/currencycloud-java-client/packages
2. Navigate to the version of currencycloud-java-client that you wish to use
3. Download the currencycloud-java-client-3.6.1-ea.jar

Get the list of all dependencies:
```Shell
mvn dependency:list -DincludeScope=runtime
```
As of version 2.0.0, this returns the following list:
```
cglib:cglib:jar:3.3.0:compile
ch.qos.logback:logback-classic:jar:1.2.3:compile
ch.qos.logback:logback-core:jar:1.2.3:compile
com.fasterxml.jackson.core:jackson-annotations:jar:2.11.1:compile
com.fasterxml.jackson.core:jackson-core:jar:2.11.1:compile
com.fasterxml.jackson.core:jackson-databind:jar:2.9.1:compile
com.fasterxml.jackson.dataformat:jackson-dataformat-yaml:jar:2.11.1:compile
com.github.mmazi:rescu:jar:2.0.2:compile
com.google.code.findbugs:jsr305:jar:3.0.2:compile
commons-codec:commons-codec:jar:1.3:compile
javax.ws.rs:javax.ws.rs-api:jar:2.1.1:compile
javax.ws.rs:jsr311-api:jar:1.1.1:compile
oauth.signpost:signpost-core:jar:1.2.1.2:compile
org.ow2.asm:asm:jar:8.0.1:compile
org.slf4j:slf4j-api:jar:1.7.30:compile
```
You will need to find each of these dependencies and download it from the [Sonatype Nexus][sonatype] as described above.

Finally, include all downloaded jars in your project's classpath.

# Usage
An example in Java 8 (Java 8 syntax is only used to format the output):
```Java
// Create API proxy; specifiy the environment to connect to and your API credentials.
CurrencyCloudClient currencyCloud = new CurrencyCloudClient(
    CurrencyCloudClient.Environment.demo, "<your login id>", "<your API key>"
);

// Make API calls
List<Currency> currencies = currencyCloud.getCurrencies();
System.out.println("Supported currencies: " + currencies.stream().map(Currency::getCode).collect(Collectors.joining(", ")));

List<Balance> balances = currencyCloud.findBalances(null, null, null, null).getBalances();
System.out.println("Balances: " + balances.stream()
        .map((b) -> String.format("%s %s", b.getCurrency(), b.getAmount()))
        .collect(Collectors.joining(", ")));

// End session
currencyCloud.endSession();
```
For a better example, see
[CurrencyCloudCookbook.java](/src/test/java/com/currencycloud/examples/CurrencyCloudCookbook.java), which is an implementation of [the Cookbook](https://connect.currencycloud.com/documentation/getting-started/cookbook) from the documentation.

## Common Misconceptions, Anti-patterns and Suggestions
1. Avoid creating one client per request
2. Sessions typically have a timeout of several tens of minutes; this is to allow customers to reuse existing sessions for as long as possible
3. Write your application so that it establishes a single instance of the CurrencyCloudClient class on startup and shares this between threads, keeping it alive until you shut the application down. This will translate into fewer requests on your part and less server load on ours
4. Requests over the internet will fail on occasion for seemingly no apparent reason, and the SDK includes a comprehensive set of [error handling capabilities](#errors) to help troubleshoot those situations. Sometimes however, the best strategy is simply to retry. This is the case particularly with transient errors like **HTTP 429 - Too Many Requests** but wrapping calls in for/while loops is discouraged as in some extreme cases this may trigger our anti-DoS defences.  As of version 1.2.3 we have introduced an [Exponential Backoff with Jitter][ebwj] retry feature which we recommend you use to safely handle retries. Please see CurrencyCloudCookbook.java and BackOffTest.java for examples
5. API calls can throw exceptions. Try/Catch blocks are your friend. Use them generously

## On Behalf Of
If you want to make calls on behalf of another user (e.g. someone who has a sub-account with you), you can execute certain commands 'on behalf of' the user's contact id. Here is an example:
```Java
currencyCloud.onBehalfOfDo("c6ece846-6df1-461d-acaa-b42a6aa74045", new Runnable() {
    public void run() {
        currencyCloud.createBeneficiary(...);
        currencyCloud.createConversion(...);
        currencyCloud.createPayment(...);
    }
});
```
Or in Java 8 and above:
```Java
currencyCloud.onBehalfOfDo("c6ece846-6df1-461d-acaa-b42a6aa74045", () -> {
    currencyCloud.createBeneficiary(...);
    currencyCloud.createConversion(...);
    currencyCloud.createPayment(...);
});
```
Each of the above transactions will be executed in scope of the limits for that contact and linked to that contact. Note that the real user who executed the transaction will also be stored.

## Concurrency and Sessions 
The SDK is thread-safe, and you should share one session across all your worker threads to avoid reaching the limit on authentication requests.

## Errors
When an error occurs in the API, the library is designed to provide as much information as possible. A `CurrencyCloudException` will then be thrown which contains useful information you can access via its methods (please consult the javadoc for more information).

When the exception is logged, it will provide information such as the following:
```yaml
BadRequestException
---
platform: Java 1.8.0_131 (Oracle Corporation)
request:
  parameters:
    login_id: non-existent-login-id
    api_key: ef0fd50fca1fb14c1fab3a8436b9ecb57528f0
  verb: post
  url: https://devapi.currencycloud.com/v2/authenticate/api
response:
  status_code: 400
  date: Wed, 29 Apr 2017 22:46:53 GMT
  request_id: 2775253392756800903
errors:
- field: api_key
  code: api_key_length_is_invalid
  message: api_key should be 64 character(s) long
  params:
    length: 64
```
This is split into 5 sections:

1. Error Type: In this case `BadRequestException` represents an HTTP 400 error
2. Platform: The Java implementation that was used in the client
3. Request: Details about the HTTP request that was made, e.g. the POST parameters
4. Response: Details about the HTTP response that was returned, e.g. HTTP status code
5. Errors: A list of errors that provide additional information

The final section contains valuable information:

- Field: The parameter that the error is linked to
- Code: A code representing this error
- Message: A human readable message that explains the error
- Params: A map that contains dynamic parts of the error message for building custom error messages

When troubleshooting API calls with Currencycloud support, including the full
error in any correspondence will be **very** helpful.

## Logging
The SDK uses [slf4j](slf4j) for logging, wich means you are free to use any of the popular logging providers supported by slf4j in your project (eg. log4j, Logback, or Java Logging), but you must add your chosen logging provider to your project's dependencies yourself. We recommend [Logback][logback]:
```xml
<dependency>
    <groupId>ch.qos.logback</groupId>
    <artifactId>logback-classic</artifactId>
    <version>1.1.3</version>
    <optional>true</optional>
</dependency>
```

# Development

## Building Currencycloud SDK from sources
To build the project from sources, you will need git and [Maven 3][maven].

In a shell, do the following:
```Shell
devuser@localhost ~ $ git clone https://github.com/CurrencyCloud/currencycloud-java.git     
devuser@localhost ~ $ cd currencycloud-java
devuser@localhost currencycloud-java $ mvn clean install
```

## Testing

Test cases can be run with `mvn test`

## Dependencies
```
co.freeside:betamax:jar:1.1.2:test
commons-lang:commons-lang:jar:2.4:test
commons-logging:commons-logging:jar:1.1.1:test
javax.servlet:servlet-api:jar:2.5:test
junit:junit:jar:4.12:test
org.apache.httpcomponents:httpclient:jar:4.2.1:test
org.apache.httpcomponents:httpcore:jar:4.2.1:test
org.codehaus.groovy:groovy-all:jar:2.4.17:test
org.eclipse.jetty:jetty-continuation:jar:7.3.1.v20110307:test
org.eclipse.jetty:jetty-http:jar:7.3.1.v20110307:test
org.eclipse.jetty:jetty-io:jar:7.3.1.v20110307:test
org.eclipse.jetty:jetty-server:jar:7.3.1.v20110307:test
org.eclipse.jetty:jetty-util:jar:7.3.1.v20110307:test
org.hamcrest:hamcrest-core:jar:1.3:test
org.hamcrest:hamcrest-junit:jar:2.0.0.0:test
org.hamcrest:java-hamcrest:jar:2.0.0.0:test
org.yaml:snakeyaml:jar:1.26:test
```

## Contributing
**We welcome pull requests from everyone!** Please see [CONTRIBUTING][contr]

Our sincere thanks for [helping us][hof] create the best API for moving money anywhere around the world!

## Versioning
This project uses [semantic versioning][semver]. You can safely
express a dependency on a major version and expect all minor and patch versions to be backwards compatible.

## Deprecation Policy
Technology evolves quickly and we are always looking for better ways to serve our customers. From time to time we need to make room for innovation by removing sections of code that are no longer necessary. We understand this can be disruptive and consequently we have designed a Deprecation Policy that protects our customers' investment and that allows us to take advantage of modern tools, frameworks and practices in developing software.

Deprecation means that we discourage the use of a feature, design or practice because it has been superseded or is no longer considered efficient or safe but instead of removing it immediately, we mark it as **@Deprecated** to provide backwards compatibility and time for you to update your projects. While the deprecated feature remains in the SDK for a period of time, we advise that you replace it with the recommended alternative which is explained in the relevant section of the code.

We remove deprecated features after **three months** from the time of announcement.

The security of our customers' assets is of paramount importance to us and sometimes we have to deprecate features because they may pose a security threat or because new, more secure, ways are available. On such occasions we reserve the right to set a different deprecation period which may range from **immediate removal** to the standard **three months**. 

Once a feature has been marked as deprecated, we no longer develop the code or implement bug fixes. We only do security fixes.

### List of features being deprecated
```
(No features are currently being deprecated)
```

# Support
We actively support the latest version of the SDK. We support the immediate previous version on best-efforts basis. All other versions are no longer supported nor maintained.

# Release History
* [3.6.1-ea] - Add create and get/set methods to improve SDK usability
* [3.6.1] - Add conversion date preference parameter to conversion/create and rates/detailed
* [3.5.1] - Add top-up margin balance endpoint
* [3.4.4] - Add funding accounts endpoint
* [3.3.1] - Add payment level fees endpoints
* [3.2.5] - Fixes /payments/authorise parameter name
* [3.2.2] - Add Reference Bank Details and Payment Delivery Date endpoints, remove deprecated IBAN and VAN endpoints, remove deprecated CurrencyCloudClient.createSettlement() method, update dependencies and add support for JDK 12
* [2.2.1] - Remove erroneous On Behalf Of parameter from conversions api endpoints. Adds beneficiary_external_reference to beneficiary create, update and find endpoints
* [2.1.3] - Add Accounts Payment Charges Settings endpoints. Adds Charge Type parameter to Payments Endpoints
* [2.0.0] - Remove deprecated methods, update dependencies and copyright
* [1.8.1] - Add Payment Confirmation and Sender Details
* [1.7.4] - Add Reporting paths and operations, remove json-smart dependency, change toString implementation, add missing Currency fields, update IBAN and VirtualAccounts, and deprecate obsolete IBAN and VirtualAccounts methods
* [1.6.1] - Add support for JDK 11
* [1.6.0] - Update Payment Purpose Code, add Payment tests, fix Javadoc warnings and update Maven plugins
* [1.5.1] - Add Payment Authorisation and Payment Purpose Code
* [1.4.4] - Add Conversion Quote Cancel, Conversion Cancel, Conversion Date Change Quote, Conversion Date Change, Conversion Date Change History, Conversion Split Preview, Conversion Split, Conversion Split History and Conversion Profit and Loss
* [1.3.0] - Add PaymentSubmission class and fix retrievePaymentSubmission
* [1.2.3] - Add VANs, add exponential backoff-and-retry, change toString in Currencycloud model classes to return RFC4627 compliant JSON, refactor ThreadSupport to private inner class and improve debug logging
* [1.0.3] - Update com.github.mmazi.rescu to latest version, desupport Java 7, add support for Java 9 and Java 10, fix bug in Beneficiary Date of Birth (#55), introduce builder pattern for *find* query parameters, deprecate tight-coupled methods and add production logging settings filtering out token and key  
* [0.9.1] - Add Transfers and IBANs, add missing API paths and operations (#42), update dependencies to newer versions, bug fixes (including #32 and #38), and other minor changes
* [0.7.8] - Address a concurrency issue discovered in the onBehalfOf functionality (#48) 

# Copyright
Copyright  &copy; 2015-2020 Currencycloud. See [LICENSE][license] for details.
Copyright &copy; modifications 2020, Ed Addario

[maven]:     https://maven.apache.org/index.html
[nexus]:     http://www.sonatype.org/nexus/
[slf4j]:     http://www.slf4j.org/
[logback]:   http://logback.qos.ch/
[rescu]:     https://github.com/mmazi/rescu
[jackson]:   https://github.com/FasterXML/jackson
[original]:  https://github.com/CurrencyCloud/currencycloud-java
[docs]:      https://connect.currencycloud.com/documentation/getting-started/introduction
[developer]: https://developer.currencycloud.com
[travis]:    https://travis-ci.org/CurrencyCloud/currencycloud-java
[semver]:    http://semver.org/
[sonatype]:  https://oss.sonatype.org/
[ebwj]:      https://aws.amazon.com/blogs/architecture/exponential-backoff-and-jitter/
[license]:   LICENSE.md
[contr]:     CONTRIBUTING.md
[hof]:       HALL_OF_FAME.md
