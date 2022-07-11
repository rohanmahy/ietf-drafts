%%%
title = "More Instant Messaging Interoperability (MIMI) problem outline"
abbrev = "MIMI problem outline"
ipr= "trust200902"
area = "art"
workgroup = "MIMI BoF"
keyword = ["mimi","immi"]

[seriesInfo]
status = "informational"
name = "Internet-Draft"
value = "draft-mahy-mimi-problem-outline-00"
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

Instant Messaging interoperability requirements have changed dramatically since
the IETF last activity in the space. This document presents an outline of problems
that need to be addressed to make possible interoperability of modern Instant
Messaging systems. The goal of this problem outline is to point at more detailed
drafts which spawn discussion and eventually spur the IETF standards process.

{mainmatter}

# Overview

The IETF has been working on Instant Messaging Interoperability since the late 1990s. 
At the time, different groups within IETF proposed separate protocol suites (SIMPLE,
APEX, and XMPP) because the community could not come to consensus on a single protocol
(arguably due to a lack of consensus on the additional requirements which made these
proposals unique). In the interests of interoperability, the IMPP Working Group
developed a general framework for interoperability [@?RFC3860], and the Common Presence
and Instant Messaging (CPIM) Message Format [@?RFC3862], an interoperability format that
could pass through gateways among these protocols, even when end-to-end encrypted.  

The CPIM model assumed standalone encryption of each message using a protocol such
as S/MIME [@?RFC8551] or PGP [@?RFC3156]. This model was not widely adopted, but many
Instant Messaging systems around this time frame began to add optional end-to-end
encryption with OTR (Off The Record) [@?OTR], and eventually incorporated variants of
the Double Ratchet protocol [@?DoubleRatchet], originally popularized by Signal.

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

Instant Messaging interoperability requirements have changed dramatically since
the IETF last activity in the space. This document presents an outline of problems
that need to be addressed to make possible interoperability of modern Instant
Messaging systems. The goal of this problem outline is to point at more detailed
drafts which spur discussion and eventually spur the IETF standards process.

The larger goals of MIMI (More Instant Messaging Interoperability) are to start
discussion; gather requirements common to many IM systems, focusing on
the most immediate needs first; develop requirements and frameworks; and
eventually to identify and evaluate existing solutions to these
specific problems; and to assemble standards and technology which already largely
exist into profiles and best current practices. Where special expertise in another
Working Group or standards body is required, that work would be delegated to the
specialty group. 


# Interoperability Problem Areas

## Naming schemes

IM systems have a number of identifiers with different characteristics which are
relevant for interoperability. 

- Domain identitifer
- Handle identifier
- User or Account identifier
- Client or Device identifier
- Group Chat or Channel identifier
- Group, Conversation, or Session identifiers
- Team or Workspace identifier

These identifiers are discussed in detail in 
[@!I-D.mahy-mimi-identity] as well as how they can be represented using URIs.

## End-to-end IM Identity 

The largest and most widely deployed Instant Messaging (IM) systems support
end-to-end message encryption using a variant of the Double 
Ratchet protocol [@?DoubleRatchet] popularized by Signal and the companion X3DH [@?X3DH]
key agreement protocol. Many vendors are also keen to support the Message Layer Security
(MLS) protocol [@?I-D.ietf-mls-protocol] and architecture [@?I-D.ietf-mls-architecture].
These protocols provide confidentiality
of sessions (with Double Ratchet) and groups (with MLS) once the participants in 
a conversation have been identified. However, the current state of most systems
require the end user to manually verify key fingerprints or blindly trust their
instant messaging service not to add and remove participants from their 
conversations. This problem is exacerbated when these systems federate or try to
interoperate.

This problem space is explored in [@!I-D.mahy-mimi-identity].

## Discovery

### Domain-level service discovery

The discovery of IM services using DNS SRV records is described in [@!RFC3861].

### User discovery and discovery of device keying material

TBC.

One vendor mentioned a strawperson outline for user discovery:

- well-known URL with query format on each domain
	- search string
		- could be handle, internal user ID, internal device ID; search by anonymous credential?
		- what type of key are you searching for?
			- keypackages
			- user keys
			- domain keys
	- searcher identity and proof of identity (optional)
	- rate limiting



## Profiles of security protocols to facilitate interoperable end-to-end encryption

Enabling strong user privacy has been a core concern of the IETF for decades, and was
the main motivation for the CPIM message format. S/MIME and PGP where proposed for
use with instant messaging systems, but never widely adopted. The first broad adoption
of end-to-end encryption in messaging was with Off The Record (OTR) [@?OTR] introduced
in 2004, which also included perfect forward secrecy (which protects past
communications from future compromises). As OTR was available in XMPP clients, it
was possible to use across domains.

