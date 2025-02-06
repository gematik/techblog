---
layout: post
title: "The 3 levels of Vau (version 1) data types explored in Kotlin"
date:   2024-12-20 16:00:00 +0200
author: Stephan Schröder
categories: tech
tags: Kotlin VAU
excerpt: "<br/>Presenting the Vau (version 1) data types in Kotlin<br/><br/>"
---

The vau encryption protocol en-/decrypts http requests and responses and can be modeled in three layers.
- level 1: just the raw data of the request or response to be wrapped/transmitted
- level 2: we add identifying and cryptographic information used match a response to it's request and to tell the backend with key to use to symmetrically encrypt the response
- level 3: the encrypted version of level 2

Let's look at how we can model this Kotlin.

## Vau Requests

The complete code for the vau requests data types can be found [here](https://github.com/simon-void/vauk/blob/main/src/main/kotlin/de/gmx/simonvoid/vau/request.kt).
(This isn't a gematik project and the code comes without warranty or support.)

### Level 1 Vau Request

This is a serialized example of a level 1 vau request: 
```text
POST /Task/$create HTTP/1.1
content-type: application/fhir+xml
authorization: Bearer eyJhbGciOiJCUDI1NlIxIiwia2lkIjoicHVrX2lkcF9zaWciLCJ0eXAiOiJhdCtKV1QifQ.eyJhdXRoX3RpbWUiOjE2OTMzOTIwNDQsInNjb3BlIjoiZS1yZXplcHQgb3BlbmlkIiwiY2xpZW50X2lkIjoiZ2VtYXRpa1Rlc3RQcyIsImdpdmVuX25hbWUiOm51bGwsImZhbWlseV9uYW1lIjpudWxsLCJkaXNwbGF5X25hbWUiOm51bGwsIm9yZ2FuaXphdGlvbk5hbWUiOiJBcnp0cHJheGlzIERyLiBtZWQuIEfDvG5kw7xsYSBHdW50aGVyIFRFU1QtT05MWSIsInByb2Zlc3Npb25PSUQiOiIxLjIuMjc2LjAuNzYuNC41MCIsImlkTnVtbWVyIjoiMS0yLVJFWkVQVE9SLUFSWlQtMDEiLCJhenAiOiJnZW1hdGlrVGVzdFBzIiwiYWNyIjoiZ2VtYXRpay1laGVhbHRoLWxvYS1oaWdoIiwiYW1yIjpbIm1mYSIsInNjIiwicGluIl0sImF1ZCI6Imh0dHBzOi8vZXJwLXJlZi56ZW50cmFsLmVycC5zcGxpdGRucy50aS1kaWVuc3RlLmRlLyIsInN1YiI6ImM5MTQyMzMyNjhhYWU0NjY3Y2E1MTE5ZGE2YzExMTVkYWNhYWQyMWI1MDgyZTY3NDQ0ZmFmNjBjNjNiYzM3MzQiLCJpc3MiOiJodHRwczovL2lkcC1yZWYuYXBwLnRpLWRpZW5zdGUuZGUiLCJpYXQiOjE2OTMzOTIwNDQsImV4cCI6MTY5MzM5MjM0NCwianRpIjoiYzhhMTU4YjItZDQ2ZC00NDJjLWE0MjMtODFmMWVkOWZkODYxIn0.c6tcPJagbL0hVWGn1YHa6FdsvrXugF4JBEaG4vomYT-FC4o_LqYRAlE-F0_KWtbeU8I78JNm31I182nko806Qg
user-agent: PostmanRuntime/7.29.3
postman-token: 0dad21d7-bb31-49e4-820d-1a1dd6dd193e
accept-encoding: gzip, deflate, br
connection: keep-alive
content-length: 270

<Parameters xmlns="http://hl7.org/fhir">
<parameter>
<name value="workflowType"/>
<valueCoding>
<system value="https://gematik.de/fhir/erp/CodeSystem/GEM_ERP_CS_FlowType"/>
<code value="163"/>
</valueCoding>
</parameter>
</Parameters>
```
In it's first line `POST /Task/$create HTTP/1.1` we see the http method, path and http version used of the original request.
The next conceptual block consists of all the headers of the original request (excluding the host header).
Next comes an empty line and then the bytes of the original request are appended. Within the context of eRezept,
the body is actually always text (more specifically json or xml) encoded in utf8, but Vau encoding isn't concerned about its content,
so treating the body as bytes is more correct when creating Vau data types.

Another curiosity of Vau in the context of eRezept is that while the newline characters used in Vau encoding is "\r\n"
while the xml or json send as a payload seems to generally use "\n".

Ok, let's put all this information into a data type:

```kotlin
class L1VauReqEnvelopeAkaInnerVau(
    val method: HttpMethod,
    val path: String,
    val httpVersion: HttpVersion,
    val headers: VauHeaders,
    val body: BodyBytes,
)
```

with `HttpMethod`, `HttpVersion`, `VauHeaders` and `BodyBytes` adding additional type safety around `String`, `Map` and `ByteArray` data types.

```kotlin
enum class HttpMethod {
    GET, POST, DELETE; // the only 3 http methods used in eRezept context
    ...
}

enum class HttpVersion(val value: String) {
    HTTP_1_1("HTTP/1.1"); // the only http version currently used in eRezept context
    ...
}

// Headers are represented/required as Map<String, String> or as Map<String, List<String>>
@JvmInline
value class VauHeaders private constructor(private val map: Map<String, List<String>>) {
    val contentType: String? get() = ...
    val jwt: String? get() = ...
    val contentLength: Int? get() = ...
    val size: Int get() = map.size

    fun asMapWithListOfValues(): Map<String, List<String>> = map
    fun asMapWithConcatenatedValues(splitStrategy: HeaderValueConcatenationStrategy = defaultValueConcatenationStrategy): Map<String, String> = ...

    companion object {
        fun fromConcatenatedHeaderValues(map: Map<String, String>): VauHeaders = ...
        fun fromSeparateHeaderValues(map: Map<String, List<String>>): VauHeaders = VauHeaders(map)
    }
}

@JvmInline
value class BodyBytes(val bytes: ByteArray) {
    // it could also makes sends to interpret the bytes as utf-8 text or output the bytes as base64
    override fun toString(): String = "BodyBytes(nrOfBytes=${bytes.size})"

    companion object {
        // since the body is often empty let's initialise a single instance for that case
        val EMPTY = BodyBytes(ByteArray(0))
    }
}
```

### Level 2 Vau Request

Unfortunately a Level 2 Vau Request, like one from Level 1, is often simply referred to "InnerVau".
The easiest way, to discern which level it is from its serialised form is to check if the first token is a number (level 2) or an http method (level 1).
It has the following form:

```text
{vauVersion} {accessToken} {requestId} {aesKey} {l1vauRequest}
```

- vauVersion: since we're specifically looking at version 1 of Vau, the first token will always be "1"
- accessToken: this is the jwt token from the *Authorization* header of the original request (so the value of the header minus the "Bearer " prefix)
- requestId: is a random lowercase hex value intended to match a response to its request (if necessary). The length isn't strictly specified, we use 32 characters.
- aesKey: is a generated, random 32-character long lowercase hex value of the AES-key used to encrypt the response symmetrically
- l1vauRequest: is exactly as specified above (and as showcased by the given example)

So as data type(s) this looks like this:

```kotlin
class L2VauReqEnvelopeAkaInnerVau(
    val version: Int = 1,
    val accessToken: AccessCode,
    val requestId: RequestId,
    val aesKey: AesKey,
    val l1InnerVau: L1VauReqEnvelopeAkaInnerVau,
)
```

with helper classes, that check if the expected format is given:

```kotlin
@JvmInline
value class AccessCode(val value: String) {
    init {
        require(value.isNotBlank()) { "accessCode must not be blank" }
    }
}

private val hexRegex: Regex = "[0-9a-f]+".toRegex() // by spec only lowercase is allowed
private fun String.assertIsHexEncoded(paramName: String) {
    require(this.matches(hexRegex)) { "param $paramName is not lowercase hex encoded: $this" }
}

@JvmInline
value class RequestId(val hexValue: String) {
    init {
        hexValue.assertIsHexEncoded("requestId")
    }
}

@JvmInline
value class AesKey(val hexValue: String) {
    init {
        hexValue.assertIsHexEncoded("aesKey")
        require(hexValue.length == 32) { "Invalid aes key length: ${hexValue.length}, expected 32" }
    }
    // the cryptographic operation will (most likely) request the aesKey as a byte array 
    fun asBytes(): ByteArray = Hex.decodeHex(hexValue)
}
```

### Level 3 Vau Request

The third level of a Vau request is in its encrypted form (asymmetrically encrypted with the public key of the receiver).
But even here there's still a certain structure to be found. The first byte contains the version of Vau used, so in our context that's always a 1.
Then come the x- and y-coordinate used to encrypt this request each taking 32 bytes. Then comes the cipher which starts off with a 12 byte initialisation vector
and everything afterwards the the tagged cipher text. 

```kotlin
class L3VauReqEnvelopeAkaOuterVau(
    val bytes: ByteArray,
) {
    val version: VauVersion = VauVersion(bytes[0])

    val xCoordinate: XCoordinate get() = XCoordinate(bytes.copyOfRange(1, 33))
    val yCoordinate: YCoordinate get() = YCoordinate(bytes.copyOfRange(33, 65))
    val cipher: Cipher
        get() = Cipher(
            bytes.copyOfRange(
                65,
                bytes.size
            )
        )

    override fun toString(): String = "L3VauReqEnvelopeAkaOuterVau(nrBytes=${bytes.size})"
}
```

with helper classes:

```kotlin
@JvmInline
value class VauVersion(val value: Byte) {
    init {
        require(value.toInt() == 1) { "Invalid version: $value, expected 1" }
    }
}

@JvmInline
value class XCoordinate(val bytes: ByteArray) {
    init {
        require(bytes.size == 32) { "Invalid byte array size: ${bytes.size}, expected 32" }
    }
}

@JvmInline
value class YCoordinate(val bytes: ByteArray) {
    init {
        require(bytes.size == 32) { "Invalid byte array size: ${bytes.size}, expected 32" }
    }
}

@JvmInline
value class Cipher(val bytes: ByteArray) {
    override fun toString(): String = "Cipher(nrOfBytes=${bytes.size})"

    val iV: InitialisationVector get() = InitialisationVector(bytes.copyOf(12))
    // tagged cipher text is returned as ByteArray because that's what the decryption algorithm expects
    val taggedCipherText: ByteArray get() = bytes.copyOfRange(12, bytes.size)
}

@JvmInline
value class InitialisationVector(val bytes: ByteArray) {
    init {
        require(bytes.size == 12) { "Invalid byte array size: ${bytes.size}, expected 12" }
    }
}
```

### Level 1 Vau Response

A level 1 Vau Response has a structure very similar to that of its request counterpart.
Here's an example:

```text
HTTP/1.1 201 Created
Content-Type: application/fhir+xml;charset=utf-8
Content-Length: 1286

<?xml version="1.0" encoding="utf-8"?>
<Task xmlns="http://hl7.org/fhir"><id value="160.000.226.640.861.41"/><meta><profile value="https://gematik.de/fhir/erp/StructureDefinition/GEM_ERP_PR_Task|1.2"/></meta><extension url="https://gematik.de/fhir/erp/StructureDefinition/GEM_ERP_EX_PrescriptionType"><valueCoding><system value="https://gematik.de/fhir/erp/CodeSystem/GEM_ERP_CS_FlowType"/><code value="160"/><display value="Muster 16 (Apothekenpflichtige Arzneimittel)"/></valueCoding></extension><identifier><use value="official"/><system value="https://gematik.de/fhir/erp/NamingSystem/GEM_ERP_NS_PrescriptionId"/><value value="160.000.226.640.861.41"/></identifier><identifier><use value="official"/><system value="https://gematik.de/fhir/erp/NamingSystem/GEM_ERP_NS_AccessCode"/><value value="6d3aff43cb7f1db78e78dde37e6ae8db6f94e6c4291eb2b7bc8876a645ee3475"/></identifier><status value="draft"/><intent value="order"/><authoredOn value="2024-09-26T13:35:54.741+00:00"/><lastModified value="2024-09-26T13:35:54.741+00:00"/><performerType><coding><system value="https://gematik.de/fhir/erp/CodeSystem/GEM_ERP_CS_OrganizationType"/><code value="urn:oid:1.2.276.0.76.4.54"/><display value="Öffentliche Apotheke"/></coding><text value="Öffentliche Apotheke"/></performerType></Task>
```

The only difference is the very first line which consists of `{transport protocol} {status code} {status text}`.
While the first two elements always use a single token, the status text can actually take up to 4 tokens (e.g. status code 505: `HTTP Version Not Supported`).
So the modelling is very straight forward:

```kotlin
class L1VauResEnvelope(
    val httpVersion: HttpVersion,
    val statusCode: StatusCode,
    val headers: VauHeaders,
    val body: BodyBytes,
)

data class StatusCode(val code: Int, val text: String)
```

### Level 2 Vau Response

A level 2 Vau response adds a Vau version and requestId in-front of the data from level 1. 

```text
1 0123456789abcdef0123456789abcdef HTTP/1.1 201 Created
Content-Type: application/fhir+xml;charset=utf-8
Content-Length: 1286

<?xml version="1.0" encoding="utf-8"?>
<Task xmlns="http://hl7.org/fhir"><id value="160.000.226.640.861.41"/><meta><profile value="https://gematik.de/fhir/erp/StructureDefinition/GEM_ERP_PR_Task|1.2"/></meta><extension url="https://gematik.de/fhir/erp/StructureDefinition/GEM_ERP_EX_PrescriptionType"><valueCoding><system value="https://gematik.de/fhir/erp/CodeSystem/GEM_ERP_CS_FlowType"/><code value="160"/><display value="Muster 16 (Apothekenpflichtige Arzneimittel)"/></valueCoding></extension><identifier><use value="official"/><system value="https://gematik.de/fhir/erp/NamingSystem/GEM_ERP_NS_PrescriptionId"/><value value="160.000.226.640.861.41"/></identifier><identifier><use value="official"/><system value="https://gematik.de/fhir/erp/NamingSystem/GEM_ERP_NS_AccessCode"/><value value="6d3aff43cb7f1db78e78dde37e6ae8db6f94e6c4291eb2b7bc8876a645ee3475"/></identifier><status value="draft"/><intent value="order"/><authoredOn value="2024-09-26T13:35:54.741+00:00"/><lastModified value="2024-09-26T13:35:54.741+00:00"/><performerType><coding><system value="https://gematik.de/fhir/erp/CodeSystem/GEM_ERP_CS_OrganizationType"/><code value="urn:oid:1.2.276.0.76.4.54"/><display value="Öffentliche Apotheke"/></coding><text value="Öffentliche Apotheke"/></performerType></Task>
```

which can be represented as

```kotlin
class L2VauResEnvelope(
    val version: Int = 1,
    val requestId: RequestId,
    val l1VauRes: L1VauResEnvelope,
)
```

with no new helper classes.

### Level 3 Vau Response

The data type for a level 3 Vau response is once more a simple wrapper around a byte array. This time without
additional structure since it has been created by symmetric encryption (using the `aesKey` of the level 2 Vau request).

```kotlin
@JvmInline
value class L3VauResEnvelopeAkaEncryptedL2(val bytes: ByteArray) {
    override fun toString(): String = "L3VauResEnvelopeAkaEncryptedL2(nrOfBytes=${bytes.size})"
}
```

### About the author

[Stephan Schröder](https://github.com/simon-void/) is a senior software developer and works for [gematik GmbH](https://www.gematik.de/).
He has a long history with Java and is intrigued by languages that offer even more safety guarantees at compiletime like [Kotlin](https://kotlinlang.org/) and [Rust](https://www.rust-lang.org/).
He also likes bouldering and [Aikido](https://flux-aikido.com/).