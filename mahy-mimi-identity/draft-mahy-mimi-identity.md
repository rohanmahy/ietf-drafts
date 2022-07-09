%%%
title = "More Instant Messaging Interoperability (MIMI) Identity Concepts"
abbrev = "MIMI Identity"
ipr= "trust200902"
area = "art"
workgroup = "mimi"
keyword = ["mimi","immi","identity"]

[seriesInfo]
status = "informational"
name = "Internet-Draft"
value = "draft-mahy-mimi-identity-00"
stream = "IETF"

[[author]]
initials="R."
surname="Mahy"
fullname="Rohan Mahy"
organization = "Wire"
  [author.address]
  email = "rohan.mahy@wire.com"
%%%

.# Abstract

This document discusses concepts in instant messaging identity interoperability
especially when using end-to-end encryption, for example with the MLS 
(Message Layer Security) Protocol.

{mainmatter}

# Terminology
The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD",
"SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to
be interpreted as described in RFC 2119 [@!RFC2219].

## Description


Identifiers and cryptographic methods of making and verifying these assertions







The IETF has been working on Instant Messaging Interoperability since the late 1990s. 
At the time, different groups within IETF proposed separate protocol suites (SIMPLE,
APEX, and XMPP) because the community could not come to consensus on a single protocol
(arguably due to a lack of consensus on the additional requirements which made these
proposals unique). In the interests of interoperability, the IMPP Working Group
developed a general framework for interoperability [@!RFC3860], and the Common Presence
and Instant Messaging (CPIM) Message Format [@!RFC3862], an interoperability format that
could pass through gateways among these protocols, even when end-to-end encrypted.  

The CPIM model assumed standalone encryption of each message using a protocol such
as S/MIME [@!RFC8551] or PGP [@!RFC3156]. This model was not widely adopted, but many
Instant Messaging systems around this time frame began to add optional end-to-end
encryption with OTR (Off The Record) [], and eventually incorporated variants of
the Double Ratchet protocol [], originally popularized by Signal.

Today, group chats are the norm, most modern instant messaging systems are end-to-end
encrypted (many by default or always) using a variant of DoubleRatchet, and the
typical feature set includes a plethora of features not included in CPIM: plain
text and rich text messaging, delivery notifications, read receipts, replies,
reactions, editing or deleting previously sent messages, and expiring messages.
Almost all systems provide a way to share files/audio/videos, and many support
calling and/or conferencing features (often using WebRTC). Some IM vendors are
implementing MLS [@!I-D.ietf-mls-protocol], a group key establishment protocol
motivated by the desire for group chat with efficient end-to-end encryption.

Unfortunately, federation of these IM systems is still rare and interoperability of
the major IM systems in almost non-existent.  It would be incredibly beneficial to
provide interoperable best practices and solutions which IM vendors can incorporate
into modern IM systems. Indeed, large customers and governments are already putting
pressure on these IM vendors. The European Union's Digital Markets Act [DMA] is a
recent dramatic example.

The goal of MIMI is to gather requirements common to many IM systems, focusing on
the most immediate needs first; to identify and evaluate existing solutions to these
specific problems; and to assemble standards and technology which already largely
exist into profiles and best current practices. Where special expertise in another
Working Group or standards body is required, that work would be delegated to the
specialty group. 

# Problem Areas

* End-to-end IM Identity: describe types of IM identifiers 
(especially in a federated domain context), problem statement, survey of existing
 relevant technologies, some concrete solutions using existing technology.







Arguably, the success of XMPP was partially due to the ease of communicating among
XMPP users in different domains with different XMPP servers, and a single
standardized address format for all XMPP users.

xmpp:

# Naming schemes

IM systems have a number of types of identifiers. Not all systems use
every type of identifier. Not every configuration of the same type of system necessarily
use the same list of identifiers.

* domain identifier
A bare domain name is often used for discovery of a specific IM service such as
`example.com` or `im.example.com`.

