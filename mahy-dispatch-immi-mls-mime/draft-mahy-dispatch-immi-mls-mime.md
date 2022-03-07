%%%
title = "Inside MLS Message Interop (IMMI) MIME type extensions"
abbrev = "MLS MIME type Extensions"
ipr= "trust200902"
area = "art"
workgroup = "dispatch"
keyword = ["mls","mime"]

[seriesInfo]
status = "informational"
name = "Internet-Draft"
value = "draft-mahy-dispatch-immi-mls-mime"
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

This document defines two new extensions to the MLS (Messaging Layer Security) Protocol
to allow for negotiation of MIME types exchanged among members of an MLS group.

{mainmatter}

# Terminology
The terms MLS client, MLS group, and KeyPackage have the same meanings as in
the MLS protocol [@!I-D.ietf-mls-protocol].

# Motivation

MLS is a group key establishment protocol motivated
by the desire for group chat with efficient end-to-end encryption. While one
of the motivations of MLS is interoperable standards-based secure messaging,
the MLS protocol does not define or prescribe any format for the encrypted
"application messages" encoded by MLS. This document describes two extensions
to MLS which allow MLS clients to advertise their supported MIME types, and
to specify which MIME types are required for a particular MLS group. These
allow clients to discover MLS groups with an interoperable and extensible set
of content types.

A companion document [@?I-D.mahy-dispatch-immi-content] describes a specific
profile for interoperable instant messaging body types.


# Extension Description

This document specifies two MLS extensions of type MimeTypeList:
`accepted_mime_types`, and `required_mime_types`.

MimeType is the ASCII string encoded as a TLS vector type containing a
single MIME type and any of its parameters.

MimeTypeList is an ordered list of MimeType objects.

~~~~~~~~
// Text string representation of a single IANA registered MIME Type.
MimeType mime_type<V>

struct {
    MimeType mime_types<V>
} MimeTypeList

~~~~~~~~

Example MIME Types:
~~~~~~~~
image/png
text/plain;charset="UTF-8"
~~~~~~~~

An MLS client which implements this specification SHOULD include the
`accepted_mime_types` extensions in its KeyPackages, listing
all the MIME types it can receive.

When creating a new MLS group, the group MAY include a `required_mime_type`
extension in the group Extensions.  When used in a group, the client
MUST include the `required_mime_types` extension in the list of extensions
in RequiredCapabilities.

MLS clients SHOULD NOT add an MLS client to an MLS group with `required_mime_types`
unless the MLS client advertises it can support all of the required MIME
Types. As an exception, a client could be preconfigured to know that
certain clients support the mandatory types.

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

# Security Considerations

The Security Considerations of MLS apply.

Use of the extensions in this document
could leak some private information both in KeyPackages and inside an MLS group.
They
could be used to infer a specific implementation, platform, or even version.
Clients should consider carefully the implications in their environment of
making a list of acceptable MIME types available.

A client which can take over group administration could prevent members from
joining or sending messages in an established group, by requiring a list of
required MIME types which the attacker knows is unsupported. This attack is
not especially helpful, as taking over group administration can have more
disruptive effects.

{backmatter}

<reference anchor="I-D.mahy-dispatch-immi-content">
   <front>
      <title>Inside MLS Message Interop (IMMI) instant message content</title>
      <author fullname="Rohan Mahy">
	 <organization>Wire</organization>
      </author>
      <date month="March" day="7" year="2022" />
   </front>
   <seriesInfo name="Internet-Draft" value="draft-mahy-dispatch-immi-content-00" />
   <format type="TXT" target="https://www.ietf.org/archive/id/draft-mahy-dispatch-immi-content-00.txt" />
</reference>





