%%%
title = "Content Negotiation for Message Layer Security (MLS)"
abbrev = "MLS Content Negotiation"
ipr= "trust200902"
area = "sec"
workgroup = "MLS"
keyword = ["mls","content","negotiation","media type","mime"]

[seriesInfo]
status = "informational"
name = "Internet-Draft"
value = "draft-mahy-mls-content-neg-00"
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

This document describes a default mechanism for the negotiation of content inside
an MLS group. It defines two new extensions to the MLS (Messaging Layer Security)
Protocol to allow for negotiation of media types exchanged among members of an
MLS group, and a minimal framing format.

{mainmatter}

# Terminology
The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", 
"SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to 
be interpreted as described in RFC 2119 [@!RFC2219].

The terms MLS client, MLS group, and KeyPackage have the same meanings as in
the MLS protocol [@!I-D.ietf-mls-protocol].

# Introduction

MLS is a group key establishment protocol which has several applications.
As described in the MLS architecture document [@!I-D.ietf-mls-architecture],
applications need to define specific behavior of the MLS Distribution Service,
the MLS Authentication Service, and the format and negotiation of application data.
This document describes a default content negotiation mechanism recommended by the
MLS architecture specification. (The MLS protocol specification does not define or
prescribe any format for the encrypted `application_data` encoded by MLS.)

This document describes two extensions
to MLS which allow MLS clients to advertise their support for specific formats inside
MLS `application_data`. These are expressed using the extensive IANA Media Types
registry (formerly called MIME Types).  The `accepted_media_types` KeyPackage
Extension lists the formats a client
supports inside `application_data`, while the `required_media_types`
GroupContext Extension specifies which media types are required for a particular
MLS group. These allow clients to confirm that all members of a group can communicate.
Finally, this document defines a minimal framing format so MLS clients can signal
which media type is being sent when multiple formats are permitted in the same group.
As clients are upgraded to support new formats they can use these extensions
to detect when all members support a new or more efficient encoding, or select the
best format or formats to send.

Note that the usage of IANA media types in general does not imply the usage of MIME
Headers [@?RFC2045] for framing. Vendor-specific media subtypes starting with
`vnd.` can be registered with IANA without standards action as described in
[@?RFC6838].  Implementations which wish to send multiple formats in a single
application message, may be interested in the `multipart/alternative` media type
defined in [@?RFC2046] or may use or define another type with similar semantics.


# Extension Description

This document specifies two MLS extensions of type MediaTypeList:
`accepted_media_types`, and `required_media_types`.  The syntax is described using
the TLS Presentation Language [@!RFC8446].

MediaType is an ASCII string encoded as a TLS vector type containing a single IANA
Media Type (including the top-level type and subtype) and any of its parameters.
The formal internal structure is defined later in this section.
MediaTypeList is an ordered list of MediaType objects.

~~~ tls
  // Text representation of a single IANA-registered Media Type.
  MediaType media_type<V>;
  
  struct {
      MediaType media_types<V>;
  } MediaTypeList;
  
  MediaTypeList accepted_media_types;
  MediaTypeList required_media_types;
~~~

The internal format of the MediaType is described in the Augmented Backus-Naur
Form (ABNF) [@!RFC5234] grammar below. The `type` is the IANA top-level media type
(ex: application), `subtype` is the IANA media subtype, and `parameter` follows the
definition in [@!RFC2045] Section 5.1.  Whitespace inside an MLS MediaType is PROHIBITED.
The type, subtype, and parameter attribute names are all case-insensitive.

~~~ abnf
  MediaType := type "/" subtype *(";" parameter)
~~~


Example Media Types:
~~~ artwork
  image/png
  text/plain;charset="UTF-8"
  application/json
  application/vnd.example.msgbus+cbor
~~~

An MLS client which implements this specification SHOULD include the
`accepted_media_types` extension in its KeyPackages, listing
all the media types it can receive.

When creating a new MLS group for an application using this specification,
the group includes a `required_media_type`
extension in the GroupInfo Extensions.  When used in a group, the client
MUST include the `required_media_types` extension in the list of extensions
in RequiredCapabilities.

MLS clients SHOULD NOT add an MLS client to an MLS group with `required_media_types`
unless the MLS client advertises it can support all of the required Media
Types. As an exception, a client could be preconfigured to know that
certain clients support the mandatory types.


# Framing of application_data

When an MLS group contains the `required_media_types` GroupContext extension,
the `application_data` sent in that group is interpreted as `ApplicationFraming`
as defined below:

~~~ tls
  struct {
      MediaType media_type;
      opaque<V> application_content;
  } ApplicationFraming;
~~~

The `media_type` MAY be zero length, in which case, the Media type of the
`application_content` is the first MediaType specified in `required_media_types`.


# IANA Considerations

This document proposes registration of two MLS Extension Types.

## accepted_media_types MLS Extension Type

The `accepted_media_types` MLS Extension Type is used inside KeyPackage objects. It
contains a mediaTypeList representing all the media Types supported by the
MLS client publishing the KeyPackage.

~~~~~~~~
  Template:
  Value: 0x0005
  Name: accepted_media_types
  
  Message(s): This extension may appear in KeyPackage objects
  Recommended: Y
  Reference: RFC XXXX
~~~~~~~~

Description: list of media types supported by the MLS client advertising the KeyPackage


## required_media_types GroupContext extension

The required_media_types MLS Extension Type is used inside GroupContext objects. It
contains a mediaTypeList representing the media Types which are mandatory for all
MLS members of the group to support.

~~~~~~~~
  Template:
  Value: 0x0006
  Name: required_media_types
  
  Message(s): This extension may appear in GroupContext objects
  Recommended: Y
  Reference: RFC XXXX
~~~~~~~~

Description: list of media types which every member of the MLS group is
required to support.

# Security Considerations

The Security Considerations of MLS apply.

Use of the extensions in this document
could leak some private information both in KeyPackages and inside an MLS group.
They could be used to infer a specific implementation, platform, or even version.
Clients should consider carefully the implications in their environment of
making a list of acceptable media types available.

A client which can take over group administration could prevent members from
joining or sending messages in an established group, by requiring a list of
required media types which the attacker knows is unsupported. This attack is
not especially helpful, as taking over group administration can have more
disruptive effects.

{backmatter}