* handle identifier (external, friendly representation)
A handle is an identifier which represents a user or service. A handle is usually
intended for external sharing (for example it could appear on/in a paper or
electronic business card). 
IM systems could have handles which are unqualified (don't contain a domain)
or qualified (contain a domain).

Jabber/XMPP | Bare JID | juliet@example.com
SIP | Address of Record (AOR) | sip:juliet@example.com
Generic example | unqualified handle | @juliet
Generic example | qualified handle | @juliet@example.com

Unqualified handles are often prefixed with a commercial at-sign ("@").

Handles in some services mutable. For example, `@alice_smith` could
become `@alice_jones` or `@alex_smith` after change of marital status or
gender transition.

* user identifier

Many systems have an internal representation of a user or account separate from the
handle. This is especially useful when the handle is allowed to change. For example
the user identifier could be a UUID or a similar construction. 

* client/device identifier (internal representation)

Most commercial instant messaging systems allow a single user to have multiple
devices at the same time, for example a desktop computer and a phone. Usually, each
client instance of the user is represented with a separate identifier with separate
keys.

Jabber/XMPP | Fully-qualified JID | juliet/balcony@example.com
SIP | Contact Address | sip:juliet@[2001:db8::0225:96ff:fe12:3456]

Could be from a UUID, MAC address, IMEI, etc.. 



* group or or channel name (external, friendly representation)


* team identifier (less common)


* group conversation or session identifier



One user may have multiple clients (for example a mobile and a desktop client).
A handle may refer to a single user or it may redirect to multiple users.
In some systems, the user identifier is a handle. In other systems the user
identifier is an internal representation, for example a UUID. Handles may be
changed/renamed, but hopefully internal user identifiers do not.
Unqualified handles are often prefixed with a commercial at-sign ("@").

Likewise, group conversation identifiers could be internal or external
representations, whereas group names or channel names are often external
friendly representations.  Unqualified channel names are often prefixed
with a hash character ("#"). Some systems have an additional level of hierarchy
with a team identifier under which groups/channels can be organized and
authorized.

This proposal relies on URIs for naming and identifiers. All the example use
the `im:` URI scheme (defined in [@!RFC3862]), but any instant messaging scheme
is acceptable.

It is easy to imagine a loose hierarchy between these identifiers
domain to user to device. In some systems, the group chat or session itself has
a position in the hierarchy underneath the domain, the user, or the device.



# Representation of identifiers using URIs

XMPP can represent a domain, a handle (bare JID), or a device (fully qualified JID). 

While the xmpp: URI scheme was only designed to represent handles and domains, the
im: URI scheme can represent all XMPP identifiers.

* im:xmpp=example.com
* im:xmpp=juliet@example.com
* im:xmpp=juliet/balcony@example.com




Imagine the hypothetical WXYZ IM protocol with support for all our identifiers:


|---------|-----------------------------|-----------------------------------------|
| id type | unqualified form            | domain qualified form                   |
|---------|-----------------------------|-----------------------------------------|
| domain  |              -              | example.com                             |
| handle  | @alice                      | @alice@example.com                      |
| user    | BFuVxW5BfqaMEfJDc8R7Qw      | BFuVxW5BfqaMEfJDc8R7Qw@example.com      |
| device  | BFuVxW5BfqaMEfJDc8R7Qw/072b | BFuVxW5BfqaMEfJDc8R7Qw/072b@example.com |
| channel | #projectX                   | #projectX@example.com                   |
| team    | ##engineering               | ##engineering@example.com               |
| channel | ##engineering/projectX      | ##engineering/projectX@example.com      |
| session | $TII9t5viBrXiXc             | $TII9t5viBrXiXc@example.com             |
|---------|-----------------------------|-----------------------------------------|




reserved the wxyz: URI scheme



wxyz:example.com
wxyz:%40alice@example.com
wxyz:BFuVxW5BfqaMEfJDc8R7Qw/072b@example.com
wxyz:#projectX@example.com
wxyz:##engineering@example.com
wxyz:$TII9t5viBrXiX@example.com

im:wxyz=example.com
im:wxyz=%40alice@example.com
im:wxyz=BFuVxW5BfqaMEfJDc8R7Qw/072b@example.com
im:wxyz=#projectX@example.com
im:wxyz=##engineering@example.com
im:wxyz=$TII9t5viBrXiX@example.com

If there is no domain, the URI can use `local.invalid` in place of a resolvable
domain name.

im:wxyz=%40alice@local.invalid

# Different Root of Trust Approaches

Different IM applications and different users of these applications may have
different trust needs. Note that the descriptions in this section use certificates
in their examples, but nothing in this section should preclude using a different
technology which provides similar assertions.

## Centralized credential hierarchy
- In a corporate or government centralized environment: 

In this environment, end-user devices trust a centralized authority operating on
behalf of their domain (for example, a Certificate Authority), that is trusted by
all the other clients in that domain (and can be trusted by federated domains). The
centralized authority could easily be associated with a traditional Identity
Provider (IdP).

"this is Alice Smith from Engineering at XYZ, Corp"


In this model, a Certificate Authority (CA) run by or on behalf of the domain generates
certificates for one or more of the identifier types described previously. 

- The CA could issue certificates covering several identifiers at once.
- The CA could deletate certificate issuing to another identifier type (for example the
domain CA could issue user certificates and delegate to the user certificates to issue
device certificates.)
- the CA could issue one certificate per identifier type

A CA could generate a certificate for a handle or user and also

Example 1:
The CA generates one certificate for a user Alice which is used to sign Alice's profile.
The CA also generates a separate certificate for Alice's desktop client and a third
for her phone client. The client certificates are used to sign MLS KeyPackages or
Double Ratchet-style prekeys. 

Example 2:
The CA generates a single certificate per client which covers both Alice's handle and
her client identifier in the same certificate. Each of these certificates is used to
sign MLS KeyPackages or Double Ratchet-style prekeys. 

Example 3:
The CA generates a single user certificate for Alice's handle and indicates that the
user certificate can issue its own certificates.
The user certificate then generates one certificate for Alice's desktop client and
another certificate for Alice's phone client.
The client certificates are used to sign MLS KeyPackages or
Double Ratchet-style prekeys. 

validatring these certificates wuold require...

The subjectAltName is a URI type 


Regardless of the specific implementation, this model features a strong hierarchy.



What is important in all these examples is that other clients involved in a session or
group chat can validate the relevant credentials of the other participants in the
session of group chat.
 


## Web of Trust 

- specific communities web of trust (OMEMO/Matrix style)

master key
  signs user key
  user key signs device keys
  master key signs another user's user key

(check this)


## Well-known service cross signing
- Keybase style: link my IM with my github with my twitter with my webpage





# Cryptographic mechanisms to protect IM identifiers

## Certs
Mature technology
No new credential type needed
Somewhat inflexible assertion structure 
Administrative pain to get Issuer certificate


## JWT DPoP
JSON Web Tokens with Distributed Proof of Presence (DPoP JWT)


Flexible container 
Easy to implement
Poorly specified
Tokens need online verification with issuing IdP

Note that this could be combined with nested JWT claims
  

## Verifiable Credentials
Well specified
Flexible assertions: attributes (security clearance, age, country) including ZKP
Hard to implement / existing use of RDF ontologies and (optional) use of DIDs and esoteric crypto
No off the shelf mechanism for proof of possession of private key
No consensus to use for identity (watch for future usage)


## Other possible mechanisms 
Below are other mechanisms which were not investigated due to a lack of time.

- Anonymous credential schemes
	- present attributes without the long-term identity (ex: travel agent for specific team)


- other Zero-knowledge proofs






# Discovery of Keying Material

- well-known URL with query format on each domain
	- search string
		- could be handle, internal user ID, internal device ID; search by anonymous credential?
		- what type of key are you searching for?
			- keypackages
			- user keys
			- domain keys
	- searcher identity and proof of identity (optional)
	- rate limiting


* General issues of interoperable end-to-end security
Signal introduced what is now know as the Double Ratchet protocol in 20xx. Today there
are over a dozen implementations of variations of Double Ratchet. While the differences
among these variations tend to be small, the is little emphasis on interoperability.


## Profiles of security protocols to enable interoperable end-to-end encryption

Enabling strong user privacy has been a core concern of the IETF for decades, and was
the main motivation for the CPIM message format. S/MIME and PGP where proposed for
use with instant messaging systems, but never widely adopted. The first broad adoption
of end-to-end encryption in messaging was with Off The Record (OTR) [OTR] introduced
in 2004, which also included perfect forward secrecy (which protects past
communications from future compromises). As OTR was available in XMPP clients, it
was possible to use across domains.

### Protocols based on Double Ratchet

Tens of instant messaging applications implement some form of end-to-end encryption
using a protocol based on the Double Ratchet protocol. Double Ratchet was originally
referred to as Axolotl Ratchet when it was introduced in 2013 and popularized in the
Signal application.  Most applications using Double Ratchet also use [X3DH] for initial
key agreement.  However the initial setup of encryption sessions among these applications
are often incompatible. 

Double Ratchet uses a fixed ciphersuite and has no content negotiation mechanism

### Instant Messaging using Messaging Layer Security

Messaging Layer Security (MLS) is a 

 Use of MLS in the Instant Messaging Context: (ex: long-lived persistent groups)



* Content negotiation

* Content format interoperability


* Administrative setup of federation: (ex: agreement on certificates, contact information, abuse policies)


[OTR] https://otr.cypherpunks.ca/otr-wpes.pdf
[DoubleRatchet] https://signal.org/docs/specifications/doubleratchet/