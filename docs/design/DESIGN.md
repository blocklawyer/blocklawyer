# Put It On The Block: Design Document

Status: Draft 1, for discussion. No code yet.
Owner: Enrico Schaefer (private project, not Traverse work)
Working name: "On The Block" (placeholder; naming is an open decision)

---

## 1. What we are building

A mobile app and matching website where members record commitments and anchor proof of them on a public blockchain. A commitment is either personal (just me) or shared (me and one or more other members). The audio and AI transcript live off-chain, encrypted. The chain holds only a fingerprint (hash) and a timestamp, which lets anyone later prove the record existed at that moment and has not been altered.

The origin case: two spouses disagree about what was said and agreed. One asks, "Are you willing to put that on the blockchain?" Both say yes, the app records the exchange, both approve the transcript, both seal it, and it is anchored. Months later either of them can pull it up and verify it.

v1 is a consumer product with a light social layer. The architecture should leave a clean path to a later "contracts mode" where authorized parties create and amend legally binding obligations.

### Design principles

1. No unconsented content, ever. Every party to a shared commitment is a verified member who affirmatively consents at each stage. There is no upload of outside recordings in v1.
2. Nothing personal goes on-chain. The chain sees hashes and timestamps only. Not names, not wallet addresses that link two people, not content.
3. Parties control their record. Access to a shared commitment is decided by its parties, not by the platform and not by one party alone.
4. Verifiable without us. A user should be able to prove a commitment is authentic even if the company is gone.
5. Fun first, but built like it will end up in a courtroom. Because some of these will.

---

## 2. Issues to resolve before we build

These are ordered by how much they change the architecture.

### 2.1 Immutability vs. deletion and privacy law

Blockchain records are permanent. Privacy laws (GDPR, CCPA/CPRA, state privacy statutes) give users deletion rights, and people will want to erase things after a breakup.

Resolution in this design: only a salted hash goes on-chain. The content key and the files are off-chain. Deleting the off-chain content and destroying the encryption keys ("crypto-shredding") leaves an on-chain hash that points to nothing and cannot be reversed into content. The salt matters: without it, a short transcript ("I agree to do the dishes") could be guessed and hashed to confirm a match.

Open question: in a shared commitment, can one party delete their copy without deleting the other party's copy? Proposed default: yes. Each party holds their own wrapped key. Party A can destroy their access; Party B keeps theirs. Full destruction of the content requires all parties, or happens when the last party deletes.

### 2.2 Recording consent laws

Several states require all parties to consent to a recording of a private conversation (California, Florida, Illinois, Maryland, Massachusetts, Montana, New Hampshire, Pennsylvania, Washington, and others; Michigan is debated). List UNVERIFIED; confirm against statutory text before launch.

Resolution in this design: a recording can only start after every party has tapped "I consent to recording" on their own device inside a live session (Section 5.2). The consent events are logged, signed, and hashed into the record itself. The recording opens with an auto-generated spoken notice ("This conversation is being recorded for On The Block by...") and each party's verbal yes is captured on audio.

Residual risk: bystanders. A child or friend talking in the background has not consented. Mitigation: in-app notice, terms of use, and a "remove segment" option during transcript review before sealing.

### 2.3 Is a commitment a contract?

v1 says "fun, not legal." Courts may not care what we called it. A signed, timestamped, mutually approved record of a promise looks a lot like a contract. Agreements between spouses also raise their own rules (postnuptial agreements often require writing, financial disclosure, and independent counsel; agreements in contemplation of divorce or on custody may be unenforceable or void as against public policy). All UNVERIFIED and state-specific.

Decisions:
- v1 terms state that commitments are not intended to create legal obligations, and each sealing screen repeats it in one line.
- We do not market v1 as a contract tool.
- The record should still be built to evidentiary standards (Section 2.4), because users will try to use it in disputes regardless.
- Contracts mode (Section 11) is a separate, opt-in product with its own identity verification, e-signature compliance (ESIGN Act and UETA, UNVERIFIED), and templates.

### 2.4 Subpoenas and evidence

