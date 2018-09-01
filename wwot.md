# Weighted Webs of Trust


##  Goal

    The initial problem the Weighted Webs of Trust is to classify software packages according to various subjective criteria in a meaningful way.

    The problem of reviews published for a given package is whether they are trustworthy, or not, and to which degree:

    -   Even experts may not be entirely objective, may make mistake, or may have agendas.
    -   Yet a single expert's opinion may be much more meaningful than that of a hundred developers; especially on specialty domains such as security.

    Today, one is left to either:

    -   Pick a handful of experts to trust, and stick to their recommendations.
    -   Trust in the wisdom of the masses, and use the "average" or "median" score.

    Ideally, one would like to review each reviewer, or cross-check them, but the sheer effort required is not sustainable.


    The goal of the Weighted Web of Trusts is to provide the casual user with a simple way to express trust in a handful of Webs, trusting in their founders to preserve the quality of the reviewers in their web, and immediately benefit from an extensive network of reviewers, with a weight associated to each reviewer proportional to the trust that their Webs has in them.

    This allows creating weighted averages and weighted distributions out of the reviews of a given package reflecting the trust invested in each Weighted Web of Trust, and thus in their respective founders.


    There may be other usages for those Weighted Webs of Trust available to the curious.


##  Quick Glossary

    A quick glossary of the technical terms described below.

    -   Weighted Web of Trust (WWoT): a Web of Trust where each participant is associated to a weight used in the weighted average mechanism.
    -   Ledger of Trust (LoT): an append-only public ledger recording the weight of each participant in a specific WWoT; the materialization of the WWoT concept.
    -   Root of Trust (RoT): the root participant of a specific WWoT.
    -   Fraction of Weight (FoW): the fraction of one's own weight that is delegated to another participant.


##  Concept of Trust: Weighted Web of Trust (WWoT)

    A Weighted Web of Trust is a Web of Trust where each participant is associated a weight. It is governed by a number of rules, exposed here.


### Sum rule.

    The first governing principle of the Weighted Web of Trust is that at any point in time, the sum of the weights of each participant in the WWoT is equal to 1.

    This makes it possible to use multiple independent WWoT as a source of trust and easily mix in their weights into a single formula:

    -   compute the Weighted Average for each WWoT independently.
    -   compute the Weighted Average of all WWoT's Weighted Averages.


### Dilution rule.

    The second governing principle of the Weighted Web of Trust is that endorsement cost the endorser.

    That is, when endorsing either a new participant or an existing participant, the endorser must lend part of its weight, representing the amount of trust they have in the other participant.

    This dilutes the power of anyone endorsing other, ensuring that no matter how many individuals they invite in which share their point of view, they can never gain more influence as a result.


    And that's it for the concept. It's really simple, which is its own beauty!



##  Implementation of Trust: Ledger of Trust (LoT)

    A Ledger of Trust is a practical implementation of the Weighted Web of Trust concept.


### Properties

    The Ledger of Trust should be:

    -   authenticated: it should not be possible to record actions on behalf of other participants.
    -   faithful: it should not be possible to rewrite the history of the WWoT.
    -   transparent: all actions since the founding of the WWoT are recorded and can be independently audited.


### Architecture

    In order to meet those properties, the Ledger of Trust is implemented as an append-only public ledger recording all actions since the founding of the WWoT, which each action signed using public key cryptography.

    In details:

    -   The founding is recorded as the Root of Trust registering their public key.

    -   Any participant can add another participant by registering their public key into the LoT; furthermore, a given participant can switch their own public key.

    -   Each individual subsequent action is then submitted with:
        -   the timestamp of the parent record,
        -   a hash of the parent's record properties,
        -   a signature of its properties by the actor.

    -   Multiple actions sharing the same parent are aggregated in a single record:
        -   the timestamp of its parent,
        -   a hash of the parent's record properties,
        -   a timestamp (UTC, Unix),
        -   a list of actions, each individually signed by their actor,
        -   a signature of the record properties by the current Root of Trust.

    The time-window to merge multiple actions is left as an exercise to the reader. A minute seems reasonable to allow actors time to sign their action and submit it and limiting to 1440 the number of records added in day.

    Note:   A single participant can only submit one action per record. Whilst a new record is being computed, any other action submitted by a participant is automatically denied.

    Note:   The timestamp is expected to be strictly monotonic, which should not be an issue within the UTC timezone. In case clock skew would lead a server to generate an unsuitable timestamp, it should simply increment the timestamp of the latest record by 1 instead.


