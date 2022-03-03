%%%
title = "Inside MLS Message Interop (IMMI) instant message content"
abbrev = "Inside MLS IM content"
ipr= "trust200902"
area = "art"
workgroup = "dispatch"
keyword = ["mls","mime"]

[seriesInfo]
status = "informational"
name = "Internet-Draft"
value = "draft-mahy-dispatch-immi-content"
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

This document defines a profile intended for instant messaging interoperability
of messages end-to-end encrypted inside the MLS (Message Layer Security)
Protocol. It adapts prior work (CPIM) to work well in the MLS context.

{mainmatter}



# Terminology
The terms MLS client, MLS group, ...

# Motivation

MLS is a group key establishment protocol motivated by the desire for group
chat with efficient end-to-end encryption. While one of the motivations of 
MLS is interoperable standards-based secure messaging, the MLS protocol does
not define or prescribe any format for the encrypted "application messages"
encoded by MLS.  The development of MLS was strongly motivated by the needs
of Instant Messaging (IM) systems. 

End-to-end encrypted instant messaging was also a motivator for the Common
Protocol for Instant Messaging (CPIM) [@RFC3862], however the model used at the time
assumed standalone encryption of each message using a protocol such as S/MIME
or PGP to interoperate between IM protocols such as SIMPLE and XMPP.  For a 
variety of practical reasons, interoperable end-to-end encryption between IM
systems was never deployed commercially.

There are now several vendors prepared to implement MLS. In order to enable
interoperable messaging conveyed "inside" MLS application messages, some 
additional specification and some minor changes are required. Also, the 
expectation of what constitutes basic features common across multiple IM
systems has grown. It would be beneficial to provide an interoperable format
for these additional features as well.  Most of these features can be 
implemented using a profile which describes how to use already-defined
URIs, message headers, and MIME types. 

This proposal assumes that MLS clients can advertise MIME types they support
and that MLS clients can determine what MIME types of required to join a
specific MLS group. A companion proposal defines two MLS extensions which
meets this requirement. It would allow implementations to define groups
with different MIME type requirements and it would allow MLS clients to send
extended or proprietary messages that would be interpreted by some members
of the group while assuring that an interoperable end-to-end encrypted
baseline is available to all members, even when the group spans multiple
systems or vendors.

Below is a list of some features commonly found in IM group chat systems:

* plain text and rich text messaging
* delivery notifications
* read receipts
* replies
* reactions
* edit or delete previously sent messages
* expiring messages
* knock / ping
* hide messages
* shared files/audio/videos
* calling / conferencing

# Overview

This proposal relies on

URIs for naming and identifiers

Types of identifiers
group chat conversation identifier
handle identifier
user identifier
MLS client identifier

One user may have multiple clients (for example a mobile and a desktop client)

Negotiation of MIME types


Specific Message headers

Specific usage of MIME header


We assume that an MLS group is already established and that either out-of-band
or using the MLS protocol or MLS extensions that the following is known to every
member of the group:

* The membership of the group (via MLS).
* The identity of any MLS client which sends an application message (via MLS).
* The MLS group ID (via MLS)
* The human readable name(s) of the MLS group, if any (out-of-band or extension).
* Which MIME types are mandatory to implement (proposed extension).
* For each member, the MIME types each supports (proposed extension).

For all messages the message header equivalent of To and Sender fields is already
known and is therefore redundant. Every message contains a message/cpim header which 
includes the From, DateTime, and Message-ID fields.

Messages send to an MLS group are delivered to every member of the group active during
the epoch in which the message was sent.


 
# Example
## Original Message

In this example, Alice Smith sends a rich-text (Markdown) message to the Engineering
Team MLS group. The following values are implied as if headers were present:

* Implied Sender header from MLS sender:
  <im:3b52249d-68f9-45ce-8bf5-c799f3cad7ec-0003@example.com>
* Implied To header from MLS group: "Engineering Team" 
  <im:9dc867ca-3a01-4385-bb69-1573601c3c0c@example.com>
~~~~~~~
Content-type: message/cpim

From: <im:alice-smith@example.com>
DateTime: 2022-02-08T22:13:45-00:00
Message-ID: <28fd19857ad7@example.com>

Content-Type: text/markdown;charset=utf-8

Hi everyone, we just shipped release 2.0. __Good work__!
~~~~~~~


### Reply