The first divorce case involving the app will produce a subpoena to us. Design choice:

Option A, platform can decrypt. Easier key recovery, easier support, and we can produce content when compelled.
Option B, end-to-end encrypted. We hold only ciphertext and metadata. We cannot produce content; the parties have it and can be compelled themselves.

Recommendation: Option B for content, with an optional user-controlled recovery method (Section 7.3). This keeps us out of the middle of family disputes and is a better privacy story. We still hold metadata (who, when, status), so a records-request policy is needed either way.

Authentication: to be useful as evidence, a record needs a chain of custody. Our design gives that: device signatures from each party, a signed consent log, hashes of audio and transcript, and an on-chain timestamp. We should publish a plain-English "How to authenticate an On The Block record" page and an open-source verifier.

### 2.5 Coercion and abuse

An app that lets one person pressure another into a recorded "commitment" can be a tool of control in an abusive relationship.

Mitigations:
- Either party can decline at any stage before sealing with no explanation and no notification text that shames ("Your request was not accepted").
- Rate limits on requests to the same person.
- Block, mute, and report on every connection.
- No public scores of other people without their consent (Section 8).
- A private "I was pressured" flag on a sealed commitment, visible only to the flagging party and to our trust-and-safety team, which can be used later to attach context.
- Safety resources link in settings.

### 2.6 AI transcript accuracy

The transcript is a machine product. It will get words wrong, and one wrong word can flip a promise.

Resolution: the audio is the canonical record. The transcript is a convenience. Every party reviews the transcript before sealing and can propose corrections; corrections must be accepted by all parties. Both audio and final transcript are hashed. The UI labels the transcript "AI transcript, approved by all parties on [date]."

### 2.7 Identity

The on-chain proof shows that some content existed at a time. It does not prove who said it. Identity comes from:
- Account verification (phone at minimum; government ID optional in v1, required in contracts mode).
- Device-bound signing keys (passkeys / secure enclave) used to sign each consent and seal event.
- Voice on the audio itself.

Open question: do we require ID verification for shared commitments in v1? Proposed: no for v1, with a visible "ID verified" badge for members who opt in.

### 2.8 Crypto exposure and app store rules

Users should never need a wallet, a seed phrase, or a token.

Decisions:
- No token, no NFT, no coin. This avoids securities, money transmission, and app store issues around crypto sales.
- The platform pays gas. Users pay in dollars (subscription or per-anchor) through normal app store billing.
- Each user gets an embedded signing key created on sign-up. It never holds funds.

### 2.9 Other issues to track

- Minors: 18+ only. Age gate at sign-up.
- Death and incapacity: what happens to a shared record when one party dies. Proposed: surviving party retains access; estate access through a documented process.
- Group commitments (3+ parties): consent and amendments require unanimity by default. Large groups are a later feature.
- Content moderation: we cannot see E2EE content. Reports must include the reporter's own decrypted copy, which they choose to share.
- Defamation and harassment through ratings (Section 8).
- Data residency if we launch outside the U.S.
- Recording in a car or public place with poor audio.
- Accessibility: a typed commitment path for members who cannot or will not record audio.

---

## 3. Core concepts (domain model)

| Concept | What it is |
|---|---|
| Member | A verified user account. Has a profile, a device signing key, and an encryption key pair. |
| Connection | A mutual relationship between two members (both accept). Required before a shared commitment. |
| Circle | An optional named group of connections ("Family," "Running club"). Used for sharing and group commitments. |
| Commitment | The core record. Personal (one party) or shared (two or more parties). Has versions. |
| Party | A member's role on a commitment. Each party has independent consent, access, and rating rights. |
| Session | The live window in which parties join, consent, and record. |
| Recording | Encrypted audio file. Canonical record. |
| Transcript | Encrypted AI transcript, edited and approved by all parties. |
| Consent Event | A signed, timestamped record of a party's yes or no at each stage. |
| Version | A sealed state of a commitment. An amendment creates a new version linked to the prior one. |
| Seal | The moment all parties sign the final version. Triggers anchoring. |
| Anchor | The on-chain proof: the version's hash included in a Merkle batch whose root is written to the chain. |
| Access Grant | Permission for a non-party (e.g., a mediator, counselor, or friend) to view a commitment. Requires consent of all parties. |
| Check-in | A periodic or ad hoc report by a party on whether a commitment is holding up. |
| Rating | A party's score of how a commitment was kept. Private to the parties by default. |

