# Put It On The Block: Design Document

Status: Draft 2, for discussion. No code yet.
Owner: Enrico Schaefer (private project, not Traverse work)
Working name: "On The Block" (placeholder; naming is an open decision)

Change log
- Draft 1 (2026-11-07): initial issues, model, architecture, wireframes.
- Draft 2 (2026-11-07): incorporated Enrico's decisions. Core object generalized from "commitment" to "Entry" with types, including Calls (predictions and claims). Binding option moved into v1 as a mutual election. One-to-one only. Mobile-first creation and sealing. Public accountability score with per-entry opt-in. Visibility controlled by the parties. Deletion setting chosen at intake. Typed entries are first-class.

---

## 1. What we are building

People say things that matter and then forget them, misremember them, or dispute them. Contracts go in a drawer or on a file server. Promises between spouses and friends depend on memory. Friendly arguments ("I'm telling you, the Lions win by 5") disappear once spoken.

On The Block lets two members, or one member alone, memorialize what was said or agreed and anchor proof of it on a public blockchain. The record is consented, reviewed by every party, sealed with each party's signature, and verifiable later by anyone the parties allow. That adds a level of certainty and accountability that ordinary conversation does not have.

The content (audio, transcript, or typed text) lives off-chain, encrypted. The chain holds only a fingerprint (salted hash) and a timestamp.

The product launches as a consumer app with a social layer. It is built to evidentiary standards from day one because people will use it in disputes. Parties can mutually elect to make an entry legally binding.

The platform is built for flexibility. Entries have types, and the type list will grow or shrink based on what users actually do. We measure use by type (metadata only, not content) and pivot marketing toward what takes off.

### Design principles

1. Member to member, with consent at every step. Every party is a member who consents on their own device. No outside recordings, no non-members.
2. Nothing personal goes on-chain. Hashes and timestamps only.
3. The parties control the record: who sees it, whether it is public, whether it counts toward a public score, whether either can delete it, and whether it is binding.
4. Verifiable without us. Proofs work even if the company is gone.
5. Fun first, built to evidence standards.

---

## 2. Issues and how we resolved them

### 2.1 Immutability vs. deletion and privacy law (decided)

Only a salted hash goes on-chain. Content and keys are off-chain. Deleting content and destroying keys ("crypto-shredding") leaves an on-chain hash that points to nothing. The salt prevents someone from guessing a short entry and confirming it against the chain.

Deletion setting is chosen at intake, per entry:
- Default: each party can delete their own copy at any time. The other party keeps theirs.
- Option: "Neither of us can delete without the other." Deletion of either copy requires both parties.
- Full destruction of the content happens when all parties delete, or when the last remaining party deletes.

For binding entries the default may need to be the locked option, so neither party can destroy the record of a contract unilaterally. See open decisions.

Privacy law still gives a user deletion rights over their own personal data. A locked setting has to be reconciled with that. Legal review item.

### 2.2 Recording consent laws (decided)

This is not a tool for recording conversations. It records a deliberate entry between members who have each consented in-app. Recording starts only after every party taps consent on their own device. A spoken notice opens the recording, and each party's verbal yes is captured. Terms of service cover the remaining obligations, including bystanders. Transcript review allows removal of a segment before sealing.

State all-party-consent statutes still go on the legal review list (UNVERIFIED).

### 2.3 Binding entries (decided: mutual election)

Every entry is non-binding by default. At intake, the parties can elect "Make this binding." Both must elect it. If either declines, the entry is non-binding.

A binding entry adds:
- Identity verification for both parties (see open decisions for the level).
- Electronic signature consent and intent-to-sign language at sealing. ESIGN Act and UETA requirements (UNVERIFIED; confirm).
- Optional governing law field.
- Stronger record defaults: locked deletion, export available to each party.
- A warning screen for categories that often carry formal requirements or limits: real estate, wills and estates, marital property and postnuptial terms, custody and support, and employment. A warning, not a block. Whether a given entry is enforceable is a question of state law we do not answer for the user.

Non-binding entries carry the line "A personal record, not a legal contract" at sealing.

Agreements between spouses raise state-specific rules on formalities, disclosure, and public policy (UNVERIFIED). Legal review item.