### Participant

    For all intents and purpose, a participant is identified by their public key. Nothing else needs be known about them, and yet any wishing to can easily prove their identity by signing something with the paired private key.

    For notification purposes, participants are invited to submit an e-mail address. If they wish not to reveal their e-mail address, they may submit, instead or in addition, a list of pairs of public keys and encrypted e-mail addresses (with authentication).

    Actions of a participant are notified to:
    -   their Parent.
    -   themselves.
    -   their Children.

    Actions are to be notified individually, at the exception of Attestations which are notified once a day.


### Dilution

    For simplicity's sake, the graph of participants is restricted to a tree, and the denominators of the fractions of weight delegated must be powers of 2.

    For each node in the tree, the denominator is selected as the smallest power of 2 greater or equal to twice the number of children of the node:

    -   1 child: use 1 or 2 as denominator.
    -   2 children: use 1, 2 or 4.
    -   3 or 4 children: use 1, 2, 4 or 8.
    -   ...

    The sum of all Fractions of Weight so specified cannot exceed 1 for any given node. In the general case, there is some excedent remaining, which is the Fraction of Weight wielded by the node itself.

    Furthermore, the Total Fraction of Weight, obtained by multiplying the Fractions of Weights from the root to the node, must a multiple of 1p-52. This factor is chosen to ensure a lossless representation of the weight of a node using a 64-bits IEEE 754 floating point.


### No rewrite

    Unless the cryptographic primitives used are broken, a snapshot of the last record of the Ledger of Trust is sufficient to prove the faithful integrity of the entire LoT.

    Thereby, while full mirrors can be useful for redundancy reasons, even individuals of modest means can simply periodically check that the snapshot they have still belong to the LoT and take a new snapshot.


##  Nuances of Trust: Categories

    Every individual has their own set of strengths and weaknesses. Categories can be used to qualify trust.


### Categories of the Ledger of Trust

    Upon creation, a set of categories (UTF-8 encoded) is specified.

    From an implementation point of view, each category is its own Weighted Web of Trust. It is just easier to handle a single LoT than one for each category.


### Qualified dilution

    When delegating part of their weight to a child participant, the parent may either:

    -   Forego any qualification, which is equivalent to specifying the full set of categories currently active.
    -   Enumerate, for each category, the specific weight they wish to delegate.

    Note:   the checks on weights is performed independently for each category.


### Adding/Removing categories.

    At any point in time, the maintainer may add or remove categories.

    The action must be vetted by a majority of its children (> 2/3) to take effect:

    -   A removal is immediate; all weights currently associated with the removed category are simply obsolete.
    -   An addition is delayed; specifically, unqualified dilutions are only valid for the set of categories existing at the time they are created.

    After adding a category, each parent must re-specify the dilution of weights for their children for the new category to take effect. If a given participant does not re-specify the weights of its children, then all participants in the sub-tree rooted in this participant have a weight of 0 in this category: they are not trusted at all.


##  Building Trust: Actions

    A number of actions has already been specified, informally. This section gathers together all actions which can be executed on the LoT, as well as their kinematics.

    The kinematics notably specify the safeguards to prevent a participant gone rogue from completely wrecking the LoT.

    By default:
    -   all participants' actions should be rate-limited by the implementation on a per-invidual basis; the recommendation is a handful of actions per day.
    -   all actions asking for a response should time-out after a reasonable period of time; the recommendation is a handful of days.
    -   weights should only be trusted after being stable for a few days (or having been reverted to a previously stable value).


### Common

    All actions share common properties, which are not repeated below:
    -   Parent Record ID, except for Found.
    -   Parent Record Hash, except for Found.
    -   ID of initiator.
    -   Optionally, ID of Parent, if taking an action On Behalf.
    -   Signature of Hash of Action, encrypted & authenticated with initiator's private key.

### Fraction-wise Acceptance

    A number of actions require the Acceptance of over 2/3 or 4/5 of the Children, fraction-wise.

    The sum of the Fraction of Weight of the Children who Accept is divided by the sum of the Fraction of Weight of all Children of the participant, and the ratio must be greater or equal to 2/3 or 4/5, as appropriate.


