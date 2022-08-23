%%%
title = "Ratchet Tree Delta Format for Message Layer Security (MLS)"
abbrev = "MLS Ratchet Tree Delta"
ipr= "trust200902"
area = "sec"
workgroup = "MLS"
keyword = ["mls","ratchet tree","ratchet_tree_delta","delta","patch"]

[seriesInfo]
status = "informational"
name = "Internet-Draft"
value = "draft-mahy-mls-ratchet-tree-delta-00"
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

This document defines an extension to the Message Layer Security (MLS) protocol
which allows a client to communicate the delta between one ratchet tree and the
previous version of that ratchet tree. The extension enables efficient external
join operations even in very large groups. 

{mainmatter}

# Terminology

The terms MLS client, MLS group, and KeyPackage have the same meanings as in
the MLS protocol [@!I-D.ietf-mls-protocol].

# Introduction

MLS was designed to make group operations efficient, even for groups of tens or
hundreds of thousands of members. One feature of MLS, the External Join,
requires that the new client have a GroupInfo object (or its equivalent) and a
copy of the complete ratchet tree. 

The joining client could somehow arrange to get the ratchet tree directly from
an existing client (A) or it could get it from the Distribution Service (DS).
If from the the DS, it needs to synthesize the ratchet_tree from unencrypted
(MlsPlaintext) handshake messages (B), or any client which generates a Commit
message, needs to provide a copy of the ratchet_tree to the DS (C).

Option B requires MLS handshake messages to be unencrypted, and requires
substantial processing on the part of the DS that would not otherwise be
needed or even useful. Both Options A and C require sending a potentially large
ratchet_tree. 

The ratchet_tree for a group of 10000 clients could easily require a few
megabytes. If a stable group of that size required keying material updates at
least every 90 days, a new multi-megabyte GroupInfo would be generated about
once every 13 minutes, assuming no adds or joins. That is a large amount of
overhead.

Fortunately the number of changes in the ratchet tree from one epoch to another
are usually small. This document proposes a ratchet_tree_delta extension which
can express just the changed nodes in a ratchet_tree from epoch to epoch.

# RatchetTreeDelta Syntax

When describing the differences between two ratchet trees, we use the terms
original ratchet_tree and new ratchet_tree.

We use the term node index to refer to the location of the node in the 
ratchet_tree, counting from zero from left to right. For example consider the
tree in the example below, which has 4 nodes (leaf_index 0 through 3) and 7
nodes total (node_index 0 through 6).

~~~~~ 
       Y
     /   \
    /     \
  X         Z
 /  \      / \
A    B    C   D
  
0    1    2   3   leaf_index

0 1  2 3  4 5 6   node_index
~~~~~

`NodeWithPosition` contains a Node object and its position in the new
ratchet_tree.

~~~~~ tls
struct {
  unit32 node_index;
  Node node;
} NodeWithPosition

struct {
  uint32 number_of_leaves;
  uint32 blanked_node_indexes<V>;
  NodeWithPosition added_or_replaced_nodes<V>;
  opaque previous_tree_hash<V>;
  opaque new_tree_hash<V>;
} RatchetTreeDelta

RatchetTreeDelta ratchet_tree_delta;
~~~~~

The ratchet_tree_delta begins with the `number_of_leaves` field which is the
total number of number of leaves in the new ratchet_tree. This number is
always a power of two (1, 2, 4, 8, 16, etc.). If the size of the tree
increases, the new ratchet_tree is populated with blank nodes. If the size of
the new tree decreases, we assume that the numerically larger half of the 
original tree was blanked and then truncated.

Next is `blanked_node_indexes` a vector of node indexes which have been
been blanked or removed from the original tree which still fit within the new
ratchet_tree. 

Next is `added_or_replaced_nodes` a vector of `NodeWithPosition` objects for
all the nodes which were either modified or added in the new ratchet_tree.

Finally the last two elements, the `previous_tree_hash` and the
`new_tree_hash`, are the tree hashes of the original ratchet_tree root node
and the new ratchet_tree root node respectively. These two items allow the
receiver to verify that the ratchet_tree_delta was applied to the intended
original ratchet_tree, and that the new ratchet_tree is correct after
applying the ratchet_tree_delta.

# When to use the extension

This `ratchet_tree_delta` is useful when a client generating a new Commit
wants to efficiently communicate the changes to the Distribution Service.

Non-members do not use the extension for joining a group. Welcome messages
and the GroupInfo object used for constructing an External Commit need
to contain or reference a full ratchet_tree.

# Examples

TBD

# IANA Considerations

## ratchet_tree_delta GroupContext extension

The `ratchet_tree_delta` MLS Extension Type is used inside GroupContext objects,
in many locations where the `ratchet_tree` extension can be used. It contains
a representation of changes made compared to the `ratchet_tree object` in the
previous epoch.

~~~~~~~~
  Template:
  Value: 0x0008
  Name: ratchet_tree_delta
  
  Message(s): This extension may appear in GroupContext objects
  Recommended: Y
  Reference: RFC XXXX
~~~~~~~~

Description: Changes since the previous `ratchet_tree` which can be applied
to construct a new `ratchet_tree`.

# Security Considerations

This section requires serious attention. The `ratchet_tree_delta` extension
contains a `previous_tree_hash` and `new_tree_hash` to insure that the client
applies the `ratchet_tree_delta` to the correct tree and that the constructed
tree matches the intended tree.

{backmatter}