### 2.4 Subpoenas and evidence (decided: end-to-end encryption)

Content is end-to-end encrypted. We hold ciphertext and metadata only. When compelled, we can produce metadata (who, when, status, hashes) but not content. The parties hold the content and can be compelled themselves.

Evidence package: device signatures from each party, a signed consent log, hashes of audio and transcript, an on-chain timestamp, and a published, open-source verifier. We publish a plain-English page on authenticating an On The Block record. We also need a records-request policy.

### 2.5 Pressure inside relationships (decided)

The concern is one person pressuring another into an entry. Mitigations:
- Either party can decline at any stage before sealing. The notice is neutral ("Not accepted").
- Rate limits on repeat requests to the same person.
- Block, mute, and report on every connection.
- Each person decides, per entry, whether it counts toward their public score.
- A private "I was pressured" flag, visible to the flagging party and trust and safety.
- Safety resources link in settings.
- Terms of service address it.

### 2.6 AI transcript accuracy (decided)

Audio is the canonical record. Transcription runs on the phone, with a server fallback that decrypts in memory for the job only and keeps nothing. Each party reviews, edits, and accepts the transcript. Edits must be accepted by both. Both audio and final transcript are hashed.

### 2.7 Identity (decided)

Proof of who comes from account verification, device-bound signing keys (passkeys / secure enclave), and the voice on the audio. Optional "ID verified" badge for non-binding use; required for binding entries.

### 2.8 Crypto exposure and business model (decided)

No token, no NFT, no coin. The platform pays gas. Revenue is a low-cost app subscription priced to be profitable after gas, storage, transcription fallback, and app store fees. Users never see a wallet or seed phrase.

### 2.9 Scope limits (decided)

- 18+ only.
- One-to-one entries, or entries with yourself. No groups in v1.
- Estate access through a documented process. Surviving party keeps access.
- Content moderation works on reports. Reporters share their own decrypted copy if they choose.

### 2.10 New issue: Calls and wagers

Calls ("Lions by 5") are a core fun use. If users attach stakes, gambling law and app store rules come into play.

Proposed: stakes are free text only ("Loser buys dinner"). The platform never holds, transfers, or settles money or anything of value on an outcome. Legal review item (UNVERIFIED) on whether free-text stakes between members create any exposure.

### 2.11 New issue: public accountability scores

A public score is the feature that creates accountability. It is also where harm can come from: a counterparty tanking someone's score, or a score being gamed.

Design (Section 8.2): a person's public score uses only entries that person opted in. Disputed outcomes are shown as disputed and do not move the score. Scores show their basis (number of entries counted). Legal review item on defamation exposure for ratings, though the opt-in design reduces it.

### 2.12 Other items to track

- Data residency if we launch outside the U.S.
- Poor audio environments.
- Accessibility: typed entries are first-class, so anyone can use the product without audio.

---

## 3. Core concepts (domain model)

| Concept | What it is |
|---|---|
| Member | A verified user account. Profile, device signing key, encryption key pair. |
| Connection | A mutual relationship between two members. Required before a shared entry. |
| Entry | The core record. Has a type, one or two parties, settings, and versions. |
| Entry type | Promise, Agreement, Call, Statement, or Personal Goal. Types drive templates, outcome options, and wording. The list is data, not code, so we can add or retire types. |
| Party | A member's role on an entry. Each party has their own consent, settings, access, and rating. |
| Entry settings | Chosen at intake: binding (mutual), visibility (Section 8.1), counts toward my score (per party), deletion rule, check-in schedule. |
| Session | The live window where parties join, consent, and record or type. |
| Recording | Encrypted audio. Canonical when present. |
| Text | Encrypted transcript or typed text, approved by all parties. |
| Consent event | Signed, timestamped yes or no at each stage. |
| Version | A sealed state of an entry. An amendment is a new version linked to the prior one. |
| Seal | All parties sign the version hash on their phones. Triggers anchoring. |
| Anchor | The version hash included in a Merkle batch; the root is written on-chain. |
| Outcome | How an entry turned out: kept or broken (Promise, Agreement, Goal), right or wrong or push (Call), none (Statement). |
| Access grant | Permission for a non-party to view. Requires both parties. |
| Check-in | A periodic report on how an entry is holding up. |
| Rating | A party's assessment of how the other party kept an entry. |
| Accountability score | A member's public score, built only from entries they opted in. |