### Root of Trust's Actions

    The Root of Trust is unlike any other participants, in that it decides whether to accept or reject the actions of other participants.

    In order to separate responsibilities, the Root of Trust is therefore NOT allowed to take any action directly, except for the initial Founding. Therefore, actions that must be executed as the Root of Trust are executed by one of its children on behalf of the RoT; see the Child's Actions section.


####    Founding

    The first record of the chain of record contains a single action, the Founding. This action is the only action with not parent hash.

    Found:
    -   Public Key of Root of Trust.
    -   For each initial founder:
        -   Public Key of the founder, which constitutes an Invitation.
    -   For each category:
        -   The name of the category (UTF-8, 128 bytes, no double-quote or non-printable character).
    -   A list of accepted encryption algorithms
    -   For each limit (rate-limit, number of children, etc...):
        -   The name of the limit.
        -   Its value. Omitted limits use the default value.

    The Founding is a momentous occasion; it calls for a celebration!

    List of limits:
    -   Number of children, power-of-two, defaults to 8.
    -   Freeze Period after Emergency Key Rotation, defaults to 1 day.
    -   Freeze Period after Invitation, defaults to 1 day.
    -   Grace Period of E-mail notification, defaults to 1 week.
    -   Grace Period of Frozen participants, defaults to 1 month.
    -   Maximum Pending Actions, defaults to 1.
    -   Rate Limit of number of Actions, defaults to 4/day.
    -   Rate Limit of number of Attestations, defaults to 1/minute + 6/hour + 48/day.
    -   Timeout of Acceptance/Declination, defaults to 1 week.

    Note:   The Maximum Pending Action limits the number of actions asking for Approval/Denial which are still neither accepted nor denied, excluding Invitations.


####    Categories Update

    A Categories Update can be used to tweak the list of categories, enforceable immediately.

    Update Categories:
    -   For each category:
        -   The name of the category (UTF-8, 128 bytes, no double-quote or non-printable character).

    Note:   Some Delegatations may mention categories which are not within this list. Those categories are simply ignored. New Delegations must respect this list, however.


####    Encryption Update

    An Encryption Update can be used to tweak the list of encryption algorithms, enforceable immediately.

    Encryption Update:
    -   A list of accepted encryption algorithms.

    Note:   Some users may be using an encryption algorithm which is no longer referenced in this list. The only action available to such users is a Key Rotation (or Emergency Key Rotation).


####    Limits Update

    A Limit Update can be used to tweak the value of the limits, enforceable immediately.

    Update Limits:
    -   For each limit (rate-limit, number of children, etc...):
        -   The name of the limit.
        -   Its value. Omitted limits use the default value.

    Note:   There may be a temporary validation of such limits when reducing them. For example, if reducing the number of children from 8 to 4, some participants may have 5 to 8 children already. This is accepted; they simply will not be able to take further actions which would violate the limits.


####    Ban

    A RoT can Ban a currently Frozen participant, preventing any further action on their part.

    Ban:
    -   ID of Frozen participant.

    Note:   participants remaining in the Frozen state for a reasonable delay (1 month) are considered ban without any explicit action.


####    Ban Lift

    A RoT can Lift the ban on a participant, restoring the participant to the Frozen state.

    Lift:
    -   ID of Banned participant.


### Parent's Actions

    A number of actions must be approved by a Grand-Parent. In the case of Children of the Root of Trust, the Root of Trust is unable to Accept anything.

    In this case, the action instead must be Accepted by over 2/3 of the Siblings of the Parent.


####    Invitation

    A participant may Invite an external individual to participate in the WWoT.

    Invite:
    -   the public key of the invited participant.

    No ID is associated to the participant until they Accept the Invitation.

    Note:   It is recommended that new participants not be allowed to issue Invitations for a reasonable period of time until after they Accepted their Invitation; such as a week.


####    Delegation

    A parent may choose to Delegate part of their weight to one or several of their children. For simplicity's sake, the fractions of all children, in each category, must be fully specified each time.

    Delegate:
    -   For each children, identified by ID:
        -   Fraction of Weight.
        OR
        -   For each category within a subset of available categories, Fraction of Weight.

    The Fraction of Weight may not be 0 for all categories; otherwise, it defeats the point of taking on a Child.


####    Rescindment

    A parent may choose to Rescind their invitation(s); this implicitly reduces the Fraction of Weight they delegate to this child to 0.

    The participant whose invitation is rescinded remains in the WWoT in a Frozen state, and can initiate no action other than Accepting the Invitation of another participant.

    Additional, all descendants of the participant are also, transitively, Frozen, until either the participant is Invited or they are, individually.

    Rescind:
    -   ID of the child participant.

    The Rescindment is not effective until the Grand-Parent Accepts it.


