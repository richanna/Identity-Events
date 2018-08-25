---
title: Push-Based SET Token Delivery Using HTTP
abbrev: draft-ietf-secevent-http-push
docname: draft-ietf-secevent-http-push-01
date: 2018-08-23
category: std
ipr: trust200902

area: Security
workgroup: Security Events Working Group
keyword: Internet-Draft

stand_alone: yes
pi: [toc, sortrefs, symrefs]

author:
-
    ins: A. Backman
    name: Annabelle Backman
    organization: Amazon
    email: richanna@amazon.com
    role: editor
-
    ins: M. Jones
    name: Michael B. Jones
    organization: Microsoft
    email: mbj@microsoft.com
    uri: http://self-issued.info/
-
    ins: P. Hunt
    name: Phil Hunt
    organization: Oracle
    email: phil.hunt@yahoo.com
-
    ins: M. Scurtescu
    name: Marius Scurtescu
    organization: Google
    email: mscurtescu@google.com
-
    ins: M. Ansari
    name: Morteza Ansari
    organization: Cisco
    email: morteza.ansari@cisco.com
-
    ins: A. Nadalin
    name: Anthony Nadalin
    organization: Microsoft
    email: tonynad@microsoft.com

normative:
    BCP26: RFC8126
    JWT: RFC7519
    JWS: RFC7515
    JWE: RFC7516
    SET: RFC8417
    TLS: RFC5246
    HTTP: RFC7231

--- abstract
This specification defines how a Security Event Token (SET) may be delivered to an intended recipient using HTTP POST. The SET is transmitted in the body of an HTTP POST request to a previously registered endpoint, and the recipient indicates success or failure via the HTTP response.

