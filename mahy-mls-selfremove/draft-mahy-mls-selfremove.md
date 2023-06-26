%%%
title = "The SelfRemove Proposal for Message Layer Security (MLS)"
abbrev = "MLS SelfRemove Proposal"
ipr= "trust200902"
area = "sec"
workgroup = "MLS"
keyword = ["mls","SelfRemove","self remove","leave","proposal"]

[seriesInfo]
status = "informational"
name = "Internet-Draft"
value = "draft-mahy-mls-selfremove-00"
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

This document describes the SelfRemove Message Layer Security (MLS) Proposal
type, which improves handling of a client removing itself from an MLS group.

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

RFC EDITOR: PLEASE REMOVE THE FOLLOWING PARAGRAPH. The source for
this draft is maintained in GitHub. Suggested changes should be
submitted as pull requests at https://github.com/rohan-wire/ietf-drafts.
Editorial changes can be managed in GitHub, but any substantive
change should be discussed on the MLS mailing list (mls@ietf.org).

The design of the MLS protocol prevents a member of
an MLS group from removing itself immediately from the group. To cause
an immediate change in the group, a member must send a Commit message.
However the sender of a Commit message knows the keying material of the
new epoch and therefore needs to be part of the group. Instead a member
wishing to remove itself can send a Remove Proposal and wait for another
member to Commit its Proposal. 

Unfortunately, MLS clients that join via an External Commit ignore
pending, but otherwise valid, Proposals. The member trying to remove itself has
to monitor the group and send a new Proposal in any new epoch until the member is
removed. In a
group with a burst of external joiners, a member connected over a
high-latency link (or one that is merely unlucky) might have to wait
several epochs to remove itself. A real-world situation in which this happens
is a member trying to remove itself from a conference call as several dozen
new participants are trying to join (often on the hour).

This document described a new `SelfRemove` Proposal extension type. It is
designed to be included in External Commits.

# Extension Description

This document specifies a new MLS Proposal type called `SelfRemove`. Its syntax
is described using the TLS Presentation Language [@!RFC8446] below (its contents
is an empty struct). It is allowed in External Commits and requires an UpdatePath. 

~~~ tls-presentation
struct {} SelfRemove;

struct {
    ProposalType msg_type;
    select (Proposal.msg_type) {
        case add:                      Add;
        case update:                   Update;
        case remove:                   Remove;
        case psk:                      PreSharedKey;
        case reinit:                   ReInit;
        case external_init:            ExternalInit;
        case group_context_extensions: GroupContextExtensions;
        case self_remove:              SelfRemove;
    };
} Proposal;
~~~

The description of behavior below only applies if all the
members of a group support this extension in their
capabilities; such a group is a "self-remove-capable group". 

An MLS client which implements this specification can send a
SelfRemove Proposal whenever it would like to remove itself
from a self-remove-capable group. Because the point of a
SelfRemove Proposal is to be available to external joiners
(which are not yet members), these proposals MUST be sent
as an MLS PublicMessage. 

Whenever a member receives a SelfRemove Proposal, it includes
it along with any other pending Propsals when sending a Commit.
It already MUST send a Commit of pending Proposals before sending
new application messages.

When a member receives a Commit with an embedded SelfRemove Proposal,
it treats the proposal like a Remove Proposal, except the leaf node to remove
is determined by looking in the Sender `leaf_index` of the original Proposal.
The member is able to verify that the Sender was a member. 

Whenever a new joiner is about to join a self-remove-capable group with an
External Commit, the new joiner MUST fetch any pending SelfRemove Proposals
along with the GroupInfo object, and include the SelfRemove Proposals
in its External Commit by reference. The new joiner validates the SelfRemove
Proposal before including it by reference, except that it skips the validation
of the `membership_tag` because a non-member cannot verify membership.

The MLS Distribution Service (DS) needs to validate SelfRemove Proposals it
receives (except that it cannot validate the `membership_tag`). If the DS
provides a GroupInfo object to an external joiner, the DS SHOULD attach any
SelfRemove proposals known to the DS to the GroupInfo object. 

As with Remove proposals, clients need to be prepared to receive the Commit
message which removes them from the group via a SelfRemove. If the DS does
not forward a Commit to a removed client, it needs to provide inform the removed
client out-of-band.

# IANA Considerations

This document proposes registration of a new MLS Proposal Type.

RFC EDITOR: Please replace XXXX throughout with the RFC number assigned to this document

## self_remove MLS Proposal Type

The `self_remove` MLS Proposal Type is used for a member to remove itself
from a group more efficiently than using a `remove` proposal type, as the
`self_remove` type is permitted in External Commits.

~~~~~~~~
  Template:
  Value: 0x0008
  Name: self_remove
  Recommended: Y
  External: Y
  Path Required: Y
  Reference: RFC XXXX
~~~~~~~~


# Security Considerations

The Security Considerations of MLS apply.

An external recipient of a SelfRemove Proposal cannot verify the
`membership_tag`. However, an external joiner also has no way to
completely validate a GroupInfo object that it receives. An insider
can prevent an External Join by providing either an invalid GroupInfo object
or an invalid SelfRemove Proposal. The security properties of external joins
does not change with the addition of this proposal type.


{backmatter}
