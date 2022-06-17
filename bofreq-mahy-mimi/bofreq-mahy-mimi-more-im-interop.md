# Name: More Instant Messaging Interoperability (MIMI)

## Description

The IETF has been working on Instant Messaging Interoperability since the late 1990s. At the time, different groups within IETF proposed separate protocol suites (SIMPLE, APEX, and XMPP) because the community could not come to consensus on a single protocol (arguably due to a lack of consensus on the additional requirements which made these proposals unique). In the interests of interoperability, the IMPP Working Group developed a general framework for interoperability [RFC3860], and the Common Presence and Instant Messaging (CPIM) Message Format [RFC3862], an interoperability format that could pass through gateways among these protocols, even when end-to-end encrypted.  

The CPIM model assumed standalone encryption of each message using a protocol such as S/MIME [RFC8551] or PGP [RFC3156]. This model was not widely adopted, but many Instant Messaging systems around this time frame began to add optional end-to-end encryption with OTR (Off The Record) [], and eventually incorporated variants of the Double Ratchet protocol [], originally popularized by Signal.

Today, group chats are the norm, most modern instant messaging systems are end-to-end encrypted (many by default or always) using a variant of DoubleRatchet, and the typical feature set includes a plethora of features not included in CPIM: plain text and rich text messaging, delivery notifications, read receipts, replies, reactions, editing or deleting previously sent messages, and expiring messages. Almost all systems provide a way to share files/audio/videos, and many support calling and/or conferencing features (often using WebRTC). Some IM vendors are implementing MLS [I-D.ietf-mls-protocol], a group key establishment protocol motivated by the desire for group chat with efficient end-to-end encryption.

Unfortunately, federation of these IM systems is still rare and interoperability of the major IM systems in almost non-existent. 
It would be incredibly beneficial to provide interoperable best practices and solutions which IM vendors can incorporate into modern IM systems. Indeed, large customers and governments are already putting pressure on these IM vendors. The European Union's Digital Markets Act [DMA] is a recent dramatic example.

The goal of MIMI is to gather requirements common to many IM systems, focusing on the most immediate needs first; to identify and evaluate existing solutions to these specific problems; and to assemble standards and technology which already largely exist into profiles and best current practices. Where special expertise in another Working Group or standards body is required, that work would be delegated to the specialty group. Specific areas for MIMI to address include:

* End-to-end IM Identity: describe types of IM identifiers (especially in a federated domain context), problem statement, survey of existing relevant technologies, some concrete solutions using existing technology.
* Discovery
* Use of MLS in the Instant Messaging Context: (ex: long-lived persistent groups)
* Content negotiation
* Content format interoperability
* Administrative setup of federation: (ex: agreement on certificates, contact information, abuse policies)

While some of the deliverables could be specific to MLS, this does not preclude the rest of the deliverables being used without MLS where architecturally coherent.

## Required Details:
* Status: WG Forming
* Responsible AD: Francesca Palombini <francesca.palombini@ericsson.com>, Murray Kucherawy <superuser@gmail.com>
* BoF proponents: Rohan Mahy <rohan.mahy@wire.com>
* BoF chairs: 
* Number of people expected to attend: 40
* Length of session: 1 hour
* Conflicts: Dispatch, artarea, secarea, OAUTH, MLS, JSON Web Proofs BoF (JWP)
* Chair Conflicts: TBD
* Technology Overlap: 
* Key Participant Conflict: Rohan Mahy (not Friday), Richard Barnes

## Information for IAB/IESG
Relationships with related ongoing standards work:
* Identity related
   - OUATH standards, especially draft-ietf-oauth-dpop
   - Verifiable Credentials
* MLS protocol for MLS-related work
* Federation: FastFed
 
Any protocols or practices that already exist in this space:
* CPIM [RFC3862] and Address resolution for IM and Presence [RFC 3861]
* MIME/Media Types specifications
* X.509 best practices (ex: RFC6125)
* ACME [RFC8555]

Which (if any) modifications to existing protocols or practices are required:
* Possibly update CPIM
* New challenge type for ACME as one approach

Which (if any) entirely new protocols or practices are required:
* Possibly define a protocol for read receipts inside an environment that looks like a message-bus (ex: MLS), but this could be broken out elsewhere.

Implementations:
* There are dozens of IM implementations, but we have a lack of standards guidance in this area, typically the opposite problem at IETF. 
* Interested vendors include: Cisco, Microsoft, Amazon, Wire

## Agenda
5 minutes: Agenda Bash / Administrativia
10 minutes: Identity problem statement
10 minutes: Content Negotiation problem statement
10 minutes: Federation administration
15 minutes: Open Discussion
10 minutes: Charter-Bashing

## Links to the mail list, draft charter, Internet-Drafts
Mailing list: mimi@mailman.ietf.org
Internet-Drafts: 
https://www.ietf.org/archive/id/draft-mahy-mimi-problem-statement-00.txt (coming)
https://www.ietf.org/archive/id/draft-mahy-mimi-identity-technologies-00.txt (coming)
https://www.ietf.org/archive/id/draft-mahy-dispatch-immi-content-00.txt


[DMA] https://oeil.secure.europarl.europa.eu/oeil/popups/ficheprocedure.do?reference=2020/0374(COD)&l=en