### 3.1 Entry types (v1)

| Type | Example | Parties | Outcome |
|---|---|---|---|
| Promise | "I'll handle the holidays this year." | 1 or 2 | Kept / broken |
| Agreement | "Your parents get Thanksgiving, mine get Christmas." | 2 | Kept / broken, per party |
| Call | "Lions win by 5." "The restaurant was on Elm Street." | 1 or 2 | Right / wrong / push |
| Statement | "Here's what we decided about the car." | 1 or 2 | None; it is a memorialized record |
| Personal Goal | "Gym three times a week through March." | 1 | Kept / broken |

Calls between two people can be opposed ("I say Lions by 5; Sam says Bears") or a single claim the other party witnesses.

---

## 4. Entry lifecycle

```
DRAFT ---> AWAITING CONSENT ---> CAPTURE ---> REVIEW ---> SEALING ---> SEALED ---> ANCHORED
            (other party          (record      (edit and     (both sign    (queued)    (on chain)
             accepts invite        audio or     approve       on phone)
             and settings)         type text)   text)
                                                                                         |
                       +--------------+-------------+---------------+-------------------+------------+
                       |              |             |               |                   |            |
                    AMENDED       FULFILLED      RELEASED        BROKEN             RESOLVED      EXPIRED
                    (new version  (both mark     (both agree     (one party marks;  (Calls:       (end date
                     through the   kept)          to end)         other may          right/wrong/  passed)
                     same flow)                                   dispute)           push)
```

Rules:
- Either party can abandon before SEALED.
- After ANCHORED, nothing is edited. Changes are new versions linked to the prior version's hash.
- Outcomes that both parties agree on are final. A one-sided outcome shows as "Marked broken by Alex; Sam disputes." We do not adjudicate.
- Status changes after anchoring are signed events, anchored in the next batch.
- Settings agreed at intake are part of the sealed version. Changing them (for example, making an entry public later) is a mutual, signed event.

Personal entries follow the same flow with one party.

---

## 5. Key flows

Creation and sealing happen in the mobile app only. The website is for viewing, verifying, profiles, connections, public entry pages, account, export, and the standalone verifier.

### 5.1 Connecting

Find by username, opt-in hashed contact matching, or QR code in person. Send a request; the other member accepts or ignores. Connection is required before a shared entry.

### 5.2 Creating a shared entry in person (origin case)

1. Initiator taps "Put it on the block," picks the type, picks the other party, adds a title.
2. Initiator proposes settings: binding or not, visibility, deletion rule, check-ins. Initiator chooses whether it counts toward their own score.
3. Other party joins the session on their own phone (QR or push), reviews proposed settings, accepts or counters, and chooses whether it counts toward their own score.
4. Both consent to capture.
5. Capture: record audio (spoken notice plays first, prompt to close with "I agree") or co-type the text.
6. Review: transcript generated on device; each party edits, can remove a segment, and approves. Edits require both.
7. Seal: summary of entry, settings, and the binding or non-binding line. Each party seals with biometrics. Binding entries add the e-signature consent and intent language.
8. "Sealed, anchoring," then "On the block" with a proof link.

### 5.3 Making a Call

