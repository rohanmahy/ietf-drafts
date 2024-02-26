# Basic Operations

This section walks a user through operations in the transport protocol in
the order they are necessary to show a basic messaging flow between Alice,
Bob, and Cathy:

- Alice get the internal identifier for Bob †
- Alice gains consent to talk to Bob ††
- Alice fetches initial keying material for Bob's clients
- (Alice create a room) †††
- Alice adds Bob to a room
- Alice sends a message to the room
- Bob sends a message to the room
- The hub/owning provider fans out the message
- Bob adds a new client
- Alice repeats the first three steps for Cathy
- Alice adds Cathy to the room
- Alice removes Bob from the room

† is a placeholder for a later discovery mechanism.

†† there is no consensus in the design team about the full extent of
consent requirements, but there is agreement that they will exist.

††† as mentioned in the text, creating a room is a local provider
action, which is out of scope of MIMI.

## Get internal identifier for Bob

This is a placeholder for a possibly more sophisticated discovery
mechanism. It is not intended to be directly implemented.

Alice fetches the internal identifier for some field of Bob's, in
this example his handle.

~~~
GET /identifierDiscovery/{domain}
~~~

The request body is described as:

~~~ tls
enum {
  reserved(0),
  handle(1),
  nick(2),
  email(3),
  phone(4),
  partialName(5),
  wholeProfile(6),
  oidcStdClaim(7),
  vcardField(8),
  (255)
} IdentifierType;

struct {
  IdentifierType type;
  string searchValue;
  select(type) {
     case oidcStdClaim:
       string claimName;
    case vcardField:
       string fieldName;
  };
} IdentifierRequest;

~~~

The response body is described as:

~~~ tls
enum {
  success(0),
  notFound(1),
  ambiguous(2),
  forbidden(3),
  unsupportedField(4),
  rateLimited(5), 
  (255)
} IdentifierDiscoveryCode;

struct {
  IdentifierDiscoverCode responseCode;
  IdentifierUri uri;
} IdentifierResponse;
~~~

**TODO**: The format of specific identifiers is discussed in
{{?I-D.mahy-mimi-identity}}. Any specific conventions which are needed
should be merged into this document.

## Get consent for Bob

Alice sends a message requesting consent to add him to a room. At some
later time, Bob grants consent to Alice.

Still **TODO** based on lack of consensus of consent scope.
See Section 6.1 of {{?I-D.mahy-mimi-transport-design-reqs}} for
some requirements.

~~~
POST /consent/{domain}/

enum {
  reserved(0),
  request(1),
  grant(2),
  reject(3),
  (255)
} TriStateOperation;

struct {
  TriStateOperation consentOperation;
  IdentifierUri requesterUri; 
  IdentifierUri targetUri;
  optional RoomId roomId;
} ConsentEntry;

struct {
  TriStateOperation moderationOperation;
  IdentifierUri requesterUri; 
  IdentifierUri targetUri;
  RoomId roomId;
} ModerationEntry; 




## Fetch initial key material for Bob

The action attempts to claim initial keying material for all the clients
of a list of users at a single provider. The keying material may not be
reused unless identified as "last resort" keying material.

If the protocol is MLS 1.0 (mls10) the initial key materials are MLS
KeyPackages.

~~~
POST /claim-keymaterial/{domain}
~~~

The request body has the following form.

~~~ tls
enum {
    reserved(0),
    mls10(1),
    (255)
} Protocol;

struct {
    opaque uri<V>;
} IdentifierUri;

struct {
    Protocol protocol;
    IdentifierUri requestingUser;
    IdentifierUri targetUsers<V>;
    IdentifierUri roomId;
    select(protocol) {
        case mls10:
            CipherSuite acceptableCiphersuites<V>;
            RequiredCapabilities requiredCapabilities<V>;
            bool lastResortAllowed;
    };
} KeyMaterialRequest;
~~~

The response contains a list of users which contain a list of clients.
Each client provides keying material or returns a (client) error code.
Each user provides a list of clients (codes or material), and fully succeeds
(all clients succeed), partially succeed (some clients succeed), or returns
a (user) error code.

~~~ tls
enum {
    success(0);
    partialSuccess(1);
    incompatibleProtocol(2);
    noCompatibleMaterial(3);
    userUnknown(4);
    noConsent(5);
    noConsentForThisRoom(6);
    userDeleted(7);
    (255)
} KeyMaterialUserCode;

enum {
    success(0);
    keyMaterialExhausted(1);
    onlyLastResort(2);
    nothingComptible(3);
    (255)
} KeyMaterialClientCode;


struct {
    KeyMaterialClientCode clientStatus;
    IdentifierUri clientUri;
    select(protocol) {
        case mls10:
            KeyPackage keyPackage;
    };
} ClientKeyMaterial;

struct {
    KeyMaterialUserCode userStatus;
    IdentifiedUri userUri;
    ClientKeyMaterial clients<V>;
} UserKeyMaterial;

struct {
    Protocol protocol;
    UserKeyMaterial users<V>;
} KeyMaterialResponse;
~~~

**ISSUE**: Do we want key material requests to be combined for multiple users?

## Create a room

Creating a room is done between a client and its local provider and is
out of scope of MIMI. However, we assume that the room has the following
policy document:

~~~
membershipStyle = membersOnly;
multiDevice = true;
moderated = false;
canAddUsers = [admin, owner];
canRemoveUsers = [admin, owner];