---

## 4. Commitment lifecycle

```
            +-----------+
            |   DRAFT   |  initiator writes a title, picks parties, sets options
            +-----+-----+
                  | invite sent
            +-----v-----------+
            | AWAITING CONSENT|  each party accepts or declines the invite
            +-----+-----------+
                  | all parties joined the live session and consented
            +-----v-----+
            | RECORDING |  audio captured; verbal notice and verbal yes recorded
            +-----+-----+
                  | stop
            +-----v------------+
            | TRANSCRIPT REVIEW|  AI transcript generated; parties propose and accept edits
            +-----+------------+
                  | all parties approve
            +-----v-----+
            |  SEALING  |  each party signs the version hash on their device
            +-----+-----+
                  | all signatures collected
            +-----v-----+
            |  SEALED   |  queued for anchoring
            +-----+-----+
                  | included in an on-chain batch
            +-----v-----+
            | ANCHORED  |  verifiable; check-ins and ratings open
            +-----+-----+
                  |
     +------------+-------------+-----------------+----------------+
     |            |             |                 |                |
 AMENDED      FULFILLED      RELEASED          BROKEN           EXPIRED
 (new version  (all parties   (all parties      (one party       (end date
  goes through  mark done)     agree to end)     marks broken;    passed)
  the same flow)                                 the other may
                                                 dispute)
```

Rules:
- Any party can abandon before SEALED. Abandoned drafts and recordings are deleted after 7 days unless a party saves their own copy (proposed; open decision).
- After ANCHORED, nothing is ever edited. Every change is a new version that references the prior version's hash. The history is the point.
- FULFILLED, RELEASED, and EXPIRED are mutual or automatic. BROKEN is unilateral, so it is shown as "Marked broken by Alex; Sam disputes" when contested. We do not adjudicate.
- Status changes after anchoring are themselves signed events and are anchored in the next batch, so the full history is provable.

Personal commitments follow the same flow with one party and no consent handshake.

---

## 5. Key flows

### 5.1 Connecting with another member

1. Find by username, phone contact match (opt-in, hashed contact discovery), or QR code in person.
2. Send connection request. The other member accepts or ignores.
3. Connected members see each other's profile per the profile's visibility settings (Section 8.1).

A connection is required before a shared commitment. This keeps strangers from sending commitment requests.

### 5.2 Creating a shared commitment (in person, the origin case)

1. Initiator taps "Put it on the block," picks the other party, adds an optional title ("Who handles the holidays this year").
2. App opens a live session and shows a QR code and a push to the other party.
3. Other party opens the session on their own phone. Both see the same screen: "Alex wants to record a commitment with you."
4. Each taps "I consent to recording." Both consents are signed on device.
5. Recording starts on the initiator's phone (or on both phones, for redundancy; open decision). App plays a short spoken notice.
6. Parties talk. A prompt on screen suggests closing with each person saying "I agree."
7. Stop. Audio uploads encrypted. Transcript is generated.
8. Each party reviews the transcript on their own phone, can propose edits or remove a segment (e.g., a bystander), and approves.
9. App shows the seal screen: title, transcript, duration, parties, the line "This is a personal record, not a legal contract," and a "Seal it" button. Each party seals with a biometric prompt.
10. Done. Status shows "Sealed, anchoring." A few minutes later: "On the block," with a link to the proof.

### 5.3 Creating a shared commitment (remote)

Same flow, but the session runs an in-app audio call (WebRTC). The call is recorded server-side into an encrypted file, or recorded on each device and merged. This is a phase 2 feature; v1 supports in-person and remote typed commitments only.

### 5.4 Typed commitment (no audio)

