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

--- abstract
This specification defines how a series of security event tokens 
(SETs) may be delivered to a previously registered receiver 
using HTTP POST over TLS initiated as a push to the receiver.

--- middle
Introduction and Overview {#intro}
=========================
This specification defines how SETs (see
{{!SET}}) can be transmitted to a previously 
registered Event Receiver using HTTP {{!RFC7231}} over TLS. The
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
Event Transmitter
: A service provider that delivers SETs to other providers known
as Event Receivers.

Event Receiver
: A service provider that registers to receive SETs from an Event
Transmitter and provides an endpoint to receive SETs via HTTP POST. 


Event Delivery {#event_delivery}
=======================

Event Delivery Process {#event_delivery_process}
----------------------
When an Event occurs, the Event Transmitter constructs a SET
token {{!SET}} that describes the Event.
 
How SETs are defined and the process by which Events are identified for 
Event Receivers is out-of-scope of this    specification.

When a SET is available for an Event Receiver, the Event Transmitter
attempts to deliver the SET based on the Event Receiver's registered
delivery mechanism:

* The Event Transmitter uses an HTTP/1.1 POST to the Event Receiver
endpoint to deliver the SET;
* Or, the Event Transmitter delivers the Event through a different
method not defined by this specification.

In Push-Based SET Delivery Using HTTP, SETs are delivered one at a
time using HTTP POST requests by an Event Transmitter to an Event
Receiver. The HTTP request body is a JSON Web Token {{!RFC7519}}
with a `Content-Type` header of `application/secevent+jwt` as
defined in Section 2.2 and 6.2 of {{!SET}}. Upon receipt, the 
Event Receiver acknowledges receipt with a response with HTTP 
Status 202, as described below in {{httpPost}}.

After successful (acknowledged) SET delivery, Event 
Transmitters SHOULD NOT be required to maintain or record SETs for 
recovery. Once a SET is acknowledged, the Event Receiver SHALL be 
responsible for retention and recovery.

Transmitted SETs SHOULD be self-validating (e.g. signed)
if there is a requirement to verify they were issued by the Event 
Transmitter at a later date when de-coupled from the original 
delivery where authenticity could be checked via the HTTP or 
TLS mutual authentication.

Upon receiving a SET, the Event Receiver reads the SET and validates 
it. The Event Receiver MUST acknowledge receipt to the Event Transmitter, using the 
defined acknowledgement or error method.

The Event Receiver SHALL NOT use the Event acknowledgement mechanism
to report Event errors other than relating to the parsing and validation
of the SET.

Push Delivery using HTTP {#httpPost}
------------------------
This method allows an Event Transmitter to use HTTP POST 
(Section 4.3.3 {{!RFC7231}}) to deliver
SETs to a previously registered web callback URI supplied by the
Event Receiver as part of a configuration process 
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

To deliver an Event, the Event Transmitter generates an event 
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

Upon receipt of the request, the Event Receiver SHALL 
validate the JWT structure of the SET as defined in 
Section 7.2 {{!RFC7519}}. The Event Receiver 
SHALL also validate the SET information as described
in Section 2 {{!SET}}.

If the SET is determined to be valid, the Event Receiver SHALL
"acknowledge" successful submission by responding with HTTP Status
202 as `Accepted` 
(see Section 6.3.3 {{!RFC7231}}).

In order
to maintain compatibility with other methods of transmission, the 
Event Receiver SHOULD NOT include an HTTP response body representation
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
Event Transmitter know that a SET has been delivered and the 
information no longer needs to be retained by the Event Transmitter. 
Before acknowledgement, Event Receivers SHOULD ensure they have 
validated received SETs and retained them in a manner appropriate 
to information retention requirements appropriate to the SET 
event types signaled. The level and method of retention of SETs
by Event Receivers is out-of-scope of this specification.

In the Event of a general HTTP error condition, the Event Receiver
MAY respond with an appropriate HTTP Status code as defined in 
Section 6 {{!RFC7231}}.

When the Event Receiver detects an error parsing or 
validating a received SET (as defined by {{!SET}}), 
the Event Receiver SHALL indicate an HTTP Status 400 error with an 
error code as described in {{errorResponse}}.

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

Error Response Handling {#errorResponse}
-----------------------

If a SET is invalid, the following error codes are defined:

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
{: #reqErrors title="SET Errors"}

An error response SHALL include a JSON
object which provides details about the error. The JSON object
includes the JSON attributes:

{: vspace="0"}
err
: A value which is a keyword that 
describes the error (see {{reqErrors}}).
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
{{!RFC7235}}. For example, the following
methodologies could be used among others:

{: vspace="0"}
TLS Client Authentication
: Event delivery
endpoints MAY request TLS mutual client authentication. 
See Section 7.3 {{!RFC5246}}. 

Bearer Tokens
: Bearer tokens 
{{!RFC6750}} MAY be used when combined with TLS and a token
framework such as OAuth 2.0 {{!RFC6749}}. 
For security considerations regarding the use of bearer tokens in
SET delivery see {{bearerConsiderations}}.

Basic Authentication
: Usage of basic
authentication should be avoided due to its use of a single factor
that is based upon a relatively static, symmetric secret.
Implementers SHOULD combine the use of basic authentication with
other factors. The security considerations of HTTP BASIC, are well
documented in {{!RFC7617}} and SHOULD be considered
along with using signed SETs (see SET Payload Authentication below).

SET Payload Authentication
: In scenarios 
where SETs are signed and
the delivery method is HTTP POST (see {{httpPost}}),
Event Receivers MAY elect to use Basic Authentication or not 
to use HTTP or TLS based authentication at all. See 
{{payloadAuthentication}} for considerations.

As per Section 4.1 of {{!RFC7235}}, a SET
delivery endpoint SHALL indicate supported HTTP authentication 
schemes via the `WWW-Authenticate` header.

Because SET Delivery describes a simple function, authorization
for the ability to pick-up or deliver SETs can be derived by
considering the identity of the SET issuer, or via an authentication
method above. This specification considers authentication as a
feature to prevent denial-of-service attacks. Because SETs are
not commands (see ), Event Receivers are free to ignore SETs that 
are not of interest.

For illustrative purposes only, SET delivery examples show an OAuth2
bearer token value {{!RFC6750}} in the authorization header.
This is not intended to imply that bearer tokens are
preferred. However, the use of bearer tokens in the specification does
reflect common practice. 

Use of Tokens as Authorizations {#authzTokens}
-------------------------------
When using bearer tokens or proof-of-possession tokens that
represent an authorization grant such as issued by OAuth (see {{!RFC6749}}), implementers SHOULD consider the type of
authorization granted, any authorized scopes (see Section 3.3 of {{!RFC6749}}), and the security subject(s) that SHOULD be mapped
from the authorization when considering local access control rules.
Section 6 of the OAuth Assertions draft {{!RFC7521}}, documents common scenarios for
authorization including:

* Clients using an assertion to authenticate and/or act on behalf
of itself;

* Clients acting on behalf of a user; and,

* A Client acting on behalf of an anonymous user (e.g., see next
section).

When using OAuth authorization tokens, implementers MUST take
into account the threats and countermeasures documented in the
security considerations for the use of client authorizations (see
Section 8 of {{!RFC7521}}). When using
other token formats or frameworks, implementers MUST take into account
similar threats and countermeasures, especially those documented by
the relevant specifications.

Security Considerations {#Security}
=======================

Authentication Using Signed SETs {#payloadAuthentication}
--------------------------------
In scenarios where HTTP authorization or TLS mutual authentication
are not used or are considered weak, JWS signed SETs SHOULD be 
used (see {{!RFC7515}} and 
Security Considerations {{!SET}}). This enables the Event Receiver
to validate that the SET issuer is authorized to deliver SETs.

HTTP Considerations
-------------------
SET delivery depends on the use of Hypertext Transfer Protocol and thus
subject to the security considerations of HTTP Section 9 {{!RFC7230}} and its related specifications.

As stated in Section 2.7.1 {{!RFC7230}}, an 
HTTP requestor MUST NOT generate the `userinfo`
(i.e., username and password) component (and its "@" delimiter) when
an "http" URI reference is generated with a message as they are now
disallowed in HTTP.

TLS Support Considerations
--------------------------
SETs contain sensitive information that is considered PII
(e.g. subject claims). Therefore, Event Transmitters and
Event Receivers MUST require the use of a transport-layer 
security mechanism. Event delivery endpoints MUST support TLS 
1.2 {{!RFC5246}} and MAY support additional 
transport-layer mechanisms meeting its security requirements. 
When using TLS, the client MUST perform a TLS/SSL server
certificate check, per {{!RFC6125}}. Implementation
security considerations for TLS can be found in "Recommendations for
Secure Use of TLS and DTLS" {{!RFC7525}}.

Authorization Token Considerations
----------------------------------
When using authorization tokens such as those issued by OAuth 2.0
{{!RFC6749}}, implementers MUST take into account threats
and countermeasures documented in Section 8 of {{!RFC7521}}.

### Bearer Token Considerations {#bearerConsiderations}
Due to the possibility of interception, Bearer tokens MUST be 
exchanged using TLS.

Bearer tokens MUST have a limited lifetime that can be determined
directly or indirectly (e.g., by checking with a validation service)
by the service provider. By expiring tokens, clients are forced to
obtain a new token (which usually involves re-authentication) for
continued authorized access. For example, in OAuth2, a client MAY use
OAuth token refresh to obtain a new bearer token after authenticating
to an authorization server. See Section 6 of {{!RFC6749}}.

Implementations supporting OAuth bearer tokens need to factor in
security considerations of this authorization method 
{{!RFC7521}}. Since security is only as good
as the weakest link, implementers also need to consider authentication
choices coupled with OAuth bearer tokens. The security considerations
of the default authentication method for OAuth bearer tokens, HTTP
BASIC, are well documented in 
{{!RFC7617}}, therefore implementers
are encouraged to prefer stronger authentication methods. Designating
the specific methods of authentication and authorization are
out-of-scope for the delivery of SET tokens, however this 
information is provided as a resource to implementers.

Privacy Considerations
======================

If a SET needs to be retained for audit purposes, JWS MAY 
be used to provide verification of its authenticity.

When sharing personally identifiable information or information
that is otherwise considered confidential to affected users, Event 
Transmitters and Receivers MUST have the appropriate legal agreements
and user consent or terms of service in place.

The propagation of subject identifiers can be perceived as personally
identifiable information. Where possible, Event Transmitters and Receivers
SHOULD devise approaches that prevent propagation --- for example, the
passing of a hash value that requires the subscriber to already know
the subject.

IANA Considerations {#IANA}
===================

There are no IANA considerations.

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