~~~

We also assume that Alice is the owner, and therefore an administrator,
of the room.

~~~
ownerRole = ["Alice@a.example"];
adminRole = ["Alice@a.example"];
activeParticipants = ["Alice@a.example"];
~~~


## Alice adds Bob

Adds, removes, and policy changes to the room are all forms of updating the
room state. They are accomplished using the updateRoom transaction which is
used for updating the room or its underlying MLS group.

~~~
POST /updateRoom/{roomId}
~~~

In MLS 1.0, any change to the room policy document is always expressed
using a `GroupContextExtensions` proposal. Likewise, any change to the
participant list is always communicated via a `ParticipantListPatch`
proposal type. The policy change needed to add a user MUST happen either
before or simultaneously with the corresponding MLS operation.

~~~ tls
enum {
  reserved(0),
  system(1),
  owner(2),
  admin(3),
  regular_user(4),
  visitor(5),
  banned(6),
  (255)
} Role;

struct {
  IdentifierUri user;
  Role roles<V>;
} UserRoles;

struct {
    UserRoles addUsers<V>;
    UserRoles updateUsers<V>;
    IdentifierUri removeUsers<V>;
} ParticipantListPatch;
~~~

Each user can be added with one or more roles. This list of roles can be
updated. Note that removing a user does not ban the user. To ban a user,
update their role to `banned`.

The updateRoom request body is described below:

~~~ tls
select(room.protocol) {
  case mls10:
    PublicMessage proposalOrCommit;
      select (proposalOrCommit.content.content_type) {
        case commit:
          optional<Welcome> welcome;
          GroupInfo groupInfo;   /* without embedded ratchet_tree */
          RatchetTreeOption ratchetTreeOption;
      };
};
~~~

In this case Alice creates a Commit containing a ParticipantListPatch
proposal adding Bob@b.example, and Add proposals for all Bob's MLS clients.
Alice includes the Welcome message which will be sent for Bob, a
GroupInfo object for the hub provider, and complete `ratchet_tree` extension.

~~~ tls
enum {
  reserved(0),
  full(1),
  compressed(2),
  delta(3),
  patch(4),
  byReference(5)
  (255)
} RatchetTreeRepresentation;

struct {
  RatchetTreeRepresentation representation;
  select (representation) {
    case full:
      Node ratchet_tree<V>;
  };
} RatchetTreeOption;
~~~

The response body is described below:

~~~ tls
enum {
  success(0),
  wrongEpoch(1),
  notAllowed(2),
  hubUnresponsive(3),
  invalidProposal(4),
  (255)
} UpdateResponseCode;

struct {
    UpdateResponseCode responseCode;
    string errorDescription;
} UpdateRoomResponse
~~~

## Alice sends a message to the room

Alice creates a message using the MIMI content format {{?I-D.ietf-mimi-content}},
and sends it to the provisionalMessage endpoint for the target room.

~~~
POST /provisionalMessage/{roomId}
~~~

If the protocol is MLS 1.0, the request body is an MLS PrivateMessage
(an application message).

The response merely indicates if the message was accepted by the hub
provider.

## The hub/owning provider fans out the message

If the hub provider accepts the message it fans the message out

The hub provider also fans out any messages which originate from itself
(ex: removing deleted users) and messages which modify the policy or
participant list.

~~~
POST /fanout/{roomId}
~~~

If the protocol is MLS 1.0, the request body is an MLSMessage with wire_format
one of PrivateMessage (application message), PublicMessage (Commit or
Proposal), or Welcome.

## Bob sends a message to the room

This is no different from Alice sending a message, which is fanned out in
exactly the same way.

## Bob adds a new client

For Bob's new client to join the MLS group and therefore fully participate
in the room with Alice, Bob needs to fetch the MLS GroupInfo (or analogous).

~~~
POST /claimGroupInfo/{roomId}
~~~

In the case of MLS 1.0, Bob provides a credential proving his client's
real or pseudonymous identity (for permission to join the group).

~~~ tls

struct {
  select (protocol) {
    case mls10:
      SignaturePublicKey requestingSignatureKey;
      Credential requestingCredential;
  };
} GroupInfoRequest;

~~~

The response body contains the GroupInfo and a way to get the ratchet_tree.

~~~ tls
struct {
  GroupInfoCode status;
  select (protocol) {
    case mls10:
      GroupInfo groupInfo;   /* without embedded ratchet_tree */
      RatchetTreeOption ratchetTreeOption;
  };
} GroupInfoResponse;
~~~

**ISSUE**: in the MLS case, what security properties are needed to protect a
GroupInfo object in the MIMI context are still under discussion. It is
possible that the requester only needs to prove possession of their private
key. The GroupInfo in another context might be sufficiently sensitive that
it should be encrypted from the end client to the hub provider (unreadable
by the local provider).

## Alice adds Cathy

Alice gets the identifier for Cathy, obtains consent, claims initial
keying material for her clients, and adds her to the room in exactly
the same way as in previous steps.

## Alice removes Bob from the room

Alice removes Bob by sending an updateRoom transaction. In MLS 1.0
it contains a Commit with a ParticipantListPatch proposal (removing
Bob@b.example) and Remove proposals for Bob's client, no Welcome,
a GroupInfo object, and a full `ratchet_tree` extension.




Typing is an e2e ephermeral message. Hubs know they can discard it if there is no active websocket or the target provider is down

