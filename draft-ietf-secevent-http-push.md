---
title: Push-Based SET Token Delivery Using HTTP
abbrev: draft-ietf-secevent-http-push
docname: draft-ietf-secevent-http-push-00
date: 2018-08-27
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
    SET: RFC8417
    BCP26: RFC8126

--- abstract
This specification defines how a series of security event tokens 
(SETs) may be delivered to a previously registered receiver 
using HTTP POST over TLS initiated as a push to the receiver.

--- middle
Introduction and Overview {#intro}
=========================
This specification defines how SETs (see
{{!SET}}) can be transmitted to a previously 
registered SET Receiver using HTTP {{!RFC7231}} over TLS. The
specification defines a method to push SETs via HTTP POST. 

Notational Conventions {#conv}
----------------------
The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL
NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED",
"MAY", and "OPTIONAL" in this document are to be interpreted as
described in BCP 14 {{!RFC2119}} {{!RFC8174}} when, and only when,
they appear in all capitals, as shown here.

For purposes of readability examples are not URL encoded.
Implementers MUST percent encode URLs as described in Section 2.1 of
{{!RFC3986}}.

Throughout this documents all figures MAY contain spaces and extra
line-wrapping for readability and space limitations. Similarly, some
URI's contained within examples, have been shortened for space and
readability reasons.

Definitions {#defn}
-------------------
This specification assumes terminology defined in the Security
Event Token specification {{!SET}}, as well as the terms defined below:

{: vspace="0"}
SET Transmitter
: A service provider that delivers SETs to other providers known
as SET Receivers.

SET Receiver
: A service provider that registers to receive SETs from an SET
Transmitter and provides an endpoint to receive SETs via HTTP POST. 


Event Delivery {#event_delivery}
=======================

Event Delivery Process {#event_delivery_process}
----------------------
When an Event occurs, the SET Transmitter constructs a SET
token {{!SET}} that describes the Event.
 
How SETs are defined and the process by which Events are identified for 
SET Receivers is out-of-scope of this    specification.

When a SET is available for an SET Receiver, the SET Transmitter
attempts to deliver the SET based on the SET Receiver's registered
delivery mechanism:

* The SET Transmitter uses an HTTP/1.1 POST to the SET Receiver
endpoint to deliver the SET;
* Or, the SET Transmitter delivers the Event through a different
method not defined by this specification.

In Push-Based SET Delivery Using HTTP, SETs are delivered one at a
time using HTTP POST requests by an SET Transmitter to an SET
Receiver. The HTTP request body is a JSON Web Token {{!RFC7519}}
with a `Content-Type` header of `application/secevent+jwt` as
defined in Section 2.2 and 6.2 of {{!SET}}. Upon receipt, the 
SET Receiver acknowledges receipt with a response with HTTP 
Status 202, as described below in {{httpPost}}.

After successful (acknowledged) SET delivery, SET 
Transmitters SHOULD NOT be required to maintain or record SETs for 
recovery. Once a SET is acknowledged, the SET Receiver SHALL be 
responsible for retention and recovery.

Transmitted SETs SHOULD be self-validating (e.g. signed)
if there is a requirement to verify they were issued by the SET 
Transmitter at a later date when de-coupled from the original 
delivery where authenticity could be checked via the HTTP or 
TLS mutual authentication.

Upon receiving a SET, the SET Receiver reads the SET and validates 
it. The SET Receiver MUST acknowledge receipt to the SET Transmitter, using the 
defined acknowledgement or error method.

The SET Receiver SHALL NOT use the Event acknowledgement mechanism
to report Event errors other than relating to the parsing and validation
of the SET.

Push Delivery using HTTP {#httpPost}
------------------------
This method allows an SET Transmitter to use HTTP POST 
(Section 4.3.3 {{!RFC7231}}) to deliver
SETs to a previously registered web callback URI supplied by the
SET Receiver as part of a configuration process 
(not defined by this document).

The SET to be delivered MAY be signed 
and/or encrypted as defined in {{!SET}}.

The HTTP Content-Type (see 
Section 3.1.1.5 {{!RFC7231}}) for the HTTP POST is 
`application/secevent+jwt` and SHALL consist of 
a single SET (see {{!SET}}).
As per Section 5.3.2 {{!RFC7231}}, the expected 
media type (`Accept` header) response is 
`application/json`.

To deliver an Event, the SET Transmitter generates an event 
delivery message and uses HTTP POST to the configured endpoint with
the appropriate `Accept` and 
`Content-Type` headers.

~~~
POST /Events  HTTP/1.1

Host: notify.examplerp.com
Accept: application/json
Authorization: Bearer h480djs93hd8
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
{: #postSet title="Example HTTP POST Request"}

Upon receipt of the request, the SET Receiver SHALL 
validate the JWT structure of the SET as defined in 
Section 7.2 {{!RFC7519}}. The SET Receiver 
SHALL also validate the SET information as described
in Section 2 {{!SET}}.

If the SET is determined to be valid, the SET Receiver SHALL
"acknowledge" successful submission by responding with HTTP Status
202 as `Accepted` 
(see Section 6.3.3 {{!RFC7231}}).

In order
to maintain compatibility with other methods of transmission, the 
SET Receiver SHOULD NOT include an HTTP response body representation
of the submitted SET or what the SET's pending status is when 
acknowledging success. In the case of an error (e.g. HTTP Status 400),
the purpose of the HTTP response body is to indicate any SET parsing, 
validation, or cryptographic errors.

The following is a non-normative example of a successful
receipt of a SET.

~~~
HTTP/1.1 202 Accepted
~~~
{: #goodPostResponse title="Example Successful Delivery Response"}

Note that the purpose of the "acknowledgement" response is to let the 
SET Transmitter know that a SET has been delivered and the 
information no longer needs to be retained by the SET Transmitter. 
Before acknowledgement, SET Receivers SHOULD ensure they have 
validated received SETs and retained them in a manner appropriate 
to information retention requirements appropriate to the SET 
event types signaled. The level and method of retention of SETs
by SET Receivers is out-of-scope of this specification.

In the Event of a general HTTP error condition, the SET Receiver
MAY respond with an appropriate HTTP Status code as defined in 
Section 6 {{!RFC7231}}.

When the SET Receiver detects an error parsing or 
validating a received SET (as defined by {{!SET}}), 
the SET Receiver SHALL indicate an HTTP Status 400 error with an 
error response as described in {{errorResponse}}.

The following is an example non-normative error 
response.

~~~
HTTP/1.1 400 Bad Request
Content-Type: application/json

{
  "err":"dup",
  "description":"SET already received. Ignored."

}
~~~
{: #badPostResponse title="Example HTTP Status 400 Response"}

Security Event Token Delivery Error Codes {#error_codes}
-----------------------------------------
Security Event Token Delivery Error Codes are strings that identify a specific type of error that may occur when parsing or validating a SET. Every Security Event Token Delivery Error Code MUST have a unique name registered in the IANA "Security Event Token Delivery Error Codes" registry established by {{iana_set_errors}}.

The following table presents the initial set of Error Codes that are registered in the IANA "Security Event Token Delivery Error Codes" registry:

| Err Value | Description
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

Error Response Handling {#errorResponse}
-----------------------
An error response SHALL include a JSON
object which provides details about the error. The JSON object
includes the JSON attributes:

{: vspace="0"}
err
: A Security Event Token Delivery Error Code (see {{error_codes}}).
description
: A human-readable text that provides
additional diagnostic information.

When included as part of an HTTP Status 400 response, the above
JSON is the HTTP response body (see {{badPostResponse}}).

Authentication and Authorization {#aa}
================================

The SET delivery method described in this specification is
based upon HTTP and depends on the use of TLS and/or standard 
HTTP authentication and authorization schemes as per 
{{!RFC7235}}.

Because SET Delivery describes a simple function, authorization
for the ability to pick-up or deliver SETs can be derived by
considering the identity of the SET issuer, or via other employed
authentication methods.  Because SETs are
not commands (see ), SET Receivers are free to ignore SETs that 
are not of interest.

For illustrative purposes only, SET delivery examples show an OAuth2
bearer token value {{!RFC6750}} in the authorization header.
This is not intended to imply that bearer tokens are
preferred. However, the use of bearer tokens in the specification does
reflect common practice. 


Security Considerations {#Security}
=======================

Authentication Using Signed SETs {#payloadAuthentication}
--------------------------------
In scenarios where HTTP authorization or TLS mutual authentication
are not used or are considered weak, JWS signed SETs SHOULD be 
used (see {{!RFC7515}} and 
Security Considerations {{!SET}}). This enables the SET Receiver
to validate that the SET issuer is authorized to deliver SETs.

TLS Support Considerations
--------------------------
SETs contain sensitive information that is considered PII
(e.g. subject claims). Therefore, SET Transmitters and
SET Receivers MUST require the use of a transport-layer 
security mechanism. Event delivery endpoints MUST support TLS 
1.2 {{!RFC5246}} and MAY support additional 
transport-layer mechanisms meeting its security requirements. 
When using TLS, the client MUST perform a TLS/SSL server
certificate check, per {{!RFC6125}}. Implementation
security considerations for TLS can be found in "Recommendations for
Secure Use of TLS and DTLS" {{?RFC7525}}.

Denial of Service
-----------------
The SET Receiver may be vulnerable to a denial-of-service attack where a malicious party makes a high volume of requests containing invalid SETs, causing the endpoint to expend significant resources on cryptographic operations that are bound to fail. This may be mitigated by authenticating SET Transmitters with a mechanism with low runtime overhead, such as mutual TLS or statically assigned bearer tokens.

Privacy Considerations
======================

If a SET needs to be retained for audit purposes, JWS MAY 
be used to provide verification of its authenticity.

When sharing personally identifiable information or information
that is otherwise considered confidential to affected users, SET 
Transmitters and Receivers MUST have the appropriate legal agreements
and user consent or terms of service in place.

The propagation of subject identifiers can be perceived as personally
identifiable information. Where possible, SET Transmitters and Receivers
SHOULD devise approaches that prevent propagation --- for example, the
passing of a hash value that requires the subscriber to already know
the subject.

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
Other Streaming Specifications
==============================
\[\[EDITORS NOTE: This section to be removed prior to publication]]

The following pub/sub, queuing, streaming systems were reviewed 
as possible solutions or as input to the current draft:

XMPP Events

The WG considered the XMPP events ands its ability to provide a single
messaging solution without the need for both polling and push modes.
The feeling was the size and methodology of XMPP was to far apart from
the current capabilities of the SECEVENTs community which focuses in
on HTTP based service delivery and authorization.

Amazon Simple Notification Service

Simple Notification Service, is a pub/sub messaging product from 
AWS. SNS supports a variety of subscriber types: HTTP/HTTPS endpoints, 
AWS Lambda functions, email addresses (as JSON or plain text), phone 
numbers (via SMS), and AWS SQS standard queues. It doesn’t directly 
support pull, but subscribers can get the pull model by creating an 
SQS queue and subscribing it to the topic. Note that this puts the 
cost of pull support back onto the subscriber, just as it is in the 
push model. It is not clear that one way is strictly better than the 
other; larger, sophisticated developers may be happy to own message 
persistence so they can have their own internal delivery guarantees. 
The long tail of OIDC clients may not care about that, or may fail 
to get it right. Regardless, I think we can learn something from the 
Delivery Policies supported by SNS, as well as the delivery controls 
that SQS offers (e.g. Visibility Timeout, Dead-Letter Queues). I’m 
not suggesting that we need all of these things in the spec, but 
they give an idea of what features people have found useful.

Other information:

* API Reference: http://docs.aws.amazon.com/AWSSimpleQueueService/latest/APIReference/Welcome.html
* Visibility Timeouts: http://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/sqs-visibility-timeout.html

Apache Kafka

Apache Kafka is an Apache open source project based upon TCP for 
distributed streaming. It prescribes some interesting general 
purpose features that seem to extend far beyond the simpler 
streaming model SECEVENTs is after. A comment from MS has been that 
Kafka does an acknowledge with poll combination event which seems 
to be a performance advantage. See: https://kafka.apache.org/intro

Google Pub/Sub

Google Pub Sub system favours a model whereby polling and acknowledgement
of events is done as separate endpoints as separate functions.

Information:

* Cloud Overview - https://cloud.google.com/pubsub/
* Subscriber Overview - https://cloud.google.com/pubsub/docs/subscriber
* Subscriber Pull(poll) - https://cloud.google.com/pubsub/docs/pull

Acknowledgments
===============
The editors would like to thanks the members of the SCIM WG which 
began discussions of provisioning events starting with: draft-hunt-scim-notify-00 in 2015.

The editors would like to thank the authors of draft-ietf-secevent-delivery-02,
on which this draft is based.

The editor would like to thank the participants in the the SECEVENTS
working group for their support of this specification.

Change Log
==========
Draft 00 - AB - Based on draft-ietf-secevent-delivery-02 with the 
following changes:

* Renamed to "Push-Based SET Token Delivery Using HTTP"
* Removed references to the HTTP Polling delivery method.
* Removed informative reference to RFC6202.

Draft 01 - AB

* Converted to Markdown
* Removed NumericDate definition (unused).
* Removed Event and Subject definitions (defined in SET).
* Removed text related to Event Streams.
* Removed Mike Jones and Phil Hunt as editors, per respective requests.
* Fixed area and workgroup to match secevent.
* Renamed Event Transmitter and Event Receiver to SET Transmitter and SET Receiver, respectively.
* Added IANA registry for SET Delivery Error Codes.
* Removed enumeration of HTTP authentication methods.
* Removed generally applicable guidance for HTTP, authorization tokens, and bearer tokens.
* Elaborated on guidance for DoS protection via authn, and moved it to Security Considerations.
* Removed redundant instruction to use `WWW-Authenticate` header.
* Removed further generally applicable guidance for authorization tokens.
