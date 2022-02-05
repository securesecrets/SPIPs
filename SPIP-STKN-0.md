# SPIP-STKN-0 Spec: Staked SNIP20
Staked Token (Staked SNIP20) version 0 is a specification for staked layer 2 tokens. They should not be considered 
compliant SNIP20s in terms of functionality as they lack the ability to transfer the tokens freely.

* [Additions](#Additions)
  * [Staking](#Staking)
    * Messages
      * [InitExtensions](#InitExtensions)
      * [Receive](#Receive)
      * [Unbond](#Unbond)
      * [ClaimUnbond](#ClaimUnbond)
      * [ClaimRewards](#ClaimRewards)
      * [StakeRewards](#StakeRewards)
    * Query
      * [TotalStaked](#TotalStaked)
      * [Staked](#Staked)
      * [Unbonding](#Unbonding)
    * Structure
      * [Stake](#Stake)
      * [Unbonding](#Unbonding)
  * [Balance](#Balance)
    * [SendBalance](#SendBalance)
    * [ReceiveBalance](#ReceiveBalance)
  * Distributors
    * Messages
      * [AddDistributors](#AddDistributors)
      * [SetDistributors](#SetDistributors)
    * Query
      * [Distributors](#Distributors)
      
## Additions
Additions that expand the SNIP20's functionality
### Staking
Converts a standard token into a stakedToken
#### Messages
##### InitExtensions
Notes the necessary modifications to the initializer for staking to properly work

| Name         | Type     | Description                     | Optional |
|--------------|----------|---------------------------------|----------|
| treasury     | string   | Where the staked funds will go  | No       |
| unbond_time  | u64      | Time it takes to unbond a token | No       |
| staked_token | Contract | Token that will be staked       | No       |

##### Receive
Used to receive the amount to stake. The ```msg``` field is used for context, the amount will either be used to stake 
or to increase token pool

| Name         | Type   | Description         | Optional |
|--------------|--------|---------------------|----------|
| sender       | string | Sender of the funds | No       |
| amount       | string | Amount to stake     | No       |
| msg          | string | Deposit context     | Yes      |

##### Unbond
Claim rewards and also unbonds the set amount, user will have to wait the time set by ```unbond_time``` 
before they can call [ClaimUnbond](#ClaimUnbond)

| Name         | Type   | Description         | Optional |
|--------------|--------|---------------------|----------|
| amount       | string | Amount to stake     | No       |

##### ClaimUnbond
Claims the unbonded tokens if available

##### ClaimRewards
Claims the staking rewards

##### StakeRewards
Claims the staking rewards and stakes them

#### Query
##### TotalStaked
Queries the total tokens staked
##### Result

```json
{
  "total_staked": {
    "tokens": "token amount",
    "shares": "share amount"
  }
}
```
##### Staked
Queries a user's stake state, along with unbonding information.

| Name    | Type   | Description     | Optional |
|---------|--------|-----------------|----------|
| address | string | Amount to stake | No       |
| key     | string | Viewing key     | No       |
| time    | u64    | Current time    | Yes      |

##### Result

```json
{
  "staked": "user staked",
  "shares": "user shares",
  "pending_rewards": "claimable rewards",
  "unbonding": "tokens still unbonding",
  "unbonded": "claimable tokens"
}
```
##### Unbonding
Checks tokens being unbonded in a rage

| Name  | Type | Description | Optional |
|-------|------|-------------|----------|
| start | u64  | Range start | No       |
| end   | u64  | Range end   | No       |

##### Result

```json
{
  "unbonding": {
    "total": "unbonding amount"
  }
}
```

#### Structures
##### Stake
Used to keep an internal staking state and to track a user's staked amount and shares to efficiently calculate rewards

| Name   | Type   | Description                          | Optional |
|--------|--------|--------------------------------------|----------|
| shares | string | Representation of total shares owned | No       |
| tokens | string | Total tokens owned by the contract   | No       |

##### Unbonding
A single unbond transaction

| Name    | Type   | Description                      | Optional |
|---------|--------|----------------------------------|----------|
| amount  | string | Amount to unbond                 | No       |
| release | u64    | When the funds will be claimable | No       |

### Balance
Used to send the account's total balance instead of sending some amount, when a smart contract requires viewing total
staked then this serves as a good alternative to sending a viewing key or permit.
#### Messages
##### SendBalance
Exposes the sender's current account balance.

| Name      | Type          | Description                                              | Optional |
|-----------|---------------|----------------------------------------------------------|----------|
| recipient | string        | Address of balance receiver                              | No       |
| code_hash | string        | Code hash of the contract                                | No       |
| msg       | Base64 string | An optional message to send along                        | Yes      |
| padding   | string        | Ignored string used to maintain constant-length messages | Yes      |

##### ReceiveBalance
Implemented by contracts that want to handle the [SendBalance](#SendBalance) function.

| Name    | Type   | Description                 | Optional |
|---------|--------|-----------------------------|----------|
| sender  | string | Address of balance receiver | No       |
| msg     | string | Optional message to receive | Yes      |
| balance | string | Balance of sender           | No       |

### Distributors Interface
Staked tokens should only be sent by allowed distributors, this prevents users from using a staked token like if it was
a normal token. Additionally, should only be allowed to send to distributors. Functions that require a distributor 
check are ```Send```, ```Transfer```, ```SendFrom```, ```TransferFrom```, ```BatchSend```, ```BatchTransfer```, 
```BatchSendFrom``` and ```BatchTransferFrom```.
#### Messages
##### AddDistributors
Adds distributors to the list of allowed distributors.

| Name         | Type             | Description                                              | Optional |
|--------------|------------------|----------------------------------------------------------|----------|
| distributors | Array of strings | Distributors to add                                      | No       |
| padding      | string           | Ignored string used to maintain constant-length messages | Yes      |

##### SetDistributors
Sets a new list of allowed distributors.

| Name         | Type             | Description                                              | Optional |
|--------------|------------------|----------------------------------------------------------|----------|
| distributors | Array of strings | Distributors to set                                      | No       |
| padding      | string           | Ignored string used to maintain constant-length messages | Yes      |

#### Queries

#### Distributors
Returns a list of allowed distributors

##### Response
```json
{
  "distributors": ["List of addresses"]
}
```