For people who do not want audio. Parties co-edit text, approve, and seal. Same consent and seal rules.

### 5.5 Amending

Any party proposes an amendment from the commitment detail screen, by recording a new session or editing text. The amendment goes through consent, review, and seal. The new version links to the old. Both versions remain viewable with a diff view.

### 5.6 Checking the record ("check the block")

From the commitment detail screen, "Verify" does the following, on the device:
1. Download and decrypt the audio and transcript.
2. Recompute their hashes and the version hash.
3. Fetch the Merkle proof and the on-chain transaction.
4. Confirm the proof leads to the root recorded on-chain at the stated block time.
5. Show "Verified: unchanged since [date, time], block [number]."

The website offers the same check as a standalone verifier: drop in an exported record bundle and it checks against the chain without needing our servers.

### 5.7 Sharing with someone outside the commitment

A party requests a grant for a specific person (a member, or a non-member via a time-limited link). All other parties must approve. Grants can be view-only, expiring, and revoked. Every view is logged and visible to the parties.

Open question: should a party be able to export their own copy without the other party's approval? Practically, they can always screen-record or retell it. Proposed: yes, a party can export their own record bundle, and the other party is notified. Blocking export would give false comfort.

---

## 6. System architecture

```
  +----------------+       +----------------+
  |  Mobile app    |       |   Website      |
  |  (iOS/Android) |       |  (web app +    |
  |                |       |   verifier)    |
  +-------+--------+       +--------+-------+
          |   device keys, E2E encryption,   |
          |   signing happen on the client   |
          +----------------+-----------------+
                           |
                    HTTPS / WebSocket
                           |
  +------------------------v--------------------------+
  |                    API layer                      |
  |  auth, members, connections, commitments,         |
  |  sessions, consent log, grants, notifications     |
  +---+-----------+------------+------------+---------+
      |           |            |            |
+-----v----+ +----v-----+ +----v-------+ +--v-------------+
| Postgres | | Object   | | Transcribe | | Anchoring      |
| metadata | | storage  | | worker     | | service        |
| (no      | | encrypted| | (ephemeral | | batch -> Merkle|
| content) | | blobs    | |  decrypt,  | | root -> chain  |
+----------+ +----------+ |  no retain)| +--+-------------+
                          +------------+    |
                                      +-----v----------+
                                      | L2 blockchain  |
                                      | Anchor registry|
                                      | contract       |
                                      +----------------+
```

### 6.1 Proposed stack

| Layer | Proposal | Why |
|---|---|---|
| Mobile | React Native (Expo) | One codebase for iOS and Android; shares TypeScript with web. |
| Web | Next.js | Same language and components; verifier page can be fully static. |
| API | TypeScript (Node) | Same language end to end. |
| Database | Postgres | Relational data (members, connections, parties, versions) fits it. |
| File storage | S3-compatible object storage | Holds only ciphertext. |
| Transcription | Whisper-class model, self-hosted or a vendor with zero-retention terms | See 6.3. |
| Chain | Ethereum L2 (Base is the leading candidate) | Low cost, EVM tooling, public and credible. |
| Signing keys | Passkeys / secure enclave, via an embedded-wallet provider or our own | No seed phrases. |

All of this is open for discussion. The architectural choices that matter most are E2E encryption, off-chain content, and batch anchoring. The specific frameworks are replaceable.

### 6.2 What the server can and cannot see

The server sees: account info, connections, which members are parties to which commitment, timestamps, status, file sizes, hashes.
The server cannot see: audio, transcript text, titles (encrypted), amendment content.

Titles are encrypted too, so a commitment list leaks nothing sensitive if our database is breached.

### 6.3 The transcription problem

E2E encryption and server-side AI transcription conflict: the transcription service has to hear the audio.

Options:
- A. On-device transcription. Fully private. Quality and speed depend on the phone. Improving fast.
- B. Ephemeral server transcription. Client sends the content key for that job only. A worker decrypts in memory, transcribes, encrypts the result to the parties, and discards everything. Strong but not "zero knowledge."
- C. Third-party API with zero-retention contract. Simplest. Weakest privacy story.

