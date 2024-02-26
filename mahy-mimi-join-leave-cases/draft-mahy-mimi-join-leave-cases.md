%%%
title = "MIMI Use Cases for Joining and Leaving an MLS group"
abbrev = "MIMI Joining/Leaving Use Cases"
ipr= "trust200902"
area = "sec"
workgroup = "MLS"
keyword = ["mimi","mls","join","leave","membership"]

[seriesInfo]
status = "informational"
name = "Internet-Draft"
value = "draft-mahy-mimi-join-leave-cases-00"
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

This document describes Instant Messaging use cases involving clients
joining and leaving an Message Layer Security (MLS) group, to be able
to map MLS actions into authorization or message transport protocol
semantics.

{mainmatter}

# Terminology
The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", 
"SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to 
be interpreted as described in RFC 2119 [@!RFC2219].

Client

Device

User


The terms MLS client, MLS group, LeafNode, KeyPackage, GroupContext,
GroupContextExtensions Proposal, Credential, CredentialType, and
RequiredCapabilities have the same meanings as in the MLS
protocol [@!I-D.ietf-mls-protocol].

# Introduction




# Example Use

Consider an MLS group containing MLS clients from three domains: alpha.example,
beta.example, and charlie.example. All three use compatible MLS-based instant
messaging services which are federated [I-D.ietf-mls-federation].

- alpha.example is a very large company or national government with their own root
certificate authority which is already present in most operating systems.
- beta.example is a large company which uses certificate authority yankee.example.
- charlie.example is a small organization whose service is hosted by a cloud
provider cirrus.example. Their certificates (both charlie and cirrus) are issued
by the certificate authority zulu.example.

Alice is a user on alpha.example, and creates a private federated MLS group,
inviting Andy (from alpha.example), Bob and Betty (from beta.example), and
Cathy and Chuck (from charlie.example). Every client in the group would like
to verify the identities of the other clients. If alpha.example is compromised,
we don't want an attacker to be able to impersonate Bob, Betty, Cathy, or Chuck
without detection. Likewise if yankee.example is compromised, we don't want an
attacker to be able to impersonate Alice, Andy, Cathy, or Chuck without detection.

# Use Cases initiated by MLS

## Commit with Add

Existing member adds an MLS client to group 

- Claim a KeyPackage for each of the clients to be added.
- Make sure all the MLS clients have compatible capabilities and policies
- Send a Commit with embedded Add proposals to the existing members of the group.
- Send a Welcome message to the newly added clients. A single Welcome per Commit
is typical, but the adding client may spread the notifications across multiple
unique Welcome messages.

## External Connit (Join)

### New client of an existing user joins all the user's groups




### Client adds itself to an open group

## External Commit (Join) with additional PSK

Client uses a one-time code to join a group

## External Proposal (Join)

Perhaps to join a group that is not active yet (a group for a large event
which starts at a fixed time.)

## Client asks to join 

## User replaces a client with a new client




# Use Cases initiated by external protocol or user action




# IANA Considerations

This document has no requirements for IANA.


# Security Considerations

TBC


{backmatter}