A reply message looks the similar, but contains an In-Reply-To CPIM header with
the ID of the original message. The implied To header is the same all 
example messages in this section. The implied sender from MLS is:

* <im:3402b3091-5b5c-48c8-9806-5dcb899bf9d2-0006@example.com>

~~~~~~~
Content-type: message/cpim

From: <im:bob-jones@example.com>
DateTime: 2022-02-08T22:13:57-00:00
Message-ID: <e701beee59f9@example.com>
In-Reply-To: <28fd19857ad7@example.com>

Content-Type: text/markdown;charset=utf-8

Right on! _Congratulations_ y'all!
~~~~~~~

### Reaction 

A reaction, uses the reaction Content-Disposition token defined in [@RFC9078]. 
This Content-Disposition token indicates that the intended disposition of the
contents of the message is a reaction. 

The content in the sample message is a single Unicode heart character (U+2665).
Discovering the range of characters each implementation could render as a
reaction can occur out-of-band and is not within the scope of this proposal. 
However, an implementation which receives a reaction character string it 
does not recognize could render the reaction as a reply, possibly prefixing
with a localized string such as "Reaction: ".  Note that a reaction could
theoretically even be another media type (ex: image, audio, or video), although
not currently implemented in major instant messaging systems.

The implied sender in this message is:

* <im:f9a2309d-1a29-4c4b-bc19-74886af03ea1-0002@example.com>

~~~~~~~
Content-type: message/cpim

From: <im:cathy-washington@example.com>
DateTime: 2022-02-08T22:13:57-00:00
Message-ID: <1a771ca1d84f@example.com>
In-Reply-To: <28fd19857ad7@example.com>

Content-Type: text/plain;charset=utf-8
Content-Disposition: reaction

♥
~~~~~~~

### Mentions