Same flow, lighter. Type "Call," state the claim (and the other party's counter-claim if opposed), optional free-text stakes, optional resolution date. At resolution, both parties mark the outcome. Disagreement shows as disputed.

### 5.4 Personal entry

One party. Pick type, capture, review, seal. Choose visibility and whether it counts toward your score.

### 5.5 Remote entries

Typed entries work remotely in v1: both parties co-edit from their own phones. Remote audio (in-app call recording) is an open decision.

### 5.6 Amending

Either party proposes an amendment. It goes through consent, capture, review, and seal. Diff view shows changes.

### 5.7 Checking the record

"Verify" decrypts on device, recomputes hashes, fetches the Merkle proof and on-chain transaction, and shows "Verified: unchanged since [date, time], block [number]." The website verifier does the same with an exported proof bundle, without our servers.

### 5.8 Sharing outside the entry

Access grants to a specific person require both parties. View-only, expiring, revocable. Every view logged and visible to the parties.

---

## 6. System architecture

```
  +----------------+       +------------------+
  |  Mobile app    |       |   Website        |
  |  create, seal, |       |  view, verify,   |
  |  record, sign  |       |  profiles, public|
  +-------+--------+       |  pages, account  |
          |                +--------+---------+
          |  device keys, E2E encryption, on-device transcription
          +----------------+-----------------+
                           |
                    HTTPS / WebSocket
                           |
  +------------------------v--------------------------+
  |                    API layer                      |
  |  auth, members, connections, entries, sessions,   |
  |  consent log, grants, outcomes, scores, notify    |
  +---+-----------+------------+------------+---------+
      |           |            |            |
+-----v----+ +----v-----+ +----v-------+ +--v-------------+
| Postgres | | Object   | | Transcribe | | Anchoring      |
| metadata | | storage  | | fallback   | | service        |
| (no      | | encrypted| | (ephemeral,| | batch -> Merkle|
| content) | | blobs    | |  no retain)| | root -> chain  |
+----------+ +----------+ +------------+ +--+-------------+
                                            |
                                     +------v---------+
                                     | L2 blockchain  |
                                     | Anchor registry|
                                     +----------------+
```

### 6.1 Proposed stack

| Layer | Proposal | Why |
|---|---|---|
| Mobile | React Native (Expo) | One codebase for iOS and Android; TypeScript shared with web. |
| Web | Next.js | Same language and components; verifier can be fully static. |
| API | TypeScript (Node) | Same language end to end. |
| Database | Postgres | Relational data fits. |
| File storage | S3-compatible | Ciphertext only. |
| Transcription | On-device Whisper-class model; server fallback | Privacy first. |
| Chain | Ethereum L2 (Base leading candidate) | Low cost, EVM tooling, public. |
| Signing keys | Passkeys / secure enclave | No seed phrases. |

### 6.2 What the server can and cannot see

Sees: accounts, connections, who is party to which entry, entry type, settings, timestamps, status, outcomes, scores, hashes.
Cannot see: audio, text, titles, amendment content.

Entry type and outcome are metadata on purpose. They power scores and the usage analytics we need to decide where to take the product, without exposing content.

### 6.3 Flexibility for pivots

- Entry types, their outcome options, templates, and wording are configuration, not hard-coded screens.
- Feature flags for social features (feed, public pages, leaderboards) so we can test them by cohort.
- Usage analytics by type, settings chosen, and outcome, never by content.

---

## 7. Encryption and keys

### 7.1 Per-entry content keys

Each version gets a random content key. Content is encrypted on device. The key is wrapped separately for each party. Access grants re-wrap for the grantee after both parties approve. Revocation deletes the grantee's wrapped key. Deletion follows the entry's deletion rule (Section 2.1); full crypto-shredding deletes all wrapped keys and the ciphertext.

### 7.2 Member keys

Signing key (device-bound), encryption key pair, multi-device enrollment from an existing device.

### 7.3 Recovery

Options: another of the member's devices (default), recovery code, trusted connection holding a recovery share, or opt-in platform escrow labeled as weaker privacy. In a shared entry, the other party can re-grant access.

---

## 8. Social layer and accountability

The user controls exposure. Default is private between the parties. Public is a choice, the way some payment apps let users choose whether a transaction shows on a public feed.

### 8.1 Visibility of an entry

| Level | Who sees it |
|---|---|
| Private (default) | The parties only. |
| Connections | Parties plus their connections. |
| Public | Anyone, including a public web page and share card. |

For a shared entry, the more restrictive choice wins. Either party can lower visibility later; raising it requires both.

### 8.2 Accountability score

- Each member has a public accountability score. Only entries they opted in count. The opt-in is per entry, per party, chosen at intake, and is independent of visibility. A private entry can still count toward the score; the score shows only the aggregate.
- Promise, Agreement, and Goal outcomes feed the accountability score: kept vs. broken.
- Calls feed a separate "Call record": right, wrong, push.
- For shared entries, an outcome counts only when both parties agree on it. Disputed outcomes are shown as a disputed count and do not move the score.
- Personal Goals are self-reported. Shown as self-reported.
- The score displays its basis ("Kept 18 of 20 counted entries").
- Members can hide their score entirely.

### 8.3 Ratings

Each party can rate how the other kept a shared entry (1 to 5, optional note). Ratings are visible to the parties. A rating affects the public score only through the outcome on an entry the rated person opted in.

### 8.4 Feed and fun

Feed shows pending actions, upcoming check-ins and resolution dates, public entries from connections, and milestones ("Alex called it: Lions by 7"). Milestones for shared entries post only if both parties allow.

---

## 9. Smart contract and anchoring

### 9.1 Batch anchoring

Sealed versions queue. Every N minutes the anchoring service builds a Merkle tree of version hashes and writes one root on-chain. Each version keeps its Merkle proof. One transaction per batch.

### 9.2 What gets hashed

```
version_hash = H(
  entry_id_salted,
  version_number,
  previous_version_hash,
  entry_type,
  settings,                  -- binding, visibility, deletion rule, score opt-ins
  H(audio_plaintext),        -- if audio present
  H(text_final),
  H(consent_log),
  party_signatures,
  salt
)
```

Plaintext hashes with salt, so the proof is about the content, not the encryption.

### 9.3 Contract shape

```
contract AnchorRegistry {
  event RootAnchored(bytes32 root, uint256 batchId, uint256 timestamp);
  function anchor(bytes32 root) external onlyAnchorer;
  function rootOf(uint256 batchId) external view returns (bytes32);
}
```

No personal data, no member addresses, no per-entry state on-chain. Roots are final. Anchorer key in a hardware-backed store with rotation. Contract and verifier open-sourced.

### 9.4 Why logic stays off-chain

Per-entry on-chain state would publicly link people forever. Binding entries do not need on-chain logic to be enforceable; they need good evidence, which the anchor provides. On-chain logic (escrow, conditional obligations) can be a later, separate contract.

### 9.5 Redundancy (phase 2)

Periodically anchor the same roots to a second chain or timestamping service.

---

## 10. Wireframes (v1 mobile screens)

Text wireframes for structure. A clickable visual prototype is the next step.

### 10.1 Home

```
+----------------------------------+
| On The Block            (bell) (o)|
|----------------------------------|
| Accountability 18/20   Calls 7-3 |
|----------------------------------|
| Needs you (2)                    |
|  [Sam wants to seal "Holidays"]  |
|  [Resolve: "Lions by 5" (Sun)]   |
|----------------------------------|
| Your entries                     |
|  Holidays plan   Agreement  Sam  |
|  Lions by 5      Call       Sam  |
|  Gym 3x/week     Goal       Me   |
|----------------------------------|
| From your connections            |
|  Jordan called it: Rain on Sat   |
|----------------------------------|
|        [ + Put it on the block ] |
| Home   Connections   Profile     |
+----------------------------------+
```

### 10.2 New entry: type

```
+----------------------------------+
| < Put it on the block            |
|----------------------------------|
| What kind?                       |
|  [ Promise   ]  I will...        |
|  [ Agreement ]  We agree...      |
|  [ Call      ]  I say...         |
|  [ Statement ]  For the record...|
|  [ Goal      ]  Just me          |
+----------------------------------+
```

### 10.3 New entry: details and settings

```
+----------------------------------+
| < Agreement                      |
|----------------------------------|
| Title [ Who handles holidays   ] |
| With  [ Sam              v ]     |
| How   (*) Record  ( ) Type       |
|----------------------------------|
| Settings (Sam will see these)    |
| Visibility   [ Private     v ]   |
| Binding      [ No          v ]   |
| Deleting     [ Each can delete v]|
| Check in     [ Monthly     v ]   |
|----------------------------------|
| [x] Count toward my score        |
|                                  |
|            [ Start session ]     |
+----------------------------------+
```

### 10.4 Live session (other party joins)

```
+----------------------------------+
| Session with Alex                |
|----------------------------------|
| Alex proposes:                   |
|  Private . Not binding .         |
|  Each can delete . Monthly       |
|  [ Accept ]  [ Change ]          |
|                                  |
| [x] Count toward my score        |
|                                  |
| Alex ..... consented (ok)        |
| You  ..... [ I consent ]         |
|                                  |
|        [ Decline ]               |
+----------------------------------+
```

### 10.5 Recording

```
+----------------------------------+
| Recording   02:41       (stop)   |
|----------------------------------|
|   ~~~~~~~~ waveform ~~~~~~~~     |
| Tip: close with each of you      |
| saying "I agree."                |
| Alex (ok)   Sam (ok)             |
+----------------------------------+
```

### 10.6 Review

```
+----------------------------------+
| < Review                         |
|----------------------------------|
| AI transcript. Audio is the      |
| official record.                 |
|                                  |
| Alex: Are you willing to put     |
|   that on the block?             |
| Sam:  Yes. My parents get        |
|   Thanksgiving, yours get        |
|   Christmas.             [edit]  |
| Alex: I agree.  Sam: I agree.    |
|                                  |
| Sam: approved   Alex: pending    |
| [ Remove a segment ] [Approve]   |
+----------------------------------+
```

### 10.7 Seal (non-binding / binding)

```
+----------------------------------+   +----------------------------------+
| Seal it                          |   | Seal it (binding)                |
|----------------------------------|   |----------------------------------|
| Who handles the holidays         |   | Sale of boat                     |
| Alex and Sam, Nov 7, 2026        |   | Alex and Sam, ID verified        |
| Private . Each can delete        |   | Locked deletion                  |
|                                  |   |                                  |
| A personal record, not a legal   |   | You both chose to make this      |
| contract.                        |   | legally binding. By sealing you  |
|                                  |   | sign it electronically.          |
| Sealed versions cannot change.   |   | [ Review e-sign disclosure ]     |
| Amend later with Sam.            |   |                                  |
|   [ Seal with Face ID ]          |   |   [ Sign and seal ]              |
+----------------------------------+   +----------------------------------+
```

### 10.8 Entry detail

```
+----------------------------------+
| < Who handles the holidays       |
|----------------------------------|
| On the block (ok) v2  Agreement  |
| Private . Not binding            |
| [ Play ] [ Text ] [ Verify ]     |
|----------------------------------|
| History  v2 Amended Dec 1 (diff) |
|          v1 Sealed Nov 7         |
|----------------------------------|
| Holding up?  You 5/5  Sam 4/5    |
| [ Check in ]                     |
|----------------------------------|
| [ Amend ] [ Mark kept ] [ ... ]  |
+----------------------------------+
```

### 10.9 Call detail

```
+----------------------------------+
| < Lions by 5                     |
|----------------------------------|
| Call . Sam vs. Alex . Public     |
| Alex: Lions by 5                 |
| Sam:  Bears win                  |
| Stakes: loser buys dinner        |
| Resolves: Sun, Nov 15            |
|----------------------------------|
| Who called it?                   |
| [ Alex ] [ Sam ] [ Push ]        |
| Alex says: Alex  Sam: pending    |
+----------------------------------+
```

### 10.10 Verify

```
+----------------------------------+
| < Verify                         |
|----------------------------------|
|  (ok) Audio matches              |
|  (ok) Text matches               |
|  (ok) Signed by Alex and Sam     |
|  (ok) Anchored on Base           |
|       Block 23,481,227           |
|       Nov 7, 2026, 2:58 PM EST   |
|  Unchanged since it was sealed.  |
|  [ View transaction ]            |
|  [ Export proof bundle ]         |
+----------------------------------+
```

### 10.11 Profile

```
+----------------------------------+
| Alex R.          (ID verified)   |
| Member since 2026                |
|----------------------------------|
| Accountability  18 of 20 kept    |
|   2 disputed, not counted        |
| Call record     7-3-1            |
|----------------------------------|
| Public entries                   |
|  Marathon 2027 (Goal)            |
|  Lions by 5 (Call) - called it   |
|----------------------------------|
| [ Connect ]                      |
+----------------------------------+
```

### 10.12 Other screens

Connections and requests; notifications; settings (devices, recovery, default visibility, default score opt-in, hide score, export, delete account, safety resources). Website: landing page, sign-in, dashboard, public entry pages, public profiles, standalone verifier.

---

## 11. Binding entries: what v1 must include

- ID verification for both parties.
- E-signature consumer disclosure and intent-to-sign at sealing (ESIGN / UETA, UNVERIFIED).
- Governing law field (optional).
- Category warnings (real estate, wills and estates, marital property and postnuptial terms, custody and support, employment).
- Locked deletion default.
- Export of a complete record bundle by either party at any time.

Later: templates and clause libraries, attorney review option, on-chain escrow.

---

## 12. Decisions made (2026-11-07)

| # | Decision |
|---|---|
| D1 | Files off-chain, encrypted. Only salted hashes on-chain via batch Merkle roots. |
| D2 | All parties are members; consent on their own device at every stage. No outside recordings. |
| D3 | Deletion: default each party can delete their own copy; parties can choose locked at intake. Full destruction when all parties, or the last party, delete. |
| D4 | Binding is a mutual election at intake. Both must agree. Default is non-binding. |
| D5 | End-to-end encryption (Option B). |
| D6 | Pressure safeguards per Section 2.5, plus terms of service. |
| D7 | On-device transcription, server fallback. Each party reviews and accepts. Typed entries are first-class. |
| D8 | Identity per Section 2.7. |
| D9 | No token. Platform pays gas. Low-cost subscription. |
| D10 | 18+. One-to-one or self only. Estate access by documented process. Report-based moderation. |
| D11 | Visibility controlled by the parties; private by default; public by choice. |
| D12 | Public accountability score with per-entry, per-party opt-in. |
| D13 | Entry types including Calls. Types are configurable to allow pivots. |
| D14 | Creation and sealing on mobile. Website for viewing, verifying, profiles, and account. |

---

## 13. Open decisions

Taken in this order, one at a time.

1. Binding entries: level of ID verification required, and whether locked deletion is mandatory or just the default.
2. ID verification for non-binding entries: optional badge (recommended) or required.
3. Visibility conflicts: more restrictive wins, and either party can lower but raising needs both (recommended).
4. Disputed outcomes: excluded from score and shown as a disputed count (recommended), or another rule.
5. Calls: free-text stakes only, no money handled (recommended).
6. Export: can a party export their own copy of a non-binding entry without the other's approval (recommended yes, with notice).
7. Capture device: record on one phone or both phones.
8. Remote audio sessions: v1 or phase 2 (recommended phase 2; remote typed entries in v1).
9. Abandoned drafts: delete after 7 days, or each party may keep a private copy.
10. Chain: Base (recommended) or other.
11. Launch geography: U.S. only (recommended).
12. Product name.
13. Repo: keep in this profile repo, or create a dedicated repo before coding (recommended dedicated).

---

## 14. Phases

Phase 0 (now): design doc, open decisions, clickable prototype, legal review list.
Phase 1 (MVP): accounts, connections, all five entry types, in-person record and typed capture, remote typed entries, review, sealing, binding election with ID verification and e-sign, batch anchoring (testnet then mainnet), verify, visibility, accountability score and Call record, ratings, check-ins, export, website (view, verify, profiles, public pages).
Phase 2: remote audio sessions, feed expansion, second-chain anchoring, standalone static verifier, templates for binding entries.
Phase 3: groups, on-chain escrow, attorney review marketplace.

---

## 15. Legal review list (all UNVERIFIED until checked against primary sources)

- State all-party-consent recording statutes as applied to in-app consented entries.
- Privacy laws (CCPA/CPRA, other state laws, GDPR if applicable); crypto-shredding as deletion; reconciling locked deletion with deletion rights.
- Biometric privacy (BIPA and similar). No voiceprint features in v1.
- ESIGN and UETA requirements for binding entries; consumer disclosure.
- Enforceability limits for agreements between spouses and partners and other category warnings.
- Gambling law and app store rules as applied to Calls with free-text stakes.
- Defamation and consumer protection exposure from public scores and ratings.
- Money transmission and securities: confirm platform-paid gas with no token avoids both.
- App Store and Google Play rules for recording, crypto, and user-generated content.
- Terms of service, privacy policy, records-request policy, trust and safety policy.