####    Transfer

    A participant may offer an existing participant to Transfer to their "family".

    Transfer:
    -   the ID of the transferred participant.

    The Transfer is not effective until the target participant Accept it.


####    Step Down

    A parent may choose to Step Down in favor of one their Children, in which case they swap place with said Children, all Fractions of Weight being conserved.

    Step Down:
    -   the ID of the child to take over.

    The Step Down is not effective until both the Grand-Parent and the target Child Accept it.


####    Acceptance/Declination

    A Parent may choose to Accept or Decline any:
    -   Move of e-mail address initiated by one of their Children.
    -   Rescindment initiated by one of their Children.
    -   Revocation initiated by one of their Children.
    -   Rotate or Emergency Rotate initiated by one of their Children.
    -   Step Down initiated by one of their Children.

    Accept/Decline:
    -   the hash of the action being accepted or declined.

    Accepts should be rare, so users are expected to do their due diligence and verify via another channel that the action is legitimate; the hash of the action is available to both verifiers and initiator to coordinate.

    Should the action timeout, the Parent is considered to have Declined.


### Child's Actions

    A number of actions must be approved by a Parent. In the case of Children of the Root of Trust, the Root of Trust is unable to Accept anything.

    In this case, the action instead must be Accepted by over 2/3 of the Siblings of the Child.

####    On Behalf

    In extraordinary circumstances, children may act on behalf of their parents.

    This may occur either if:
    -   the parent is the Root of Trust.
    -   the parent is incapacitated, or unwilling to act at all.

    In this case, the Child propose to execute an action On Behalf of their parent.

    The action is ineffective until it has been approved by:
    -   their Grand-Parent, and over 2/3 of their Siblings (fraction-wise).
    -   OR, over 4/5 of their Siblings.


####    Acceptance/Declination.

    A (potential) child may choose to Accept or Decline any:
    -   Invitation to enter the WWoT.
    -   Transfer to another parent.
    -   Step Down of their current parent in their favor.

    Accept/Decline:
    -   the hash of the action being accepted or declined.
    -   for Invite, a non-encrypted e-mail address.
    -   for Invite, for each public key:
        -   public key.
        -   encrypted & authenticated e-mail address.

    Accepts should be rare, so users are expected to do their due diligence and verify via another channel that the action is legitimate; the hash of the action is available to both verifiers and initiator to coordinate.

    Should the action timeout, the child is considered to have Declined.


### Participant's Actions

    A number of actions must be approved by a Parent. In the case of Children of the Root of Trust, the Root of Trust is unable to Accept anything.

    In this case, the action instead must be Accepted by over 3/4 of the Children of the RoT.

    The actions in this section cannot be executed On Behalf; though for Emergency Key Rotation it is unnecessary.


####    Attestation

    The most common action of any participant should actually be to use their private key to sign "things" *outside* of the Ledger of Trust.

    Without any Attestation present in the ledger, however, such signatures should be considered null and void.

    Attest:
    -   the hash which was signed.
    -   the signature of the hash.
    -   a plain text description of what was signed (UTF-8, 1024 bytes, no double-quote or non-printable character).


####    Revocation

    A participant may realise that an attestation was erroneous, or illegitimate. In this case, they may revoke it.

    Revoke:
    -   the hash of the attestation.
    -   a plain text description of the reason (UTF-8, 1024 bytes, no double-quote or non-printable character).

    The Revocation is not effective until the Parent Accepts it.


####    Key Rotation

    At any point in time, a participant may decide to Rotate their public key.

    Rotate:
    -   the new public key.
    -   the old public key.
    -   signature with old key.

    The Rotate is not effective until the Parent Accepts it.


####    Emergency Key Rotation

    In the extraordinary case in which a participant loses access to their private key, they can Emergency Rotate their public key.

    Emergency Rotate:
    -   the new public key.
    -   the old public key.

    The Emergency Rotate is not effective until both the Parent, 2/3 of the Siblings (if any, fraction-wise) and over 2/3 of the Children (if any, fraction-wise) Accept it.

    Furthermore, it is recommended that a Freeze period of at least one full day be applied during which any action except another Emergency Rotate submitted by the participant will be refused by the LoT.

    Note:   the Emergency Rotate must be signed only by the new private key, obviously; also the new participant must first become a member of the WWoT.

    Note:   the Emergency Rotate can only be used by a new participant, as their first action after Accepting the Invitation.


