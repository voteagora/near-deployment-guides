# Staging deployment

**NOT LIVE**

## Security council

Security council uses `hos-root.sputnik-dao.near` DAO contract.

See details: https://hos-root.near.page/

Members: https://hos-root.near.page/?page=settings&tab=members

## Contracts

Github commit: [4c9079df73020b9e35dc807146404f7415b0a0be](https://github.com/fastnear/house-of-stake-contracts/tree/4c9079df73020b9e35dc807146404f7415b0a0be) -- v1.0.1

| Account ID | Description | Ownership | Explorer | Contract Hash |
| - | - | - | - | - |
| stsaingdao.near | Top level account for all governance contracts | NEAR Foundation | https://nearblocks.io/address/stagingdao.near | - |
| venear.stagingdao.near | Front contract for venear lockups | hos-root.sputnik-dao.near | https://nearblocks.io/address/vnear.stagingdao.near | 3hGeRfDqDzPBpXyDrnCTMoBTdP2Ly4AypjemR6uebj3G |
| vote.stagingdao.near | Voting contract | hos-root.sputnik-dao.near | https://nearblocks.io/address/vote.stagingdao.near | 8AgTdvpLpJcYrGJK3jcS718adCwiTXYRRA5Qx4pT6xqd |
| - | Lockup contract | venear.stagingdao.near | - | EV4eXNuKVkcYisktcT4sk9XfFFRvcefy51Qs2hQkhnK1 |

## Parameters

```bash
CHAIN_ID=mainnet
CONTRACTS_SOURCE=release

# Owner is Security Council DAO
OWNER_ACCOUNT_ID=hos-root.sputnik-dao.near

# Lockup deployer - security council + NF
LOCKUP_DEPLOYER_ACCOUNT_ID=hos-root.sputnik-dao.near

# Global whitelist managed by NF
STAKING_POOL_WHITELIST_ACCOUNT_ID=lockup-whitelist.near

# !!! CHANGED FROM MAINNET !!! Delays
VOTING_DURATION_NS=1800000000000 # 30 minute in ns
UNLOCK_DURATION_NS=60000000000 # 1 minute in ns

# Guardians that can pause contracts
export GUARDIAN_ACCOUNT_IDS='["as.near", "c65255255d689f74ae46b0a89f04bbaab94d3a51ab9dc4b79b1e9b61e7cf6816","e953bb69d1129e4da87b99739373884a0b57d5e64a65fdc868478f22e6c31eac", "fastnear-hos.near", "lane.near", "root.near"]'

# Reviewers for proposals: security council + screening committee
export REVIEWER_ACCOUNT_IDS='["as.near","c65255255d689f74ae46b0a89f04bbaab94d3a51ab9dc4b79b1e9b61e7cf6816","e953bb69d1129e4da87b99739373884a0b57d5e64a65fdc868478f22e6c31eac", "fastnear-hos.near", "lane.near", "root.near", "guiwickb.near", "gauntletgov.near"]'

# Governance power growing over time (VENEAR_GROWTH_NUMERATOR/VENEAR_GROWTH_DENOMINATOR)*(1B*365*60*60*24)
VENEAR_GROWTH_NUMERATOR=15850000000000 # 50% per annum linear, calculated in ns

# Deposit and fee parameters
LOCAL_DEPOSIT=100000000000000000000000 # 0.1N, enough for 10000 bytes
MIN_LOCKUP_DEPOSIT=2000000000000000000000000 # 2N
BASE_PROPOSAL_FEE=100000000000000000000000 # 0.1N
VOTE_STORAGE_FEE=1250000000000000000000 # 0.00125N
```

## Deployment

```bash
VENEAR_ACCOUNT_ID=venear.stagingdao.near
VOTING_ACCOUNT_ID=vote.stagingdao.near
```

### venear.stagingdao.near

```bash
near contract deploy $VENEAR_ACCOUNT_ID use-file res/$CONTRACTS_SOURCE/venear_contract.wasm with-init-call new json-args '{
  "config": {
    "unlock_duration_ns": "'$UNLOCK_DURATION_NS'",
    "staking_pool_whitelist_account_id": "'$STAKING_POOL_WHITELIST_ACCOUNT_ID'",
    "lockup_code_deployers": ["hos-root.sputnik-dao.near", "fastnear-hos.near", "voteagora.near", "root.near"],
    "local_deposit": "'$LOCAL_DEPOSIT'",
    "min_lockup_deposit": "'$MIN_LOCKUP_DEPOSIT'",
    "owner_account_id": "'$OWNER_ACCOUNT_ID'",
    "guardians": ["as.near", "c65255255d689f74ae46b0a89f04bbaab94d3a51ab9dc4b79b1e9b61e7cf6816","e953bb69d1129e4da87b99739373884a0b57d5e64a65fdc868478f22e6c31eac", "fastnear-hos.near", "lane.near", "root.near"]
  },
  "venear_growth_config": {
    "annual_growth_rate_ns": {
      "numerator": "'$VENEAR_GROWTH_NUMERATOR'",
      "denominator": "1000000000000000000000000000000"
    }
  }
}' prepaid-gas '10.0 Tgas' attached-deposit '0 NEAR' network-config $CHAIN_ID sign-with-keychain send
```

### vote.stagingdao.near

```bash
near contract deploy $VOTING_ACCOUNT_ID use-file res/$CONTRACTS_SOURCE/voting_contract.wasm with-init-call new json-args '{
  "config": {
    "venear_account_id": "'$VENEAR_ACCOUNT_ID'",
    "reviewer_ids": ["as.near","c65255255d689f74ae46b0a89f04bbaab94d3a51ab9dc4b79b1e9b61e7cf6816","e953bb69d1129e4da87b99739373884a0b57d5e64a65fdc868478f22e6c31eac", "fastnear-hos.near", "lane.near", "root.near", "guiwickb.near", "gauntletgov.near"],
    "owner_account_id": "'$OWNER_ACCOUNT_ID'",
    "voting_duration_ns": "'$VOTING_DURATION_NS'",
    "max_number_of_voting_options": 16,
    "base_proposal_fee": "'$BASE_PROPOSAL_FEE'",
    "vote_storage_fee": "'$VOTE_STORAGE_FEE'",
    "guardians": ["as.near", "c65255255d689f74ae46b0a89f04bbaab94d3a51ab9dc4b79b1e9b61e7cf6816","e953bb69d1129e4da87b99739373884a0b57d5e64a65fdc868478f22e6c31eac", "fastnear-hos.near", "lane.near", "root.near"]
  }
}' prepaid-gas '10.0 Tgas' attached-deposit '0 NEAR' network-config $CHAIN_ID sign-with-keychain send
```

### Set lockup contract

```bash
near contract call-function as-transaction $VENEAR_ACCOUNT_ID prepare_lockup_code file-args res/$CONTRACTS_SOURCE/lockup_contract.wasm prepaid-gas '100.0 Tgas' attached-deposit '1.98 NEAR' sign-as $LOCKUP_DEPLOYER_ACCOUNT_ID network-config $CHAIN_ID
```

### Clean up keys

```bash
near account list-keys venear.stagingdao.near network-config mainnet now
near account delete-keys venear.stagingdao.near public-keys <KEY> network-config mainnet

near account list-keys vote.stagingdao.near network-config mainnet now
near account delete-keys vote.stagingdao.near public-keys <KEY> network-config mainnet
```

## Configuration

### venear.stagingdao.near

```bash
near contract call-function as-read-only venear.stagingdao.near get_config json-args {} network-config mainnet now
{
  "guardians": [
    "as.near",
    "c65255255d689f74ae46b0a89f04bbaab94d3a51ab9dc4b79b1e9b61e7cf6816",
    "e953bb69d1129e4da87b99739373884a0b57d5e64a65fdc868478f22e6c31eac",
    "fastnear-hos.near",
    "lane.near",
    "root.near"
  ],
  "local_deposit": "100000000000000000000000",
  "lockup_code_deployers": [
    "hos-root.sputnik-dao.near",
    "fastnear-hos.near",
    "voteagora.near",
    "root.near"
  ],
  "lockup_contract_config": null,
  "min_lockup_deposit": "2000000000000000000000000",
  "owner_account_id": "hos-root.sputnik-dao.near",
  "proposed_new_owner_account_id": null,
  "staking_pool_whitelist_account_id": "lockup-whitelist.near",
  "unlock_duration_ns": "60000000000"
}
```

### vote.stagingdao.near

```bash
near contract call-function as-read-only vote.stagingdao.near get_config json-args {} network-config mainnet now
{
  "base_proposal_fee": "100000000000000000000000",
  "guardians": [
    "as.near",
    "c65255255d689f74ae46b0a89f04bbaab94d3a51ab9dc4b79b1e9b61e7cf6816",
    "e953bb69d1129e4da87b99739373884a0b57d5e64a65fdc868478f22e6c31eac",
    "fastnear-hos.near",
    "lane.near",
    "root.near"
  ],
  "max_number_of_voting_options": 16,
  "owner_account_id": "hos-root.sputnik-dao.near",
  "proposed_new_owner_account_id": null,
  "reviewer_ids": [
    "as.near",
    "c65255255d689f74ae46b0a89f04bbaab94d3a51ab9dc4b79b1e9b61e7cf6816",
    "e953bb69d1129e4da87b99739373884a0b57d5e64a65fdc868478f22e6c31eac",
    "fastnear-hos.near",
    "lane.near",
    "root.near",
    "guiwickb.near",
    "gauntletgov.near"
  ],
  "venear_account_id": "venear.stagingdao.near",
  "vote_storage_fee": "1250000000000000000000",
  "voting_duration_ns": "1800000000000"
}
```
