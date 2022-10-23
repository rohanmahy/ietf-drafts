%%%
title = "Content Type Advertisement for Message Layer Security (MLS)"
abbrev = "MLS Content Advertisement"
ipr= "trust200902"
area = "sec"
workgroup = "MLS"
keyword = ["mls","content","advertisement","media type","mime"]

[seriesInfo]
status = "informational"
name = "Internet-Draft"
value = "draft-mahy-mls-content-adv-00"
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

This document describes a default mechanism for the advertisement of content types
and content type capabilities inside the Message Layer Security (MLS). It defines
two new extensions and a minimal framing format.

This document describes a default mechanism for the negotiation of content inside
an MLS group. It defines two new extensions to the MLS (Messaging Layer Security)
Protocol to allow for negotiation of media types exchanged among members of an
MLS group, and a minimal framing format.

{mainmatter}

# Terminology
The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", 
"SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to 
be interpreted as described in RFC 2119 [@!RFC2219].

The terms MLS client, MLS group, LeafNode, GroupContext, KeyPackage,
GroupContextExtensions Proposal, and RequiredCapabilities have
the same meanings as in the MLS protocol [@!I-D.ietf-mls-protocol].

# Introduction

MLS is a group key establishment protocol which has several applications.
As described in the MLS architecture document [@!I-D.ietf-mls-architecture],
applications need to define specific behavior of the MLS Distribution Service,
the MLS Authentication Service, and the format and negotiation of application data.
This document describes a default content advertisement mechanism recommended by the
MLS architecture specification. (The MLS protocol specification does not define or
prescribe any format for the encrypted `application_data` encoded by MLS.)

MLS includes a framework for advertising extension capabilities in LeafNodes which
are used to represent each member in an MLS group and also included in
KeyPackages. There is also an existing mechanism in which an MLS group specifies
which MLS extensions are mandatory within the group. When the membership of a group
changes, or when the policy of the group changes, it is responsibility of the
committer to insure that the membership and policies are compatible.

This document describes two extensions to MLS. The first allows MLS clients
to advertise their support for specific formats inside MLS `application_data`.
These are expressed using the extensive IANA Media Types registry (formerly
called MIME Types).  The `accepted_media_types` LeafNode extension lists the
formats a client supports inside `application_data`. The second, the
`required_media_types` GroupContext extension specifies which media types
need to be supported by all members of a particular MLS group. 
These allow clients to confirm that all members of a group can communicate.
Finally, this document defines a minimal framing format so MLS clients can signal
which media type is being sent when multiple formats are permitted in the same group.
As clients are upgraded to support new formats they can use these extensions
to detect when all members support a new or more efficient encoding, or select the
relevant format or formats to send.

Note that the usage of IANA media types in general does not imply the usage of MIME
Headers [@?RFC2045] for framing. Vendor-specific media subtypes starting with
`vnd.` can be registered with IANA without standards action as described in
[@?RFC6838].  Implementations which wish to send multiple formats in a single
application message, may be interested in the `multipart/alternative` media type
defined in [@?RFC2046] or may use or define another type with similar semantics
(see Appendix A for a container format defined using TLS Presentation Language
syntax [@!RFC8446]).

# Extension Description

This document specifies two MLS extensions of type MediaTypeList:
`accepted_media_types`, and `required_media_types`. The syntax is described using
the TLS Presentation Language [@!RFC8446].

MediaType is a TLS encoding of a single IANA media type (including top-level
type and subtype) and any of its parameters. Even if the `parameter_value`
would have required formatting as a `quoted-string` in a text encoding, only
the contents inside the `quoted-string` are included in `parameter_value`.
MediaTypeList is an ordered list of MediaType objects.

~~~ tls
struct {
    opaque parameter_name<V>;
    /* Note: parameter_value never includes the quotation marks of an
     * RFC 2045 quoted-string */
    opaque parameter_value<V>;
} Parameter;

struct {
    /* media_type is an IANA top-level media type, a "/" character,
     * and the IANA media subtype */
    opaque media_type<V>;
    
    /* a list of zero or more parameters defined for the subtype */
    Parameter parameters<V>;
} MediaType;

struct {
    MediaType media_types<V>;
} MediaTypeList;

MediaTypeList accepted_media_types;
MediaTypeList required_media_types;
~~~

Example IANA media types with optional parameters:
~~~ artwork
  image/png
  text/plain ;charset="UTF-8"
  application/json
  application/vnd.example.msgbus+cbor
~~~