Recommendation: A where the device supports it, B as fallback, disclosed plainly in the privacy policy. Avoid C for content.

---

## 7. Encryption and keys

### 7.1 Per-commitment content keys

1. Each version gets a random content key.
2. Audio and transcript are encrypted with it on the device.
3. The content key is wrapped (encrypted) separately for each party's public key and stored with the version.
4. To grant access to someone new, an existing party's device unwraps the key and re-wraps it for the grantee, only after all parties approve.
5. To revoke, we delete the grantee's wrapped key. (They may have kept a copy while they had access. The UI says so.)
6. Crypto-shredding = deleting all wrapped keys and the ciphertext.

### 7.2 Member keys

Each member has:
- A signing key (for consents, seals, status changes). Device-bound passkey where possible.
- An encryption key pair (for receiving wrapped content keys).
- Multi-device support by enrolling new devices from an existing device.

### 7.3 Recovery

E2E encryption means a member who loses every device loses access. That will happen, and the support calls will be painful.

Options, user-selectable:
- Recovery through another of their own devices (default).
- Recovery code written down at setup.
- Social recovery: a trusted connection holds a recovery share.
- Platform-escrowed recovery (opt-in, clearly labeled as weakening privacy).

Note the backstop: in a shared commitment, the other party still has the record and can re-grant access.

---

## 8. Social layer

The goal is a fun app people open more than once a month. The risk is turning a marriage into a public scoreboard.

### 8.1 Profiles

- Display name, photo, short bio, member since, ID-verified badge.
- Optional public stats: number of commitments sealed, self-reported "kept" rate. Off by default.
- Profile visibility: public, connections only, or private.

### 8.2 Ratings and check-ins

- Each party can rate how a commitment held up (1 to 5, plus optional note) at any check-in or at close.
- Ratings are visible only to the parties to that commitment. Never public by default.
- A member's own aggregate ("You kept 18 of 20") is private to them unless they choose to show it.
- No member can publish a rating about another member. This removes most of the defamation and harassment risk.
- Check-in reminders on a schedule the parties set ("Remind us monthly").

### 8.3 Optional public sharing

Parties can unanimously choose to share a commitment publicly (e.g., a public pledge, a friendly bet, a fitness goal with a friend). Public commitments get a share card and can be "witnessed" by connections. This is the growth and fun engine.

### 8.4 Feed

A feed of: your pending actions, upcoming check-ins, public commitments from connections, and milestones ("Alex and Sam kept their commitment for 100 days"). Milestones about shared commitments post only with all parties' opt-in.

---

## 9. Smart contract and anchoring

### 9.1 Why batch

Writing one transaction per commitment is expensive at scale and leaks activity patterns. Instead:

1. Sealed versions queue up.
2. Every N minutes (or when the queue hits a size limit), the anchoring service builds a Merkle tree of version hashes.
3. One transaction writes the Merkle root to the chain.
4. Each version stores its Merkle proof.

Cost is one transaction per batch, regardless of how many commitments are in it.

### 9.2 What gets hashed

```
version_hash = H(
  commitment_id_salted,
  version_number,
  previous_version_hash,
  H(audio_ciphertext or audio_plaintext),   -- open decision, see below
  H(transcript_final),
  H(consent_log),
  party_signatures,
  salt
)
```

Open decision: hash the plaintext or the ciphertext? Plaintext hash proves content regardless of how it was encrypted, which is what a court cares about. It requires salting (done). Recommendation: hash plaintext with salt.

### 9.3 Contract shape

Minimal and boring on purpose:

```
contract AnchorRegistry {
  event RootAnchored(bytes32 root, uint256 batchId, uint256 timestamp);
  function anchor(bytes32 root) external onlyAnchorer;
  function rootOf(uint256 batchId) external view returns (bytes32);
}
```

- No personal data, no addresses of members, no per-commitment entries.
- Not upgradeable for the stored data. Roots once written are final.
- The anchorer key is held by the anchoring service, in a hardware-backed key store, with a rotation process.
- Open-source the contract and the verifier.