####    E-mail Move

    At any point in time, a participant may decide to Move to a new e-mail address.

    Move:
    -   optional, the new e-mail address, unencrypted.
    -   for each public key:
        -   public key.
        -   encrypted & authenticated e-mail address.

    The Move is not effective until the Parent Accepts it.

    Note:   It is recommended that notifications be send to the old e-mail address for another week.

    Note:   Mirrors should send notifications of actions to the e-mail address only if their public key is used.


##  Speeding up Trust: Indexes

    The chain of records of a Ledger of Trust is useful for audit, however it does not allow quickly accessing the current weight, for a selected category, of any given participant.

    In order to speed up accesses, it is recommended that the LoT provides a set of index files, as specified below.

    This is, however, optional, and mirrors for example may choose to eschew it to minimize storage cost.


### Format

    The indexes should be easily accessible from any language, and the history of their modification should also be easily accessible.

    As such, it is recommended that the indexes:
    -   be committed in a Version Control System, alongside the chain of records.
    -   be structured with one directory per index,
    -   within which CSV files of no more than 4,096 records be stored, with the details.

    Even though CSV can be a surprisingly intricate format, care will be taken here to avoid its intricacies: strings will be quoted and contain neither quotes nor special characters.

    Furthermore, all numbers, hashes, etc... are hexadecimal encoded and quoted. Always.


### Key Index

    The Key Index maps the public key(s) of an invidual to its ID in the "keys" directory.


    Each CSV file within the directory is named after the range of keys it contains. To do so, the keys are encoded in hexadecimal, and a prefix is used to specify an inclusive range.

    Example:    "00aa-00bb.csv" contains all keys whose prefix of 4 letters falls into ["00aa", "00bb"].

    Note:       Both prefixes should have the same size.


    The CSV file itself contains a CSV header, and one record per key: "Key", "Algorithm", "ID", "Valid Until".

    Valid Until is the Unix Timestamp of the record in which the Key Rotation which invalidated the key is contained.


### User Index

    The User Index maps the ID of an individual to its Key in the "users" directory.


    Each CSV file within the directory is named after the range of IDs it contains, encoded in hexadecimal, in fixed chunks of 4096 users.

    Since IDs are sequential, starting from 0 for the Root of Trust, the first file will be "0-1000.csv", even if it only contains a handful of users.


    The CSV file itself contains a CSV header, and one record per ID: "ID", "Key", "Key-1", "Valid Until-1", "Key-2", "Valid Until-2", ...

    Valid Untils are Unix Timestamp. The number of Keys presented is dynamic, and expanded as needed to fit users. As a result, it is expected that many records will contain no value in those columns.


### Weight Index

    The Weight Index maps each user ID to their current weight within each category of the Ledger of Trust in the "weights" directory.


    Each CSV file within the directory is named after the range of IDs it contains, encoded in hexadecimal, in fixed chunks of 4096 users.

    Since IDs are sequential, starting from 0 for the Root of Trust, the first file will be "0-1000.csv", even if it only contains a handful of users.


    The CSV file itself contains a CSV header, and one record per ID: "ID", "<name of category 0>", "<name of category 1>", ...

    Weights use floating point hexadecimal encoding, as given by "%a" (printf): mantissa, "p", exponent.

    Example:

    ```
    2af,"1p-2"
    ```

    Note:   "1p-2" corresponds to `1 * 2**-2`, that is `1 / 2**2` or `1/4`.

    Reminder:   The smallest exponent which can be achieved is -52.


### Attestation Index

    The Attestation Index maps each attestation submitted to the Ledger of Trust to its submitter in the "attestations" directory.


    It is expected that the number of attestations be massive, therefore in order not to overload the filesystem, sub-directories may be used.

    Each CSV file within a directory is named after the range of attestation hashes it contains, encoded in hexadecimal, using the same prefix-range system as for the Key Index.

    The number of files in a directory is limited to a maximum of 256, if it should be exceeded, a new layer of sub-directories should be created. The sub-directories are named using 4 hexits of the attestation hash:
    -   the first layer of sub-directories uses the first 4 hexits,
    -   the second layer of sub-directories uses the next 4 hexits,
    -   ...

    Note:   The CSV file name should contain the full prefix, repeating the hexits mentioned in its parent directories if necessary.


    The CSV file itself contains a CSV header, and one record per attestation: "Attestation Hash", "Signature", "Timestamp", "ID".

    Timestamp is the Unix Timestamp of the record in which the Attestation is contained.


