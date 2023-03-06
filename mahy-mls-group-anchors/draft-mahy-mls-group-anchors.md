%%%
title = "Per-group Credential Anchors for Message Layer Security (MLS)"
abbrev = "MLS Group Credential Anchors"
ipr= "trust200902"
area = "sec"
workgroup = "MLS"
keyword = ["mls","credential anchor","trust root","groupcontext","per-group"]

[seriesInfo]
status = "informational"
name = "Internet-Draft"
value = "draft-mahy-mls-group-anchors-00"
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

This document describes a Message Layer Security (MLS) GroupContext extension
to restrict the set of trust anchors used for identity validation in MLS
groups. It is useful in federated or interoperability environments to
allow a specific federated domain to assert identities for its own identifiers
but not for the identifiers of other domains.

{mainmatter}

# Terminology
The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", 
"SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to 
be interpreted as described in RFC 2119 [@!RFC2219].

The terms MLS client, MLS group, LeafNode, GroupContext, KeyPackage,
GroupContextExtensions Proposal, Credential, CredentialType, and
RequiredCapabilities have the same meanings as in the MLS
protocol [@!I-D.ietf-mls-protocol].

# Introduction

A typical desktop or mobile operating system may have hundreds of root
certificates configured. Not all of these certificates are appropriate
to make identity assertions about every domain which participates in a
federated MLS group. The members of a federated group should be able to
restrict the specific trust anchors expected, on a per-domain basis.
The root of the trust anchor still needs to be among the operating system
or application trusted root certificates. 

In addition, this extension allows the domain validation to be restricted
to an intermediary certificate which is anchored in one of the trusted root
certificates. For example, the domain `example.com` might use the Certificate
Authority "Large Commercial Certificate Authority LLC" as the root for its domain
certificates, but only the intermediate certificate `ca.messaging.example.com`
actually makes assertions about MLS identities for that domain.

While this extension initially only specifies behavior for X.509 certificates
and the `x509` credential type in MLS, other credential types with strong
cryptographic verification, such as VerifiableCredentials, could extend this
extension to include the relevant notion of trust anchors. 

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

With this extension, the clients in an MLS group maintain a list of identity domains
and each of their corresponding trust anchors. This does not replace the operating system
or application trusted root certificates, it just associates a specific domain with
a specific trust anchor. 

# Extension Description

This document specifies a GroupContext MLS extension `group_trust_anchors`
of type PerDomainTrustAnchors. The syntax is described using
the TLS Presentation Language [@!RFC8446].

Each PerDomainTrustAnchor represents a specific identity domain which is
expected and authorized to participate in the MLS group. It contains the
domain name and the specific trust anchor used to validate identities for
members in that domain. 

~~~ tls
struct {
    opaque domain_name<V>;
    CredentialType credential_type;
    select (Credential.credential_type) {
        case x509:
            Certificate chain<V>;
    };
} PerDomainTrustAnchor;
    
struct {
    PerDomainTrustAnchor domain_anchors<V>;  
} PerDomainTrustAnchors;

PerDomainTrustAnchors group_trust_anchors;
~~~

An MLS client which implements this specification SHOULD include the
`group_trust_achors` extension in the `extensions` field in the
GroupContext of groups it creates. It includes the per-domain trust
anchors for all the domains expected and authorized to participate in
the group. As new members of the group are added or removed, the member
which Commits these membership changes is expected to maintain the list
of trust roots up-to-date by also including an appropriate 
GroupContextExtensions Proposal any time the list of expected federated
domains changes. Likewise, when any of the trust anchors used in a domain
changes, an appropriate member needs to Commit a GroupContextExtensions
Proposal updating the list of trust roots.

# IANA Considerations

This document proposes registration of a new MLS Extension Type.

RFC EDITOR: Please replace XXXX throughout with the RFC number assigned to this document

## group_trust_anchors MLS Extension Type

The `group_trust_anchors` MLS Extension Type is used inside GroupContext objects. It
contains a PerDomainTrustAnchors object representing the trust anchors which are expected
for identity validation inside the MLS group.

~~~~~~~~
  Template:
  Value: 0x0009
  Name: group_trust_anchors
  Message(s): This extension may appear in GroupContext objects
  Recommended: Y
  Reference: RFC XXXX
~~~~~~~~


# Security Considerations

The Security Considerations of MLS apply.

Improper use of the extension in this document could allow the creator of an
MLS group or the sender of a GroupContextExtensions Proposal to maliciously
add or remove authorized domains from a group, and to impersonate members from
specific domains. Therefore it is vital that all clients which implement this
extension 
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
