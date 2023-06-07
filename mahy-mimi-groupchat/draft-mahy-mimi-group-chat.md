%%%
title = "Group Chat Framework for More Instant Messaging Interoperability (MIMI)"
abbrev = "MIMI Group Chat"
ipr= "trust200902"
area = "art"
workgroup = "MIMI"
keyword = ["group chat","muc","multiuser chat", "multi-user chat", "mls", "xep-45"]

[seriesInfo]
status = "informational"
name = "Internet-Draft"
value = "draft-mahy-mimi-group-chat-00"
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

This document describes a group instant messaging ("group chat") semantic framework for
the More Instant Messaging Interoperability (MIMI) Working Group. It describes
several properties and policy options which can be combined to model a wide
range of chat and multimedia conference types. It also describes how to build
these options as an overlay to Messaging Layer Security (MLS) groups. 
This document borrows heavily from the semantics of the Multi-User Chat (MUC)
extension of the Extensible Messaging and Presence Protocol (XMPP).

{mainmatter}

# Terminology

## General Terms

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", 
"SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to 
be interpreted as described in RFC 2119 [@!RFC2219].

The terms MLS client, MLS group, LeafNode, GroupContext, KeyPackage,
GroupContextExtensions Proposal, Credential, CredentialType, and
RequiredCapabilities have the same meanings as in the Messaging Layer Security (MLS)
protocol [@!I-D.ietf-mls-protocol].

Room:
: A room, also known as a chat room or group chat, is a virtual space users
figuratively enter in order to participate in text-based conferencing. 
When used with MLS it typically has a 1:1 relationship with an MLS group.

Room ID:
: An identifier which uniquely identifies a room.

User: 
: A single human user or automated agent (ex: chat bot) with a distinct identifiable
representation in a room.

User ID:
: An internal identifier which uniquely identifies a user.

Nickname:
: The identifier by which a user is referred inside a room. Depending on the
context it may be a display name, handle, pseudonym, or temporary identifier.
The nickname in one room need not correlate with the nickname for the same user
in a different room. 

Client: 
: A instant messaging agent instance associated with a specific user account on a
specific device. For example, the mobile phone instance used by the user 
alice@example.com.

Client ID:
: An internal identifier which uniquely identifies one client/device instance
of one user account.

Multi-device support:
: A room which supports users each of which can have multiple client instances
(ex: a Desktop client and mobile phone).

User-Affiliation:
: A long-lived association reflecting the privilege level of a user in a room.
Possible values are owner, admin, member, none, or outcast.

Client-Role:
: A temporary position of a client in a room, distinct from the
User-Affiliation of the corresponding user. Possible values are moderator,
participant, visitor, and none.

## 

Ban:
: To remove a user from a room such that the user is not allowed to re-enter
the room (until and unless the ban has been removed). A banned user has a
user-affiliation of "outcast".

Kick:
: To temporarily remove a participant or visitor from a room. The user is allowed
to re-enter the room at any time. 

Silence: 
: To remove the permission to send messages in a room.

Voice (verb):
: To grant the permission to send messages in a room.

Voice (noun):
: The privilege to send messages in a room.


## Client-Role and User-Affiliation Values


## Room Capabilities

The following room capabilities are included from MUC.

Hidden vs. Public:
: A public room is discoverable/searchable. A hidden room is not.

Members-Only vs. Open:
: An open room can be joined by any non-banned user. A members-only room can
only be joined by a user in the member list.

Moderated vs. Unmoderated:
: An an unmoderated room, any user in the room can send messages to the room.
In a moderated room, only users who have "voice" can send messages to the room.

Not a part of MUC.

Fixed-Membership vs. Flexible-Membership:
: Fixed-membership rooms have the list of occupants specified when they are
created. Other users cannot be added. Occupants cannot leave or be removed.
The most common case of a fixed-membership room is a 1:1 conversation. Only a
single fixed-membership room can exist for any unique set of occupants. By
contrast, in a Flexible-Membership room, authorized users can add or remove users
to the room.

Not included from MUC.

Password-Protected vs. Unsecured:
: foo

Persistent vs. Temporary:
: bar

Non-anonymous vs. Semi-anonymous:

## Permissions

Logging Policy

History Policy

Bot Policy

Direct Messaging


# Introduction



# Example Use



# Extension Description



# IANA Considerations

This document proposes registration of a new media type


# Security Considerations




{backmatter}
