---
title: Push-Based SET Token Delivery Using HTTP
abbrev: draft-ietf-secevent-http-push
docname: draft-ietf-secevent-http-push-01
date: 2018-08-27
category: std
ipr: trust200902

area: Security
workgroup: Security Events Working Group
keyword: Internet-Draft

stand_alone: yes
pi: [toc, sortrefs, symrefs, comments]

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

{:richanna: source="AB"}

--- middle
Introduction {#intro}
============
This specification defines a mechanism by which a holder of a Security Event Token ({{!SET}}) may deliver the SET to an intended recipient via HTTP POST {{!HTTP}}.

Push-Based SET Delivery over HTTP POST is intended for scenarios where all of the following are true:

* The holder of the SET is capable of making outbound HTTP requests
* The recipient is capable of hosting an HTTP endpoint that is accessible to the transmitter
* The transmitter and recipient are known to one another
* The transmitter and recipient have an out-of-band mechanism for exchanging configuration metadata and/or security keys

Notational Conventions {#conv}
----------------------
The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL
NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED",
"MAY", and "OPTIONAL" in this document are to be interpreted as
described in BCP 14 {{!RFC2119}} {{!RFC8174}} when, and only when,
they appear in all capitals, as shown here.

Throughout this document all figures may contain spaces and extra
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


Transmitting a SET {#tx_request}
==================
To transmit a SET to a given SET Receiver, the SET Transmitter makes an HTTP POST request to the SET Receiver's SET Receipt Endpoint.  The `Content-Type` header of this request MUST be `application/secevent+jwt` as defined in Sections 2.2 and 6.2 of {{!SET}}, and the `Accept` header MUST be `application/json`. The body of the request MUST be the SET itself, represented as a {{!JWT}} in either JWE Compact Serialization or JWS Compact Serialization. [^jwt_format]{: source="AB"}

[^jwt_format]: The bit about JWE/JWS Compact Serialization is redundant, as per RFC7519 "JWTs are always represented using" one of those formats. But including it seems helpful.

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


Handling a SET Transmission Request {#tx_handling}
===================================
Upon receipt of a SET, the SET Receiver SHALL validate that all of the following are true:

* The SET Receiver can validate the SET as described in Section 7.2 of {{!JWT}} and Section 2 of {{!SET}}.
* The SET Receiver can validate the SET as described by the defining document(s) for the event represented within the SET, if applicable.
* The SET Receiver is identified as an intended audience of the SET.
* The SET Receiver can authenticate that the SET was issued by the issuer specified within the SET (See {{aa}}).

The SET Receiver MAY apply further validation tests, as appropriate to their use case.

The SET Receiver SHOULD NOT perform extensive business logic that processes the event expressed by the SET synchronously within the handling of a SET transmission request.  Such logic SHOULD be executed asynchronously from delivery, in order to minimize the expense and impact of SET delivery on the SET Transmitter.

Success Response {#tx_success}
----------------
If the SET Receiver successfully parses and validates the SET transmitted in a SET Transmission Request, the SET Receipt Endpoint SHALL respond with an HTTP Response Status Code of 202. The body of the response MUST be empty, and the response SHOULD NOT include a `Content-Type` header.

The following is a non-normative example of a response indicating that the SET was successfully delivered to the SET Receiver:

~~~
HTTP/1.1 202 Accepted
~~~
{: #successResponse title="Example Successful Delivery Response"}

Failure Response {#tx_failure}
----------------
If the SET Receiver encounters an error parsing or validating the SET transmitted in a SET Transmission Request, the SET Receipt Endpoint MUST respond with an HTTP Response Status Code of 400. The `Content-Type` header of this response MUST be "application/json", and the body of the response MUST be a JSON object containing two name/value pairs:

{: vspace="0"}
description
: A string whose value is a human-readable description of the error that occurred.

err
: A string whose value is the name of a Security Event Token Delivery Error Code.

The SET Receipt Endpoint MAY indicate generic HTTP errors by responding with an appropriate HTTP Response Status Code, as described in Section 6 of {{!HTTP}}.

The following is a non-normative response indicating a delivery failure due to a failure during signature validation:

~~~
HTTP/1.1 400 Bad Request
Content-Type: application/json

{
  "err":"jws",
  "description":"Unable to validate signature."

}
~~~
{: #failureResponse title="Example Delivery Failure Response"}

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


Authentication and Authorization {#aa}
================================
The mechanisms by which the SET Receipt Endpoint authenticates SETs and callers, and confirms that a caller is authorized to transmit a given SET are implementation specific and thus out-of-scope for this document.  SET Receipt Endpoints SHOULD use standard transportation layer security mechanisms (such as TLS 1.2 {{!TLS}} or later). When TLS is used, the SET Receipt Endpoint MUST support TLS 1.2 or greater, and MAY support other versions of TLS.

If integrity and authenticity are not provided at the transportation layer, the SET Receipt Endpoint MUST [^must_sign]{:richanna} sign and/or encrypt the SET in accordance with {{!JWS}} and {{!JWE}}. The SET Receipt Endpoint MAY use both transportation layer security and SET signatures and/or encryption.

[^must_sign]: Can we make this a MUST? Or are there use cases for making this a SHOULD?

The SET Receipt Endpoint MAY use standard HTTP authentication mechanisms (such as OAuth 2.0 Bearer tokens {{!RFC6750}}) in order to authenticate the caller.

See {{security}} for further guidance.


Delivery Reliability {#reliability}
====================
Delivery reliability requirements may vary from implementation to implementation.  SET Transmitters MAY attempt to re-transmit a SET if they have not received a response indicating successful delivery of that SET, and SHOULD wait some amount of time before attempting re-transmission.  The specific length of this waiting period may depend on the types of events being transmitted, and/or the SET Transmitter's or SET Receiver's requirements.  As such, it is out-of-scope for this specification.  SET Transmitters MAY apply an algorithm such as exponential backoff to determine how long to wait in between prior to re-transmission of a given SET.

SET Transmitters MUST NOT re-transmit a SET once the SET Receipt Endpoint sends a response indicating successful delivery of the SET.

SET Transmitters with high reliability requirements may be tempted to always retry failed transmissions, however it should be noted that for many types of SET delivery errors a retry is extremely unlikely to be successful.  For example, `json`, `jwtParse`, and `setParse` all indicate structural errors in the content of the SET that are likely to remain when re-transmitting the same SET.  Others such as `jws` or `jwe` may be transient, for example if cryptographic material has not been properly distributed to the SET Receipt Endpoint.

Implementers SHOULD evaluate their reliability requirements and the impact of various retry mechanisms on the performance of their systems to determine the correct strategy for various error conditions.

The SET Transmitter MAY provide an out-of-band mechanism by which a SET Receiver may be notified of delivery failures, and MAY retain SETs that it failed to deliver and make them available to the SET Receiver via other means.

The SET Receiver SHOULD ensure that the SET is persisted in a way that is sufficient to meet the SET Receiver's own reliability requirements, and MUST NOT [^retry_dep]{:richanna} expect or depend on a SET Transmitter to re-transmit or otherwise make available to the SET Receiver a SET once the SET Receiver acknowledges that it was received successfully.

[^retry_dep]: Does this MUST NOT make sense, given retention of SETs that failed delivery is permitted, per above?


Security Considerations {#security}
=======================

Denial-of-Service
-----------------
The SET Receipt Endpoint may be vulnerable to a denial-of-service attack where a malicious party makes a high volume of requests containing invalid SETs, causing the endpoint to expend significant resources on cryptographic operations that are bound to fail. This may be mitigated by authenticating SET Transmitters with a mechanism with low runtime overhead, such as mutual TLS, or statically assigned bearer tokens.

Authenticating Persisted SETs
-----------------------------
While the SET Receipt Endpoint can rely upon transport layer mechanisms, HTTP authentication methods, and/or other context from the transmission request to authenticate the SET Transmitter and validate the authenticity of the SET, this context is typically unavailable to systems that the SET Receipt Endpoint forwards the SET onto, or to systems that retrieve the SET from storage.  If the SET Receiver requires the ability to validate SET authenticity outside of the context of the SET Receipt Endpoint, then the SET Transmitter SHOULD sign the SET in accordance with {{!JWS}}, or encrypt it using an authenticated encryption scheme in accordance with {{!JWE}}.[^should_sign]{:richanna}

[^should_sign]: This is normative text. Should we move it out of Security Considerations?


Privacy Considerations
======================

When sharing personally identifiable information or information that is otherwise considered confidential to affected users, SET Transmitters and Receivers MUST have the appropriate legal agreements and user consent or terms of service in place.[^legal]{:richanna}

[^legal]: Does this belong in the specification?

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

Draft 00 - AB
-------------
Based on draft-ietf-secevent-delivery-02 with the following changes:

* Renamed to "Push-Based SET Token Delivery Using HTTP"
* Removed references to the HTTP Polling delivery method.
* Removed informative reference to RFC6202.

Draft 01 - AB
-------------

* Removed "Implementers MUST percent encode URLs..."
* Changed MAY to may in "...all figures MAY contain spaces and extra line-wrapping..."
* Rewrote most of Abstract, Introduction,
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
* Made it a normative requirement for SET Receivers to not depend on the SET Transmitter persisting successfully delivered SETs.
* Added IANA registry for SET delivery error codes.
