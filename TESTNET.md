# TestNet deployment

## Contracts

Github commit: [4c9079df73020b9e35dc807146404f7415b0a0be](https://github.com/fastnear/house-of-stake-contracts/tree/4c9079df73020b9e35dc807146404f7415b0a0be)

| Account ID | Description | Ownership | Explorer | Contract Hash |
| - | - | - | - | - |
| dao-testnet | Top level account for all governance contracts | Illia right now | https://testnet.nearblocks.io/address/dao-testnet | - |
| venear.dao-testnet | Front contract for venear lockups | dao-testnet | https://testnet.nearblocks.io/address/vnear.dao-testnet | 3hGeRfDqDzPBpXyDrnCTMoBTdP2Ly4AypjemR6uebj3G |
| vote.dao-testnet | Voting contract | dao-testnet | https://testnet.nearblocks.io/address/vote.dao-testnet | 8AgTdvpLpJcYrGJK3jcS718adCwiTXYRRA5Qx4pT6xqd |
| - | Lockup contract | venear.dao-testnet | - | EV4eXNuKVkcYisktcT4sk9XfFFRvcefy51Qs2hQkhnK1 |

## Configuration

### venear.dao-testnet

```bash
near contract call-function as-read-only venear.dao-testnet get_config json-args {} network-config testnet now
{
  "guardians": [
    "dao-testnet"
  ],
  "local_deposit": "100000000000000000000000",
  "lockup_code_deployers": [
    "dao-testnet"
  ],
  "lockup_contract_config": {
    "contract_hash": "EV4eXNuKVkcYisktcT4sk9XfFFRvcefy51Qs2hQkhnK1",
    "contract_size": 155781,
    "contract_version": 1
  },
  "min_lockup_deposit": "100000000000000000000000",
  "owner_account_id": "dao-testnet",
  "proposed_new_owner_account_id": null,
  "staking_pool_whitelist_account_id": "whitelist.f863973.m0",
  "unlock_duration_ns": "300000000000"
}
```

### vote.dao-testnet

```bash
near contract call-function as-read-only vote.dao-testnet get_config json-args {} network-config testnet now
{
  "base_proposal_fee": "100000000000000000000000",
  "guardians": [
    "dao-testnet"
  ],
  "max_number_of_voting_options": 16,
  "owner_account_id": "dao-testnet",
  "proposed_new_owner_account_id": null,
  "reviewer_ids": [
    "testmewell.testnet"
  ],
  "venear_account_id": "venear.dao-testnet",
  "vote_storage_fee": "1250000000000000000000",
  "voting_duration_ns": "600000000000"
}
```