### 9.4 Why not put more logic on-chain

Smart contracts with per-commitment state, party addresses, and status on-chain would make relationships between people public forever (anyone could see that wallet X and wallet Y share 40 commitments). For a fun, private app that is the wrong trade. Contracts mode may justify more on-chain logic later, for example escrowed obligations. That is a separate design.

### 9.5 Optional redundancy

Anchor the same root to a second chain or to a timestamping service periodically, so the proof survives if one chain falters. Low cost, high credibility. Phase 2.

---

## 10. Wireframes (v1 screen list)

Text wireframes for structure. A clickable visual prototype is the next step.

### 10.1 Home

```
+----------------------------------+
| On The Block            (bell) (o)|
|----------------------------------|
| Needs you (2)                    |
|  [Sam wants to seal "Holidays"]  |
|  [Check-in due: "Gym 3x/week"]   |
|----------------------------------|
| Your commitments                 |
|  Holidays plan      Sealed  Sam  |
|  Gym 3x/week        Active  Me   |
|  No phones at dinner Kept  Sam   |
|----------------------------------|
| From your connections            |
|  Jordan pledged: Marathon 2027   |
|----------------------------------|
|        [ + Put it on the block ] |
| Home   Connections   Profile     |
+----------------------------------+
```

### 10.2 New commitment

```
+----------------------------------+
| < New commitment                 |
|----------------------------------|
| What is it?                      |
| [ Who handles the holidays     ] |
|                                  |
| Who's in?                        |
|  (o) Just me                     |
|  (*) Me and...  [ Sam      v ]   |
|                                  |
| How?                             |
|  (*) Record it                   |
|  ( ) Type it                     |
|                                  |
| Remind us to check in            |
|  [ Monthly            v ]        |
|                                  |
|            [ Start session ]     |
+----------------------------------+
```

### 10.3 Live session (consent)

```
+----------------------------------+
| Session with Sam                 |
|----------------------------------|
|  Alex  ......... consented (ok)  |
|  Sam   ......... waiting         |
|                                  |
|  [ QR code for Sam to scan ]     |
|                                  |
| Recording starts when everyone   |
| consents. Anyone can stop it.    |
|                                  |
|        [ Cancel session ]        |
+----------------------------------+
```

### 10.4 Recording

```
+----------------------------------+
| Recording   02:41       (stop)   |
|----------------------------------|
|   ~~~~~~~~ waveform ~~~~~~~~     |
|                                  |
| Tip: close with each of you      |
| saying "I agree."                |
|                                  |
| Alex (ok)   Sam (ok)             |
+----------------------------------+
```

### 10.5 Transcript review

```
+----------------------------------+
| < Review transcript              |
|----------------------------------|
| AI transcript. Audio is the      |
| official record.                 |
|                                  |
| Alex: Are you willing to put     |
|   that on the block?             |
| Sam:  Yes. My parents get        |
|   Thanksgiving, yours get        |
|   Christmas.             [edit]  |
| Alex: I agree.                   |
| Sam:  I agree.                   |
|                                  |
| Sam: approved   Alex: pending    |
|                                  |
| [ Remove a segment ] [Approve]   |
+----------------------------------+
```

### 10.6 Seal

```
+----------------------------------+
| Seal it                          |
|----------------------------------|
| Who handles the holidays         |
| Alex and Sam, Nov 7, 2026, 2:41  |
|                                  |
| A personal record, not a legal   |
| contract.                        |
|                                  |
| Once sealed, this version cannot |
| be changed. You can amend it     |
| later with Sam's agreement.      |
|                                  |
|      [ Seal with Face ID ]       |
+----------------------------------+
```

### 10.7 Commitment detail

