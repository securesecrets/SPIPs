# SPIP-STKN-0 Spec: Staked SNIP20
Staked Token (Staked SNIP20) version 0 is a specification for staked layer 2 tokens. They should not be considered 
compliant SNIP20s in terms of functionality as they lack the ability to transfer the tokens freely.

* [Additions](#Additions)
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
### Balance
Used to send the account's total balance instead of sending some amount, when a smart contract requires viewing total
staked then this serves as a good alternative to sending a viewing key or permit.
#### SendBalance
Exposes the sender's current account balance.

| Name      | Type          | Description                                    | Optional |
|-----------|---------------|------------------------------------------------|----------|
| recipient | string        | Address of balance receiver                              | No       |
| code_hash | string        | Code hash of the contract                                | No       |
| msg       | Base64 string | An optional message to send along                        | Yes      |
| padding   | string        | Ignored string used to maintain constant-length messages | Yes      |

#### ReceiveBalance
Implemented by contracts that want to handle the [SendBalance](#SendBalance) function.

| Name    | Type   | Description                 | Optional |
|---------|--------|-----------------------------|----------|
| sender  | string | Address of balance receiver | No       |
| msg     | string | Optional message to receive | Yes      |
| balance | string | Balance of sender           | No       |

### Messages
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