In instant messaging systems and social media, a mention allows special
formatting and behavior when a name, handle, or tag associated with a
known group is encountered, often when prefixed with a commercial-at (@)
character for mentions of users or a hash (#) character for groups or tags. 



Mention (via Markdown)

~~~~~~~
Content-type: message/cpim

From: <wireapp:cathy-washington@example.com;type=handle>
Sender: <wireapp:f9a2309d-1a29-4c4b-bc19-74886af03ea1-00000002@example.com;type=client>
To: "Engineering Team" <wireapp:9dc867ca-3a01-4385-bb69-1573601c3c0c@example.com;type=conversation>
DateTime: 2022-02-08T22:14:03-00:00
Message-ID: <4dcab7711a77@example.com>
In-Reply-To: <28fd19857ad7@example.com>

Content-Type: text/markdown;charset=utf-8

Kudos to [@Alice Smith](im:alice-smith@example.com)
for making this happen!
~~~~~~~

Mention (via HTML)

~~~~~~~
Content-type: message/cpim

From: <wireapp:cathy-washington@example.com;type=handle>
Sender: <wireapp:f9a2309d-1a29-4c4b-bc19-74886af03ea1-00000002@example.com;type=client>
To: "Engineering Team" <wireapp:9dc867ca-3a01-4385-bb69-1573601c3c0c@example.com;type=conversation>
DateTime: 2022-02-08T22:14:03-00:00
Message-ID: <4dcab7711a77@example.com>
In-Reply-To: <28fd19857ad7@example.com>

Content-Type: text/html;charset=utf-8

<p>Kudos to <a href="wireapp:alice-smith@example.com;type=handle">@Alice
Smith</a> for making this happen!</p>
~~~~~~~

### Edit

The Supersedes header is not often used, but it semantics are clear: the
message included in the body is a replacement for the message with the 
superseded message ID.

~~~~~~~
Supersedes: <msgId>;action={replace|hide|delete}
~~~~~~~

### Delete

In IM systems a delete means that the author of a specific message has 
retracted the message, regardless if other users have read the message
or not. A delete uses the Supersedes header with an empty body
~~~~~~~
Supersedes: <msgId>;action={replace|hide|delete}
~~~~~~~

Expiring

Expiring message are sent simply by including an Expires header in the CPIM message body.  
~~~~~~~
Expires: 
~~~~~~~

Knock

A knock or ping is represented as a text/plan body containing a single CRLF
with the Content-Disposition MIME header of alert

~~~~~~~
Content-Disposition: alert
~~~~~~~

## Read Receipt

Existing methods of delivery notification and read receipts assume action
by intermediary and feature many limitations. Often only a single
notification is allowed per message.

The format below for message/im-disposition-notification has one 
IM-Disposition line per message, with the disposition of the original 
message in a parameter. 



~~~~~~~
Content-type: message/cpim

From: <wireapp:bob-jonesh@example.com;type=handle>
Sender: <wireapp:3402b3091-5b5c-48c8-9806-5dcb899bf9d2-00000006@example.com;type=client>
To: "Engineering Team" <wireapp:9dc867ca-3a01-4385-bb69-1573601c3c0c@example.com;type=conversation>
DateTime: 2022-02-08T22:13:57-00:00
Message-ID: <1a294c4bbc19@example.com>

Content-type: message/im-disposition-notification

IM-Disposition: <9dc867ca9857@example.com>;disposition=read
IM-Disposition: <28fd19857ad7@example.com>;disposition=read|error|delivered|not-delivered|deleted|

~~~~~~~


Hide
~~~~~~~
Supersedes: <msgId>;action={replace|hide|delete}
~~~~~~~

Assets

~~~~~~~
Content-Type: message/external-body; access-type="URL";
 URL="https://example.com/storage/bigfile.m4v";
 size=
~~~~~~~


Call
~~~~~~~
Content-Type: message/external-body; access-type="URL";
 URL="https://example.com/join/12345"
Content-Disposition: session
~~~~~~~

## Delivery notification
MLS AppAck

~~~~~~~
Content-type: message/cpim

From: <wireapp:bob-jonesh@example.com;type=handle>
Sender: <wireapp:3402b3091-5b5c-48c8-9806-5dcb899bf9d2-00000006@example.com;type=client>
To: "Engineering Team" <wireapp:9dc867ca-3a01-4385-bb69-1573601c3c0c@example.com;type=conversation>
DateTime: 2022-02-08T22:13:57-00:00
Message-ID: <1a294c4bbc19@example.com>

Content-type: message/im-disposition-notification

Final-Recipient: <wireapp:3402b3091-5b5c-48c8-9806-5dcb899bf9d2-00000006@example.com;type=client>
Original-Message-ID: <28fd19857ad7@example.com>
IM-Disposition: read
~~~~~~~

# CPIM profile





# IANA Considerations

This document proposes registration of two MLS Extension Types. 

## accepted_mime_types MLS Extension Type

The accepted_mime_types MLS Extension Type is used inside KeyPackage objects. It
contains a MimeTypeList representing all the MIME Types supported by the 
MLS client publishing the KeyPackage.

~~~~~~~~
Template:
Value: 0x0005
Name: accepted_mime_types

Message(s): This extension may appear in KeyPackage objects
Recommended: Y
Reference: RFC XXXX
~~~~~~~~

Description: list of MIME types supported by the MLS client advertising the KeyPackage


## required_mime_types GroupContext extension

The required_mime_types MLS Extension Type is used inside GroupContext objects. It
contains a MimeTypeList representing the MIME Types which are mandatory for all
MLS members of the group to support.

~~~~~~~~
Template:
Value: 0x0006
Name: required_mime_types

Message(s): This extension may appear in GroupContext objects
Recommended: Y
Reference: RFC XXXX
~~~~~~~~

Description: list of MIME types which every member of the MLS group is 
required to support.




draft-mahy-dispatch-immi-content
draft-mahy-dispatch-immi-mls-mime


Inside MLS Message Interop (IMMI)

draft-mahy-dispatch-immi-extensions
draft-mahy-dispatch-immi-im-profile


profile of CPIM with the following fields:
implied from MLS message
To: the MLS group
Sender: the MLS client 



message/cpim
application/im-read-notifications
message/vnd.wireapp-message

application/json
text/plain
text/markdown
text/xml
text/html
image/jpeg
image/png
image/gif
audio/
audio/
video/mp4
video/vp8



required
From: the identity of message sender. for example im:rohan@example.com
this identity could be pseudonymous or anonymous if the group policy allows
DateTime:
Message-ID:
Content-type:

recommended
In-Reply-To:
Content-Disposition:

required MIME types
message/cpim
multipart/alternative
multipart/mixed
multipart/parallel
text/plain

recommended
text/markdown
text/html
message/im-disposition-notification
image/jpeg
image/png

IM-Disposition: <28fd19857ad7@example.com>;disposition=read





as discussed in [mls-federation]




Problem Discussion

Using MLS to negotiate "inner" MIME type

Naming conventions and scoping

{backmatter}





