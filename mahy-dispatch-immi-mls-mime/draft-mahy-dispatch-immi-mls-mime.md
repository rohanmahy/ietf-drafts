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

This document defines two new extensions to the MLS (Message Layer Security) Protocol 
to allow for negotiation of MIME types exchanged among members of an MLS group.

{mainmatter}

# Terminology
The terms MLS client, MLS group, ...

# Motivation

MLS [MLS] is a group key establishment protocol motivated by the desire for group
chat with efficient end-to-end encryption. While one of the motivations of 
MLS is interoperable standards-based secure messaging, the MLS protocol does
not define or prescribe any format for the encrypted "application messages"
encoded by MLS. This document describes two extensions to MLS which allow 
MLS clients to advertise their supported MIME types, and to specify which
MIME types are required for a particular MLS group. These allow clients to
discover MLS groups with an interoperable and extensible set of content
types.

A companion document describes a specific profile for interoperable instant
messaging body types.


# Extension Description

MimeType is the ASCII string encoded as a TLS vector type containing a 
single MIME type and any of its parameters.

~~~~~~~~
image/png
text/plain;charset="UTF-8"
~~~~~~~~

MimeTypeList is an ordered list of MimeType objects.

~~~~~~~~
struct {
    MimeType mime_types<V>
} MimeTypeList

~~~~~~~~


## Group creation

Add the extension to the list of extensions in RequiredCapabilities. 


MLS clients MUST NOT add to an MLS client to an MLS group

An MLS group which contains



SHOULD NOT include in a Proposal 
SHOULD NOT Commit 

MUST NOT join 


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

{backmatter}