##  Attack Vectors

    In order to be useful, a Ledger of Trust must be resilient to attacks from unscrupulous individuals.

    This chapter attempts to list all potential attack vectors, and whether they are defeated or mitigated and which counter-measures can be employed against them.


### Denial of Service

    External actors may simply cause a denial of service. This may achieved in many ways, such as preventing the server from being contacted, or crushing the server under load.

    Consequence:
    -   No new action is accepted in the LoT.
    -   The LoT cannot consulted via this service.

    Detection:
    -   Mirrors: cannot access, notify Children of the RoT via e-mail.
    -   Users: cannot access.

    Mitigation:
    -   The LoT can be consulted on Mirrors or File Hosting services.


### Rogue Host

    The host or service which maintains the LoT can be subverted, which may give access to the private key of the Root of Trust.

    The implications of getting access to the Root of Trust are considered in the next section; within this section, only the subversion of the host itself is considered.

    Note that the only sensitive information pertaining to the Ledger of Trust on the server would be:
    -   the private key of the Root of Trust,
    -   optionally, the private key(s) necessary to sync the Ledger of Trust unto File Hosting systems.


####    Denial of Service

    The service can be shut-down, preventing it from processing actions or answering inquiries.

    Consequence:
    -   No new action is accepted in the LoT.
    -   The LoT cannot consulted via this service.

    Detection:
    -   Mirrors: cannot access.
    -   Users: cannot access.

    Mitigation:
    -   The LoT can be consulted on Mirrors or File Hosting services.


####    Targetted Denial of Service.

    The service may be modified to selectively shut-out a portion of users.

    Consequence:
    -   Those users cannot submit new actions.
    -   Those users cannot consult the LoT.

    Detection:
    -   Mirrors: if affected users use dual-submits.
    -   Users: if affected.

    Mitigation:
    -   The LoT can be consulted on Mirrors or File Hosting services.


### Rogue Root of Trust

    The Root of Trust is the single-most important participant as it is in charge of maintaining the chain of records of the Ledger of Trust.

    At the same time, though, it is also the least participant, as it can take no action.


####    Actions Drop

    The service may be modified to ack actions without actually including them in the LoT.

    Consequence:
    -   Those actions dropped are not included in the LoT.

    Detection:
    -   Mirrors: if affected users use dual-submits.
    -   Users: if checking the LoT.


####    Selective View

    The service may be modified to present a different view of the LoT to different clients.

    Consequence:
    -   Affected users see a different "truth".

    Detection:
    -   Integrity checks: mismatching parent's record hash of an action.
    -   Comparison checks: the service and mirrors/file hosting services present a different view of the LoT.


####    Forged Action

    The service may attempt to include Actions on behalf of participants.

    Consequence:
    -   Corruption of the LoT.

    Detection:
    -   Integrity checks: the signature of the Action does not match their public key, the server should thus have refused them.


### Compromised Participant

    In case of a key being compromised, the participant should first worry about the means by which the key was obtained, and what else may have been accessed affected.

    Consequence:
    -   The rogue actor may use the key to issue actions on the LoT on behalf of the legitimate user.

    Detection:
    -   Each participant receives an Attestation bundle each day, containing:
        -   all the Attestations they did,
        -   all the Attestations their parent did,
        -   all the Attestations their children did.
    -   Other actions are immediately notified to parent, participant and their children.

    Mitigation:
    -   The Key Rotation, or Emergency Key Rotation, must be used to prevent further access.
    -   Illegitimate actions carried out must be reversed.

    Only specific actions used to take over an account more definitely are listed here; other regular actions that a participant may do are listed in "Rogue Participant".


####    Lock Out (via Key Rotation or Emergency Key Rotation)

    If the account is compromised, an adversary may attempt to Rotate its key.

    Consequence:
    -   The legitimate user is locked out.

    Defeat:
    -   Parent can deny the request.


####    Tune Out (via Move)

    If the account is compromised, an adversary can attempt to tune out the owner by using Move.

    Consequence:
    -   The owner will not receive further notification, after the grace period.

    Defeat:
    -   Parent can deny the request.

    Mitigation:
    -   Old e-mail address receives notifications of Actions for a grace period.
    -   Another Move can correct the situation.