### Protocols based on Double Ratchet

Signal introduced what is now know as the Double Ratchet protocol in 2013. Today there
are over a dozen implementations of variations of Double Ratchet. While the differences
among these variations tend to be small, there is little emphasis on interoperability.

Tens of instant messaging applications implement some form of end-to-end encryption
using a protocol based on the Double Ratchet protocol. Double Ratchet was originally
referred to as Axolotl Ratchet when it was introduced in 2013 and popularized in the
Signal application.  Most applications using Double Ratchet also use [@?X3DH] for initial
key agreement.  However the initial setup of encryption sessions among these applications
are often incompatible. 

Most implementations of Double Ratchet use a fixed ciphersuite and have no 
content negotiation mechanism.

### Instant Messaging using Messaging Layer Security

Messaging Layer Security (MLS) [@?I-D.ietf-mls-protocol] is a group key agreement
protocol with application message encryption and authentication. As described in
the MLS architecture [@?I-D.ietf-mls-architecture], the protocol does not define
the specific behavior of the Distribution Service (DS) or the Authentication Service
(AS). 

Some specific issues involving the use of MLS is a multi-domain Instant Messaging
context are discussed in the MLS federation draft [@?I-D.ietf-mls-federation].

More documentation on the use of MLS in the Instant Messaging Context: 
(ex: long-lived persistent groups) would be invaluable.


## Content negotiation

Content negotiation is protocol specific. In MLS, the requirements for content
negotiation are discussed in the MLS architecture document [@?I-D.ietf-mls-architecture]
and a specific content negotiation mechanism that can be used within MLS is described
in [@?I-D.mahy-mls-content-neg].

## Content format interoperability

The expectation of basic or common features in IM systems has grown. Below is a list
of some features commonly found in most IM group chat systems:

* plain text and rich text messaging
* delivery notifications
* read receipts
* replies
* reactions
* edit or delete previously sent messages
* expiring messages
* knock / ping
* shared files/audio/videos
* calling / conferencing

Once messages are encrypted end-to-end there is no further opportunity for content
negotiation. 
Exploring requirements, semantics, and an example common format for messages,
which would allow proprietary messages or
extensions to be delivered in parallel to the same users is described in
[@!I-D.mahy-mimi-content]. It discusses all of the features above except for
calling and conferencing.


## Administrative setup of federation

(ex: agreement on certificates, contact information, abuse policies). 
TBC.

## Authorization features

Is it necessary to standardize IM application authorization features such as 
moderation roles?

# IANA Considerations

This document requires no action of IANA.

# Security Consideration

TBC.

{backmatter}

<reference anchor="I-D.mahy-mimi-identity">
   <front>
      <title>More Instant Messaging Interoperability (MIMI) Identity Concepts</title>
      <author fullname="Rohan Mahy">
	 <organization>Wire</organization>
      </author>
      <date month="July" day="11" year="2022" />
   </front>
   <seriesInfo name="Internet-Draft" value="draft-mahy-mimi-identity-00" />
   <format type="TXT" target="https://www.ietf.org/archive/id/draft-mahy-mimi-identity-00.txt" />
   <format type="HTML" target="https://www.ietf.org/archive/id/draft-mahy-mimi-identity-00.html" />
</reference>

<reference anchor="OTR" target="https://otr.cypherpunks.ca/otr-wpes.pdf">
  <front>
    <title>Off-the-Record Communication, or, Why Not To Use PGP</title>
    <author fullname="Nikita Borisov">
      <organization>UC Berkeley</organization>
    </author>
    <author fullname="Ian Goldberg">
      <organization></organization>
    </author>
    <author fullname="Eric Brewer">
      <organization>UC Berkeley</organization>
    </author>
    <date month="October" day="28" year="2004"/>
  </front>
</reference>

<reference anchor="DoubleRatchet" target="https://signal.org/docs/specifications/doubleratchet/">
  <front>
    <title>The Double Ratchet Algorithm</title>
    <author fullname="Trevor Perrin">
      <organization>Signal</organization>
    </author>
    <author fullname="Moxie Marlinspike">
      <organization>Signal</organization>
    </author>
    <date month="November" day="20" year="2016"/>
  </front>
</reference>

<reference anchor="X3DH" target="https://signal.org/docs/specifications/x3dh/">
  <front>
    <title>The X3DH Key Agreement Protocol</title>
    <author fullname="Moxie Marlinspike">
      <organization>Signal</organization>
    </author>
    <author fullname="Trevor Perrin">
      <organization>Signal</organization>
    </author>
    <date month="November" day="4" year="2016"/>
  </front>
</reference>