--- middle
Introduction {#intro}
============
This specification defines a mechanism by which a holder of a Security Event Token ({{!SET}}) may deliver the SET to an intended recipient via HTTP POST {{!HTTP}}.

Push-Based SET Delivery over HTTP POST is intended for scenarios where:

* The holder of the SET is capable of making outbound HTTP requests
* and the recipient is capable of hosting an HTTP endpoint that is accessible to the transmitter
* and the transmitter and recipient are known to one another
* and the transmitter and recipient have an out-of-band mechanism for exchanging configuration metadata and/or security keys

Notational Conventions {#conv}
----------------------
The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL
NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED",
"MAY", and "OPTIONAL" in this document are to be interpreted as
described in BCP 14 {{!RFC2119}} {{!RFC8174}} when, and only when,
they appear in all capitals, as shown here.

Throughout this document all figures MAY contain spaces and extra
line-wrapping for readability and space limitations.

Definitions {#defn}
-------------------
This specification assumes terminology defined in the Security
Event Token specification {{!SET}}, as well as the terms defined below:

{: vspace="0"}
SET Receipt Endpoint
: An HTTP endpoint operated by the SET Receiver, which is configured to
receive SETs in accordance with this specification.

SET Receiver
: An entity that is prepared to receive one or more SETs from a SET
Transmitter.

SET Transmitter
: An entity that is in possession of a SET that is to be transmitted to a
SET Receiver. This MAY be the issuer of the SET or an entity affiliated
with the issuer, but this is neither guaranteed nor required to be the case.

SET Delivery {#event_delivery}
============
To deliver a SET to a given SET Receiver, the SET Transmitter makes a request to the SET Receipt Endpoint. This request contains within it the SET itself. The SET Receiver replies to this request with a response indicating whether or not the SET was successfully received and validated by the SET Receiver.

Upon receipt of a SET, the SET Receiver SHOULD validate that all of the following are true:

* The SET Receiver can parse the SET
* The SET is authentic (i.e. it was issued by the issuer specified within the SET)
* The SET Receiver is identified as the intended audience of the SET

The mechanisms by which the SET Receiver performs this validation are out-of-scope for this document. SET parsing and issuer and audience identification are defined in {{!SET}}. How the SET Receiver may validate authenticity is implementation specific, and will vary depending on authentication mechanisms, and whether the SET is signed and/or encrypted (See {{aa}}).

The SET Receiver SHOULD ensure that the SET is persisted in a way that is sufficient to meet the SET Receiver's own reliability requirements, and MUST NOT expect or depend on a SET Transmitter to re-transmit or otherwise make available to the SET Receiver a SET once the SET Receiver acknowledges that it was received successfully.

Once the SET has been validated and persisted, the SET Receiver SHOULD immediately return a response indicating that the SET was successfully delivered. The SET Receiver SHOULD NOT perform extensive business logic that processes the event expressed by the SET prior to sending this response. Such logic SHOULD be executed asynchronously from delivery, in order to minimize the expense and impact of SET delivery on the SET Transmitter.

The SET Transmitter SHOULD NOT re-transmit a SET, unless the response from the SET Receipt Endpoint in previous transmissions indicated a potentially recoverable error (such as server unavailability that may be transient, or a decryption failure that may be due to misconfigured keys on the SET Receiver's side). In the latter case, the SET Transmitter MAY re-transmit a SET, after an appropriate delay to avoid browning out the SET Receipt Endpoint (See {{reliability}}).

The SET Transmitter MAY provide an out-of-band mechanism by which a SET Receiver may be notified of delivery failures, and MAY retain SETs that it failed to deliver and make them available to the SET Receiver via other means.

SET Transmission Request {#tx_request}
------------------------
To transmit a SET to a given SET Receiver, the SET Transmitter makes an HTTP POST request to the SET Receiver's SET Receipt Endpoint.  The `Content-Type` header of this request MUST be `application/secevent+jwt` as defined in Sections 2.2 and 6.2 of {{!SET}}, and the request body MUST be the SET itself, represented as a {{!JWT}}.

The mechanism by which the SET Transmitter determines the SET Receipt Endpoint is not defined by this specification and may be implementation-specific.

The following is a non-normative example of a SET transmission request:

~~~
POST /Events  HTTP/1.1

Host: notify.examplerp.com
Accept: application/json
Content-Type: application/secevent+jwt

eyJhbGciOiJub25lIn0
.
eyJwdWJsaXNoZXJVcmkiOiJodHRwczovL3NjaW0uZXhhbXBsZS5jb20iLCJmZWV
kVXJpcyI6WyJodHRwczovL2podWIuZXhhbXBsZS5jb20vRmVlZHMvOThkNTI0Nj
FmYTViYmM4Nzk1OTNiNzc1NCIsImh0dHBzOi8vamh1Yi5leGFtcGxlLmNvbS9GZ
WVkcy81ZDc2MDQ1MTZiMWQwODY0MWQ3Njc2ZWU3Il0sInJlc291cmNlVXJpcyI6
WyJodHRwczovL3NjaW0uZXhhbXBsZS5jb20vVXNlcnMvNDRmNjE0MmRmOTZiZDZ
hYjYxZTc1MjFkOSJdLCJldmVudFR5cGVzIjpbIkNSRUFURSJdLCJhdHRyaWJ1dG
VzIjpbImlkIiwibmFtZSIsInVzZXJOYW1lIiwicGFzc3dvcmQiLCJlbWFpbHMiX
SwidmFsdWVzIjp7ImVtYWlscyI6W3sidHlwZSI6IndvcmsiLCJ2YWx1ZSI6Impk
b2VAZXhhbXBsZS5jb20ifV0sInBhc3N3b3JkIjoibm90NHUybm8iLCJ1c2VyTmF
tZSI6Impkb2UiLCJpZCI6IjQ0ZjYxNDJkZjk2YmQ2YWI2MWU3NTIxZDkiLCJuYW
1lIjp7ImdpdmVuTmFtZSI6IkpvaG4iLCJmYW1pbHlOYW1lIjoiRG9lIn19fQ
.
~~~
{: #postSet title="Example SET Transmission Request"}

Successful Delivery {#tx_success}
-------------------
If the SET Receiver successfully parses and validates the SET transmitted in a SET Transmission Request, the SET Receipt Endpoint MUST respond with an HTTP Response Status Code of 202. The body of the response MUST be empty.

The following is a non-normative example of a response indicating that the SET was successfully delivered to the SET Receiver:

~~~
HTTP/1.1 202 Accepted
~~~
{: #goodPostResponse title="Example Successful Delivery Response"}

Delivery Failure {#tx_failure}
----------------
The SET Receipt Endpoint MAY indicate generic HTTP errors by responding with an appropriate HTTP Response Status Code, as described in Section 6 of {{!HTTP}}.

If the SET Receiver encounters an error parsing or validating the SET transmitted in a SET Transmission Request, the SET Receipt Endpoint MUST respond with an HTTP Response Status Code of 400. The `Content-Type` header of this response MUST be "application/json", and the body of the response MUST be a JSON object containing two name/value pairs:

{: vspace="0"}
description
: A string whose value is a human-readable description of the error that occurred.

err
: A string whose value is the name of a Security Event Token Delivery Error Code.

### Security Event Token Delivery Error Codes {#error_codes}
Security Event Token Delivery Error Codes are strings that identify a specific type of error that may occur when parsing or validating a SET. Every Security Event Token Delivery Error Code MUST have a unique name registered in the IANA "Security Event Token Delivery Error Codes" registry established by {{iana_set_errors}}.

The following table presents the initial set of Error Codes that are registered in the IANA "Security Event Token Delivery Error Codes" registry:

| Error Code | Description
|-----------+-------------|
| json      | Invalid JSON object. |
| jwtParse  | Invalid or unparsable JWT or JSON structure. |
| jwtHdr    | In invalid JWT header was detected. |
| jwtCrypto | Unable to parse due to unsupported algorithm. |
| jws       | Signature was not validated. |
| jwe       | Unable to decrypt JWE encoded data. |
| jwtAud    | Invalid audience value. |
| jwtIss    | Issuer not recognized. |
| setType   | An unexpected Event type was received. |
| setParse  | Invalid structure was encountered such as an inability to parse or an incomplete set of Event claims. |
| setData   | SET event claims incomplete or invalid. |
| dup       | A duplicate SET was received and has been ignored. |
{: #tabErrors title="SET Delivery Error Codes"}

The following is a non-normative response indicating a delivery failure due to a failure during signature validation:

~~~
HTTP/1.1 400 Bad Request
Content-Type: application/json

{
  "err":"jws",
  "description":"Unable to validate signature."

}
~~~
{: #badPostResponse title="Example Delivery Failure Response"}


Authentication and Authorization {#aa}
================================
The mechanisms by which the SET Receipt Endpoint authenticates callers and confirms that they are authorized to transmit the SET are implementation specific and out-of-scope for this document. SET Receipt Endpoints SHOULD use standard transportation layer security mechanisms (such as TLS 1.2 {{!TLS}} or later). When TLS is used, the SET Receipt Endpoint MUST support TLS 1.2 or greater, and MAY support other versions of TLS.

If integrity and authenticity are not provided at the transportation layer, the SET Receipt Endpoint MUST sign and/or encrypt the SET in accordance with {{!JWS}} and {{!JWE}}. The SET Receipt Endpoint MAY use both transportation layer security and SET signatures and/or encryption.

The SET Receipt Endpoint MAY use standard HTTP authentication mechanisms (such as OAuth 2.0 Bearer tokens {{!RFC6750}}) in order to authenticate the caller.

See {{security}} for further guidance.


Delivery Reliability {#reliability}
====================
Delivery reliability requirements may vary from implementation to implementation.  This specification provides the response from the SET Receipt Endpoint in such a way as to provide the SET Transmitter with the information necessary to determine what further action is required, if any, in order to meet their requirements.  SET Transmitters with high reliability requirements may be tempted to always retry failed transmissions, however it should be noted that for many types of SET delivery errors, a retry is extremely unlikely to be successful.  For example, `json`, `jwtParse`, and `setParse` all indicate structural errors in the content of the SET that are likely to remain when re-transmitting the same SET.  Others such as `jws` or `jwe` may be transient, for example if cryptographic material has not been properly distributed to the SET Receipt Endpoint.

Implementers SHOULD evaluate their reliability requirements and the impact of various retry mechanisms on the performance of their systems to determine the correct strategy for various error conditions.


Security Considerations {#security}
=======================

Denial-of-Service
-----------------
The SET Receipt Endpoint may be vulnerable to a denial-of-service attack where a malicious party makes a high volume of requests containing invalid SETs, causing the endpoint to expend significant resources on cryptographic operations that are bound to fail. This may be mitigated by implementing an authentication mechanism with low runtime overhead, such as mutual TLS, or statically assigned bearer tokens.

Authenticating Persisted SETs
-----------------------------
While the SET Receipt Endpoint can rely upon transport layer mechanisms, HTTP authentication methods, and/or other context from the transmission request to authenticate the SET Transmitter and validate the authenticity of the SET, this context is typically unavailable to systems that the SET Receipt Endpoint forwards the SET onto, or to systems that retrieve the SET from storage.  If the SET Receiver requires the ability to validate SET authenticity outside of the context of the SET Receipt Endpoint, then the SET Transmitter SHOULD sign the SET in accordance with {{!JWS}}, or encrypt it using an authenticated encryption scheme in accordance with {{!JWE}}.


Privacy Considerations
======================

When sharing personally identifiable information or information that is otherwise considered confidential to affected users, SET Transmitters and Receivers MUST have the appropriate legal agreements and user consent or terms of service in place.

The propagation of subject identifiers can be perceived as personally identifiable information. Where possible, SET Transmitters and Receivers SHOULD devise approaches that prevent propagation --- for example, the passing of a hash value that requires the subscriber to already know the subject.

IANA Considerations {#IANA}
===================

Security Event Token Delivery Error Codes {#iana_set_errors}
-----------------------------------------
This document defines Security Event Token Delivery Error Codes, for which IANA is asked to create and maintain a new registry titled "Security Event Token Delivery Error Codes".  Initial values for the Security Event Token Delivery Error Codes registry are given in {{tabErrors}}.  Future assignments are to be made through the Expert Review registration policy ({{!BCP26}}) and shall follow the template presented in {{iana_set_errors_template}}.

### Registration Template {#iana_set_errors_template}

{: vspace="0"}
Error Code
: The name of the Security Event Token Delivery Error Code, as described in {{error_codes}}. The name MUST be a case-sensitive ASCII string consisting only of upper-case letters ("A" - "Z"), lower-case letters ("a" - "z"), and digits ("0" - "9").

Descrption
: A brief human-readable description of the Security Event Token Delivery Error Code.

Change Controller
: For error codes registered by the IETF or its working groups, list "IETF Secevent Working Group".  For all other error codes, list the name of the party responsible for the registration.  Contact information such as mailing address, email address, or phone number may also be provided.

Defining Document(s)
: A reference to the document or documents that define the Security Event Token Delivery Error Code. The definition MUST specify the name and description of the error code, and explain under what circumstances the error code may be used. URIs that can be used to retrieve copies of each document at no cost SHOULD be included.

### Initial Registry Contents

#### json
* Name: json
* Description: Invalid JSON object.
* Change Controller: IETF Secevent Working Group
* Defining Document(s): {{error_codes}} of this document.

#### jwtParse
* Name: jwtParse
* Description: Invalid or unparsable JWT or JSON structure.
* Change Controller: IETF Secevent Working Group
* Defining Document(s): {{error_codes}} of this document.

#### jwtHdr
* Name: jwtHdr
* Description: In invalid JWT header was detected.
* Change Controller: IETF Secevent Working Group
* Defining Document(s): {{error_codes}} of this document.

#### jwtCrypto
* Name: jwtCrypto
* Description: Unable to parse due to unsupported algorithm.
* Change Controller: IETF Secevent Working Group
* Defining Document(s): {{error_codes}} of this document.

#### jws
* Name: jws
* Description: Signature was not validated.
* Change Controller: IETF Secevent Working Group
* Defining Document(s): {{error_codes}} of this document.

#### jwe
* Name: jwe
* Description: Unable to decrypt JWE encoded data.
* Change Controller: IETF Secevent Working Group
* Defining Document(s): {{error_codes}} of this document.

#### jwtAud
* Name: jwtAud
* Description: Invalid audience value.
* Change Controller: IETF Secevent Working Group
* Defining Document(s): {{error_codes}} of this document.

#### jwtIss
* Name: jwtIss
* Description: Issuer not recognized.
* Change Controller: IETF Secevent Working Group
* Defining Document(s): {{error_codes}} of this document.

#### setType
* Name: setType
* Description: An unexpected Event type was received.
* Change Controller: IETF Secevent Working Group
* Defining Document(s): {{error_codes}} of this document.

#### setParse
* Name: setParse
* Description: Invalid structure was encountered such as an inability to parse or an incomplete set of Event claims.
* Change Controller: IETF Secevent Working Group
* Defining Document(s): {{error_codes}} of this document.

#### setData
* Name: setData
* Description: SET event claims incomplete or invalid.
* Change Controller: IETF Secevent Working Group
* Defining Document(s): {{error_codes}} of this document.

#### dup
* Name: dup
* Description: A duplicate SET was received and has been ignored.
* Change Controller: IETF Secevent Working Group
* Defining Document(s): {{error_codes}} of this document.

--- back

Acknowledgments
===============
The authors would like to thank the members of the SCIM WG which
began discussions of provisioning events starting with: draft-hunt-scim-notify-00 in 2015.

The authors would like to thank the authors of draft-ietf-secevent-delivery-02, on which this draft is based.

Change Log
==========
Draft 00 - AB - Based on draft-ietf-secevent-delivery-02 with the following changes:

* Renamed to "Push-Based SET Token Delivery Using HTTP"
* Removed references to the HTTP Polling delivery method.
* Removed informative reference to RFC6202.

Draft 01 - AB:

* Removed all text related to Event Streams.
* Added "SET Receipt Endpoint" definition.
* Removed Subject and Event definitions (already defined in SET).
* Removed NumericDate definiton (unused).
* Removed "Event Streams" definition (now unused).
* Replaced "Event Transmitter" with "SET Transmitter".
* Replaced "Event Receiver" with "SET Receiver".
* Replaced "Event Receiver endpoint" with "SET Receipt Endpoint".
* Removed enumeration of HTTP authentication schemes.
* Revised text related to authentication schemes and denial-of-service protection, and moved it to Security Considerations.
* Removed generally applicable guidance regarding HTTP, TLS, authorization tokens, and bearer tokens.
* Moved text regarding using JWS to determine authenticity of retained SETs to Security Considerations.
* Added guidance regarding retries and non-transient error conditions.
* Added IANA registry for SET delivery error codes.