### Rogue Participant

    A participant can go rogue. Whether because of an emotional surge, or a drinks/drugs induced euphory, a participant can suddenly act in unanticipated ways.

    All actions that a participant may undertake in such state can be detected and reverted, though it is obviously best if this proves unnecessary.

    A few select actions will be reviewed first, as they merit special consideration, while others will be reviewed in group.


####    Rogue Attestations

    This is, ultimately, the greatest damage that a participant may carry out. While the Ledger of Trust itself can be repaired, its reputation may tumble.

    Consequence:
    -   Disingenuous votes/scores/reviews/... were issued.

    Mitigation:
    -   Unless the participant carries significant weight, their vote/score/review is hopefully drowned in the noise.
    -   They can be revoked, though their reputation will suffer.


####    Flood of Attestations

    A rogue participant could also attempt to simply drown out the Ledger of Trust by sheer volume of requests.

    Generating random hashes is easy enough, and the requests would be properly signed.

    Consequence:
    -   Bloated chain of records.

    Mitigation:
    -   Rate limiting. Though attestations constitute the bulk of actions, so their rate-limit is much higher than any other actions.
    -   Later Revocation, to purge the indexes.


####    Flood of Invitations

    One way of bypassing rate-limiting for one's account, is to use multiple accounts!

    Consequence:
    -   Some bloat in the chain of records.

    Defeat:
    -   New accounts cannot, themselves, issue Invitations for a reasonable period.

    Mitigation:
    -   Strict rate-limiting; only a handful of Invitations can be issued each day.
    -   New accounts can only use other rate-limited actions.

    Still, a Flood of Invitations followed by a Flood of Attestations mean that a single participant can, under the default limits, submit 240 Attestations within a day.


####    Flood of Accept/Decline.

    Consequence:
    -   Denial of Service, by overload.

    Defeat:
    -   Unknown hashes are immediately rejected.
    -   Existing hashes referring to others' actions are immediately rejected.
    -   Existing hashes referring to one's already approved/denied actions are immediately rejected.


####    Shut-down (via Delegation)

    A form of rage-quit: reducing the fraction of all children to their minimum.

    Consequence:
    -   Descendants' weight is reduced to 0 in many categories, transitively.

    Mitigation:
    -   Only a subset of votes/scores/reviews/... is affected.
    -   Temporary, until reverted or Transfer occur.


####    Other Actions

    Other actions include: Ban, Lift, Update Categories, Update Limits, Transfer, Rescindment, Step Down, Revocation, Key Rotation and Move.

    Consequence:
    -   Potential disturbance.

    Defeat:
    -   Only a single such pending action can be proposed at a time.
    -   Denied by default, and rare enough they should not be Accepted without thoughts.

    Mitigation:
    -   Requests accepted by mistake can be reverted.


##  Costs

    A small server, or even consumer hardware, should be sufficient to run a small instance of a Ledger of Trust. Or to avoid most headaches with hardware, a small instance in the cloud.


### Processing Power

    A Ledger of Trust server will require processing power proportional to the number of actions carried out by its participants, since it is mostly reactive.

    The only time-based action it carries out is sending a daily bundle of Attestations issued that day, which is itself proportional to the number of such issued Attestations and the affected number of participants.

    Note:   It is possible for the main server to focus on accepting new actions, while a mirror is tasked with maintaining the indexes on a File Hosting system, in case maintaining the indexes proves to be a bottleneck.


### Memory

    The server should only need to maintain in memory:

    -   the last record in the chain, ready to process the next batch of actions.
    -   an index of all participants, with their currently used public key, and their relationships.
    -   an index of all participants and their rate limits.
    -   an index of the on-disk files, if it maintains one.

    Specifically, the server need not remember past actions, and certainly not the mass of attestations. Instead, queries can be deflected to File Hosting systems such as GitHub or GitLab.


### Storage

    Using an external File Hosting system, the server itself could run storage-less.

    The one exception, unless one trusts other mirrors to detect inconsistencies, would be to maintain the last record in the chain locally.

    To speed up reboots, a local copy of all the indexes would still be preferrable, though.


##  Conclusion

    The Weighted Web of Trust concept allows Trust to scale, and its Ledger of Trust implementation is both simple and secure enough to be practical.