For the example media type for `text/plain`, the `media_type` field
would be `text/plain`, `parameters` would contain a single Parameter
with a `parameter_name` of `charset` and a `parameter_value` of `UTF-8`.

An MLS client which implements this specification SHOULD include the
`accepted_media_types` extension in its LeafNodes, listing
all the media types it can receive. As with all other extensions, the
client also includes `accepted_media_types` in its `capabilities` field in
its LeafNodes (including LeafNodes inside its KeyPackages).

When creating a new MLS group for an application using this specification,
the group MAY include a `required_media_type` extension in the GroupContext
Extensions. As with all other extensions, the client also includes
`required_media_types` in its `capabilities` field in its LeafNodes
(including LeafNodes inside its KeyPackages). When used in a group, the client
MUST include the `required_media_types` and `accepted_media_types` extensions
in the list of extensions in RequiredCapabilities.

MLS clients SHOULD NOT add an MLS client to an MLS group with `required_media_types`
unless the MLS client advertises it can support all of the required MediaTypes.
As an exception, a client could be preconfigured to know that certain clients
support the requried types. Likewise, an MLS client is already forbidden from
issuing or committing a GroupContextExtensions Proposal which introduces required
extensions which are not supported by all members in the resulting epoch.


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

The `media_type` MAY be zero length, in which case, the media type of the
`application_content` is interpreted as the first MediaType specified in
`required_media_types`.

# IANA Considerations

This document proposes registration of two MLS Extension Types.

RFC EDITOR: Please replace XXXX throughout with the RFC number assigned to this document

## accepted_media_types MLS Extension Type

The `accepted_media_types` MLS Extension Type is used inside LeafNode objects. It
contains a MediaTypeList representing all the media types supported by the
MLS client referred to by the LeafNode.

~~~~~~~~
  Template:
  Value: 0x0005
  Name: accepted_media_types
  Message(s): This extension may appear in LeafNode objects
  Recommended: Y
  Reference: RFC XXXX
~~~~~~~~


## required_media_types MLS Extension Type

The required_media_types MLS Extension Type is used inside GroupContext objects. It
contains a MediaTypeList representing the media types which are mandatory for all
MLS members of the group to support.

~~~~~~~~
  Template:
  Value: 0x0006
  Name: required_media_types
  Message(s): This extension may appear in GroupContext objects
  Recommended: Y
  Reference: RFC XXXX
~~~~~~~~


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

# TLS Presentation Language multipart container format

In a heterogenous group of MLS clients, it is often desirable to send more than one
media type as alternatives, such that MLS clients have a choice of which media
type to render. For example, imagine an MLS group containing a set of clients
which support a common video format and a subset which only support animated GIFs.
The sender could send a `multipart/alternative` [@?RFC2046] container containing
both media types. Every client in the group could render something resembling the
media sent.

Likewise it is often desirable to send more than one media type intended to be
rendered together as in (for example a rich text document with embedded images),
which can be represented using the `multipart/mixed` [@?RFC2046] media type.

Some implementors complain that the multipart types are unnatural to use inside a
binary protocol which requires explicit lengths. Concretely, an implementation has
to scan through the entire content to construct a boundary token which is not
contained in the content.

The author does not care one whit about the specific syntax used, but presents
a multipart container format using the TLS presentation language syntax.

Note that there is a minor semantic difference between multipart/alternative and
the proposal below. In multipart/alternative, the parts are presented in
preference order by the sender. The receiver is support to render the first type
which it supports. This container includes an ordering flag. As well, even if the
flag is ordered, it is up to the IETF community to decide if it is acceptable for
the receiver to choose its "best" format to render among an ordered preference list
provided by the sender, or if the receiver must respect the ordered preference of
the sender.

``` tls
struct {
    /* a valid "Language-tag" as defined in RFC 5646 */
    opaque language_tag<1..52>;
} LanguageTag;

struct {
  ContentType content_type;
  LanguageTag content_languages<V>;
  opaque<V> body;
} Part;

enum {
  reserved(0),
  multipart_container_v1(1),
  (255)
} MultipartVersion;

enum {
  reserved(0),
  mixed(1),
  alternative(2),
  (255)
} MultipartSemantics;

enum {
  reserved(0),
  unordered(1),
  ordered(2),
  (255)
} MultipartOrdering;

struct {
    uint8 container_version;
    uint16 number_of_parts;
    MultipartSemantics semantics;
    MultipartOrdering ordering;
    Part parts<V>;
} MultipartContainer;
```