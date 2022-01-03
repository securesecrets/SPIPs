# SPIP-GOV-0 Spec: L2 Governance with adjustable privacy
Governance module's SPIP version 0 is a specification for layer 2 governance modules that 
provide adjustable privacy. This specification's voting interface depends on a SNIP-20 that 
implements [SPIP-STKN-0](./SPIP-STKN-0.md). The spec also provides ways to combat lack of participation and 
more protocol control without risking decentralization.

* links

# Introduction
## Scope
This specification's goals are to provide a generic interface for future governance based modules.
The [interface](#ContractInterface) is divided by different optional sections that can be implemented
according to your projects requirements; [Base](#Base) being the minimum that should get implemented.

# Contract Interface

## Base
The base section defines the minimum to define to follow functionality.
### Messages
#### CreateProposal
Creates a proposal that follows the base profile. If no ```msg``` is set then it can be considered a text only proposal
for off-chain actions.

| Name     | Type          | Description                                                           | Optional |
|----------|---------------|-----------------------------------------------------------------------|----------|
| msg      | Base64 string | The cosmos sdk message that will trigger if proposal is accepted      | Yes      |
| metadata | Base64 string | Contains a description of the proposal, can use a json format for UIs | No       |
| padding  | string        | Ignored string used to maintain constant-length messages              | Yes      |

#### Receive
Interface for receiving allowed SNIP20 tokens used to fund proposals that are in funding periods. Sender addresses and 
amounts must be stored to allow sending them back when a proposal's final vote status is ```approved```, ```denied``` or
```canceled```.

| Name    | Type          | Description                                              | Optional |
|---------|---------------|----------------------------------------------------------|----------|
| from    | String        | Where the funds will come back to if the proposal fails  | No       |
| amount  | Uint128       | The funding amount                                       | No       |
| msg     | Uint128       | The proposal's ID                                        | No       |
| padding | string        | Ignored string used to maintain constant-length messages | Yes      |

#### ReceiveBalance
Must be received from an address that is recognized as the voting token. It's used to vote on proposals.

| Name    | Type    | Description                     | Optional |
|---------|---------|---------------------------------|----------|
| sender  | string  | Voter address                   | No
| msg     | string  | Base64 message of [Vote](#Vote) | No
| balance | Uint128 | Voting power                    | No

#### TriggerProposal
Triggers a proposal when it contains a set ```msg``` and has already passed through all of its phases.

| Name | Type    | Description | Optional |
|------|---------|-------------|----------|
| id   | string  | Proposal ID | No

### Queries

#### Proposals
Queries multiple proposals. Permits can be used to also reveal user vote.
Depending on privacy settings, proposals could require permits.

| Name            | Type                                      | Description                                  | Optional | Removable |
|-----------------|--------------------------------------------|---------------------------------------------|----------|-----------|
| start           | Uint128                                    | Where to start the range, will default to 0 | Yes      | No
| total           | Uint128                                    | Maximum number to return                    | No       | No
| status_filters  | Array of [ProposalStatus](#ProposalStatus) | Filters proposals by status                 | Yes      | No
| representatives | string array                               | Representative votes to show                | Yes      | Yes |
| show_vote       | bool                                       | show permit signer vote                     | No       | No
| permit          | [Permit](#Permit)                          | Query permit                                | Yes      | Yes

##### Result

```json
[
  {
    "id": "ID",
    "proposal": "Proposal struct",
    "status": "Proposal status",
    "committee-votes": "OPTIONAL: Vote to show distribution",
    "committee-deadline": "OPTIONAL u64",
    "funding": "OPTIONAL: current/final funding",
    "funding-deadline": "OPTIONAL u64",
    "votes": "OPTIONAL: Vote to show distribution",
    "voting-deadline": "OPTIONAL u64",
    "representative-votes": [
      {
        "representative": "address",
        "vote": "Vote to show distribution"
      }
    ],
    "vote": "OPTIONAL: Vote"
  }
]
```

### Structures
#### Proposal
Proposal structure, must be stored with ID as its key. Its privacy features will be given by its associated committee.

| Name      | Type          | Description                    | Optional | Removable |
|-----------|---------------|--------------------------------|----------|-----------|
| committee | string        | The committee that proposed it | No       | Yes
| proposer  | string        | The proposer's address         | No       | No
| msg       | Base64 string | Message to trigger if approved | Yes      | No
| metadata  | Base64 string | Proposal description           | No       | No

#### ProposalStatus
Shows the proposal's current phase
* ```Funding``` - Still receiving funds
* ```CommitteeVote``` - Committee is voting
* ```Voting``` - Open to all voters
* ```Expired``` - Did not reach deadline expectations
* ```Rejected``` - Majority voted ```No```
* ```Vetoed``` - Majority voted ```NoWithVeto```
* ```Passed```- Majority voted ```Yes```

#### Permit
Permits follow Shade Protocol's permit structures
TODO: add info regarding allowed committee queries and other settings

| Name               | Type          | Description                                | Optional | Removable |
|--------------------|---------------|--------------------------------------------|----------|-----------|
| committees         | string array  | Limits which committee information to view | Yes      | Yes
| view_proposal_data | bool          | Allows to view proposal interactions       | Yes      | No
| proposals          | string array  | Limits which proposals can be viewed       | Yes      | No

#### Vote
The sum of all votes must be less or equal to 100

| Name         | Type   | Optional 
|--------------|--------|----------
| yes          | string | Yes
| no           | string | Yes
| no_with_veto | string | Yes
| abstain      | string | Yes


## Committees
Allows other elected parties to have proposal parameters different from the general user base. These proposals can be
restricted to a closed set of messages called CommitteeMsg, this prevents specific committees from creating off-topic
proposals, and limit their overall network power.

Default committees to keep in mind are:
* ```Public``` - This encompasses all stakers
* ```Admin``` - Initial contract admins

### Messages
#### AddCommittee
Creates a new committee

| Name      | Type          | Description                    | Optional | Removable |
|-----------|---------------|--------------------------------|----------|-----------|
| name      | Base64 string | Readable name                  | No       | No
| metadata  | Base64 string | Description                    | No       | No
| members   | string array  | Addresses of members involved  | No       | No
| profile   | string        | ID of [Profile](#Profile)      | No       | Yes

#### UpdateCommittee
Updates an existing committee

| Name      | Type          | Description                    | Optional | Removable |
|-----------|---------------|--------------------------------|----------|-----------|
| id        | string        | Committee ID                   | No       | No
| name      | Base64 string | Readable name                  | Yes      | No
| metadata  | Base64 string | Description                    | Yes      | No
| members   | string array  | Addresses of members involved  | Tes      | No
| profile   | string        | ID of [Profile](#Profile)      | Yes      | Yes

#### RemoveCommittee
Removes a committee

| Name      | Type          | Description                    | Optional | Removable |
|-----------|---------------|--------------------------------|----------|-----------|
| id        | string        | Committee ID                   | No       | No

#### ProposeCommitteeMsg
Proposal created by a committee member. The proposal must conform to an existing [CommitteeMsg](#CommitteeMsg).

| Name         | Type          | Description                          | Optional |
|--------------|---------------|--------------------------------------|----------|
| committee_id | string        | The committee rules for the proposal | No       |
| msg_id       | string        | CommitteeMsg to trigger              | Yes      |
| variables    | string array  | Variables for the CommitteeMsg       | Yes      |
| metadata     | base64 string | Proposal description                 | No       |

#### AddCommitteeMsg
Creates a new [CommitteeMsg](#CommitteeMsg) template. The msg template must contain the contract's msg variable standard;
in Shade Protocol, that standard is "{}"

| Name         | Type          | Description                           | Optional |
|--------------|---------------|---------------------------------------|----------|
| committees   | string array  | Committees that can use this template | No       |
| msg_template | base64 string | Message template                      | No       |

#### UpdateCommitteeMsg
Updates a [CommitteeMsg](#CommitteeMsg)

| Name       | Type          | Description                           | Optional |
|------------|---------------|---------------------------------------|----------|
| id         | string        | Msg id                                | No |
| committees | string array  | Committees that can use this template | No       |

#### RemoveCommitteeMsg
Removes a [CommitteeMsg](#CommitteeMsg)

| Name       | Type          | Description                           | Optional |
|------------|---------------|---------------------------------------|----------|
| id         | string        | Msg id                                | No |

### Queries

#### Committees
Queries multiple committee information, information is bound by their privacy profile.

| Name            | Type              | Description   | Optional | Removable |
|-----------------|-------------------|---------------|----------|-----------|
| start           | string            | ID start      | Yes      | No |
| total           | string            | Total results | Yes      | No |
| permit          | [Permit](#Permit) | Query permit  | Yes      | Yes |

```json
[
  {
    "id": "committee ID",
    "committee": "committee struct"
  }
]
```

#### CommitteeMsgs
Queries multiple CommitteeMsg

| Name            | Type              | Description     | Optional |
|-----------------|-------------------|-----------------|----------|
| id              | string            | CommitteeMsg ID | No       |
| start           | string            | ID start      | Yes      | No |
| total           | string            | Total results | Yes      | No |

##### Result

```json
[
  {
    "id": "committeeMsg ID",
    "CommitteeMsg": "committeeMsg struct"
  }
]
```

### Structures
#### Committee

| Name            | Type              | Description   | Optional | Removable |
|-----------------|-------------------|---------------|----------|-----------|
| name      | Base64 string | Readable name                  | No       | No
| metadata  | Base64 string | Description                    | No       | No
| members   | string array  | Addresses of members involved  | No       | No
| profile   | string        | ID of [Profile](#Profile)      | No       | Yes

#### CommitteeMsg
A proposal Msg that is bound by a set amount of variable fields.

| Name            | Type              | Description   | Optional |
|-----------------|-------------------|---------------|----------|
| committees   | string array  | Committees that can use this template | No       |
| msg_template | base64 string | Message template                      | No       |

## Profiles
Allow better control over the safety and privacy features that proposals will need if 
[Committees](#Committees) are implemented.
### Messages
#### AddProfile
Creates a new profile

| Name                          | Type   | Description                                                  | Optional |
|-------------------------------|--------|--------------------------------------------------------------|----------|
| proposal_privacy              | bool   | Limit if only committee members can view proposal info       | No
| committee_vote_deadline       | u64    | Time limit for committee voting                              | Yes
| committee_vote_quorum         | string | Required percentage of committee member participation        | Yes
| committee_vote_threshold      | string | Required committee proportion of ```Yes``` votes             | Yes
| committee_vote_veto_threshold | string | Required ```NoWithVeto``` votes in proportion to total votes | Yes
| committee_member_privacy      | bool   | Privacy of committee members                                 | Yes
| funding_deadline              | u64    | Time limit for funding                                       | Yes
| funding_required              | string | Funding amount required                                      | Yes
| funder_privacy                | bool   | Do not reveal funder addresses                               | Yes
| vote_deadline                 | u64    | Time limit for voting                                        | Yes
| vote_quorum                   | string | Required percentage of staker participation                  | Yes
| vote_threshold                | string | Required proportion of ```Yes``` votes                       | Yes
| vote_veto_threshold           | string | Required ```NoWithVeto``` votes in proportion to total votes | Yes
| failed_deposit_loss           | string | Percentage of deposit to be lost when proposal fails         | Yes
| veto_deposit_loss             | string | Percentage of deposit to be lost when proposal is vetoed     | Yes

#### UpdateProfile
Updates an existing profile

| Name                          | Type   | Description                                                  | Optional |
|-------------------------------|--------|--------------------------------------------------------------|----------|
| id                            | string | Profile ID                                                   | No
| proposal_privacy              | bool   | Limit if only committee members can view proposal info       | Yes
| committee_vote_deadline       | u64    | Time limit for committee voting                              | Yes
| committee_vote_quorum         | string | Required percentage of committee member participation        | Yes
| committee_vote_threshold      | string | Required committee proportion of ```Yes``` votes             | Yes
| committee_vote_veto_threshold | string | Required ```NoWithVeto``` votes in proportion to total votes | Yes
| committee_member_privacy      | bool   | Privacy of committee members                                 | Yes
| funding_deadline              | u64    | Time limit for funding                                       | Yes
| funding_required              | string | Funding amount required                                      | Yes
| funder_privacy                | bool   | Do not reveal funder addresses                               | Yes
| vote_deadline                 | u64    | Time limit for voting                                        | Yes
| vote_quorum                   | string | Required percentage of staker participation                  | Yes
| vote_threshold                | string | Required proportion of ```Yes``` votes                       | Yes
| vote_veto_threshold           | string | Required ```NoWithVeto``` votes in proportion to total votes | Yes
| failed_deposit_loss           | string | Percentage of deposit to be lost when proposal fails         | Yes
| veto_deposit_loss             | string | Percentage of deposit to be lost when proposal is vetoed     | Yes

#### RemoveProfile
Removes an existing profile

| Name            | Type              | Description   | Optional |
|-----------------|-------------------|---------------|----------|
| id              | string            | Profile ID    | No       

### Queries
#### Profiles
Queries multiple profiles

| Name            | Type              | Description     | Optional |
|-----------------|-------------------|-----------------|----------|
| id              | string            | Profile ID | No       |
| start           | string            | ID start      | Yes      | No |
| total           | string            | Total results | Yes      | No 

```json
[
  {
    "id": "profile ID",
    "profile": "profile struct"
  }
]
```

### Structures
#### Profile
The developer implementing this spec should feel free to add/remove settings.

| Name                          | Type   | Description                                                  | Optional |
|-------------------------------|--------|--------------------------------------------------------------|----------|
| proposal_privacy              | bool   | Limit if only committee members can view proposal info       | No
| committee_vote_deadline       | u64    | Time limit for committee voting                              | Yes
| committee_vote_quorum         | string | Required percentage of committee member participation        | Yes
| committee_vote_threshold      | string | Required committee proportion of ```Yes``` votes             | Yes
| committee_vote_veto_threshold | string | Required ```NoWithVeto``` votes in proportion to total votes | Yes
| committee_member_privacy      | bool   | Privacy of committee members                                 | Yes
| funding_deadline              | u64    | Time limit for funding                                       | Yes
| funding_required              | string | Funding amount required                                      | Yes
| funder_privacy                | bool   | Do not reveal funder addresses                               | Yes
| vote_deadline                 | u64    | Time limit for voting                                        | Yes
| vote_quorum                   | string | Required percentage of staker participation                  | Yes
| vote_threshold                | string | Required proportion of ```Yes``` votes                       | Yes
| vote_veto_threshold           | string | Required ```NoWithVeto``` votes in proportion to total votes | Yes
| failed_deposit_loss           | string | Percentage of deposit to be lost when proposal fails         | Yes
| veto_deposit_loss             | string | Percentage of deposit to be lost when proposal is vetoed     | Yes

## Representatives
Combat lack of voter participation by allowing them a way to entrust their vote. If a user that has allowed a 
representative to vote on his behalf votes, then his voting power is discounted from the representative.
### Messages
#### AddRepresentative
Creates a representative account

| Name     | Type          | Description         | Optional |
|----------|---------------|---------------------|----------|
| name     | string        | Representative name | No
| metadata | base64 string | description         | No

#### UpdateRepresentative
Updates a representative account

| Name     | Type          | Description         | Optional |
|----------|---------------|---------------------|----------|
| id       | string        | Representative ID   | No
| name     | string        | Representative name | Yes
| metadata | base64 string | description         | Yes

#### RemoveRepresentative
Removes a representative account

| Name     | Type          | Description         | Optional |
|----------|---------------|---------------------|----------|
| id       | string        | Representative ID   | No

#### VoteAsRepresentative
Lets a representative vote on behalf of entrusted voter

| Name               | Type         | Description         | Optional |
|-------------------|---------------|---------------------|----------|
| representative_id | string        | Representative ID   | No
| proposal_id       | string        | Proposal ID         | No 
| vote              | [Vote](#Vote) | Representative name | No

#### ReceiveBalance
Entrusts voting power on one representative, representative vote can be overridden if entrusted voter votes.
Must be received from an address that is recognized as the voting token.

| Name    | Type    | Description                     | Optional |
|---------|---------|---------------------------------|----------|
| sender  | string  | Voter address                   | No
| msg     | string  | Representative ID               | No
| balance | Uint128 | Voting power                    | No

### Queries
#### Representatives
Queries multiple representatives

| Name            | Type              | Description     | Optional |
|-----------------|-------------------|-----------------|----------|
| id              | string            | Representative ID | No       |
| start           | string            | ID start      | Yes      | No |
| total           | string            | Total results | Yes      | No 

```json
[
  {
    "id": "representative ID",
    "representative": "representative struct"
  }
]
```

### Structures
#### Representative

| Name     | Type          | Description         | Optional |
|----------|---------------|---------------------|----------|
| name     | string        | Representative name | No
| metadata | base64 string | description         | No
| address  | string        | address             | No