```
+----------------------------------+
| < Who handles the holidays       |
|----------------------------------|
| On the block (ok)  v2  Sam & Me  |
| Sealed Nov 7, 2026               |
| [ Play ] [ Transcript ] [Verify] |
|----------------------------------|
| History                          |
|  v2  Amended Dec 1  (diff)       |
|  v1  Sealed Nov 7                |
|----------------------------------|
| How's it holding up?             |
|  You: 5/5   Sam: 4/5             |
|  [ Check in ]                    |
|----------------------------------|
| Access: Sam, Me                  |
|  [ Share with someone ]          |
|----------------------------------|
| [ Amend ] [ Mark done ] [ ... ]  |
+----------------------------------+
```

### 10.8 Verify

```
+----------------------------------+
| < Verify                         |
|----------------------------------|
|  (ok) Audio matches              |
|  (ok) Transcript matches         |
|  (ok) Signed by Alex and Sam     |
|  (ok) Anchored on Base           |
|       Block 23,481,227           |
|       Nov 7, 2026, 2:58 PM EST   |
|                                  |
|  Unchanged since it was sealed.  |
|                                  |
|  [ View transaction ]            |
|  [ Export proof bundle ]         |
+----------------------------------+
```

### 10.9 Other screens

Connections list and requests; member profile (with visibility rules); circles; notifications; settings (devices, recovery, privacy, export, delete account, safety resources); web: landing page, sign-in, dashboard mirroring mobile, public commitment pages, standalone verifier.

---

## 11. Path to contracts mode (later)

What has to be added, not rebuilt:
- Strong identity (ID verification) for all parties.
- E-signature compliance package: intent-to-sign language, consumer consent to electronic records, record retention, audit trail. ESIGN Act and UETA (UNVERIFIED; confirm requirements).
- Templates and clause libraries, possibly attorney-drafted.
- Optional counsel review and a role for a lawyer as a party or witness.
- Jurisdiction and governing law selection.
- Possibly on-chain logic for escrow or conditional obligations, as a separate contract.
- Carve-outs: family law agreements, wills, real estate, and other categories with formal execution requirements.

The v1 data model (versions, signed consent log, amendment chain, access grants) already supports this. The main additions are identity, compliance text, and templates.

---

## 12. Open decisions for Enrico

1. Product name.
2. End-to-end encryption (recommended) or platform-decryptable.
3. ID verification in v1: optional badge (recommended) or required for shared commitments.
4. Record on one phone or both phones.
5. Remote audio sessions in v1 or phase 2 (recommended phase 2).
6. Abandoned recordings: delete after 7 days, or let each party keep a private copy.
7. Can one party export their own copy without the other's approval (recommended yes, with notice).
8. Can one party delete their copy without deleting the other's (recommended yes).
9. Chain choice: Base (recommended) or other.
10. Business model: subscription, per-anchor fee, or free tier with paid features.
11. Launch geography: U.S. only at first (recommended), which also limits privacy and recording-law scope.
12. Whether this repo (currently the GitHub profile repo) is the long-term home, or we create a dedicated repo before coding.

---

## 13. Proposed phases

Phase 0 (now): design doc, open decisions, clickable wireframe prototype, legal review list.
Phase 1 (MVP): accounts, connections, personal and shared in-person commitments (record and typed), transcript review, sealing, batch anchoring on a testnet then mainnet, verify screen, private ratings and check-ins, export.
Phase 2: remote sessions, amendments with diff, public commitments and feed, circles, second-chain anchoring, web verifier as standalone static site.
Phase 3: contracts mode.

---

## 14. Legal review list (all UNVERIFIED until checked against primary sources)

- State all-party-consent recording statutes, and how they apply when both parties consent in-app.
- Privacy laws: CCPA/CPRA, other state comprehensive privacy laws, and GDPR if any EU users; how crypto-shredding satisfies deletion rights.
- Biometric privacy (Illinois BIPA and similar) if voice prints or face data are processed. Avoid voiceprint features in v1.
- Enforceability of informal agreements between spouses and partners; when a "fun" record could be argued to be a contract.
- ESIGN and UETA for contracts mode.
- Money transmission and securities: confirm that platform-paid gas with no token avoids both.
- App Store and Google Play rules for recording, crypto, and user-generated content.
- Terms of use, privacy policy, records-request (subpoena) policy, and trust-and-safety policy.
