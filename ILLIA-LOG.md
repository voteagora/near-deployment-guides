
# Deployment log

git clone git@github.com:fastnear/house-of-stake-contracts.git


## Actual configuration

### Contract builds of 1.0.1

Github commit: 4c9079df73020b9e35dc807146404f7415b0a0be

venear hex: 28084bded782740487ef5f5663189bf220d4947ae8895d0e0658534b154e72cb
bs58: 3hGeRfDqDzPBpXyDrnCTMoBTdP2Ly4AypjemR6uebj3G

lockup_contract hex: c85850faee618a88f1af3af73054daad64bbe275ff099adf280b1cc7f932f938
bs58: EV4eXNuKVkcYisktcT4sk9XfFFRvcefy51Qs2hQkhnK1

voting contract hex: 6a7ca70952b8e1a39c02afbe758c2fb22b744431a0e72d33b7bc5c5d2fbd83c8
bs58: 8AgTdvpLpJcYrGJK3jcS718adCwiTXYRRA5Qx4pT6xqd

### TestNet

(near doesn't work with registrar account)
 npx near-cli create-account dao-testnet --masterAccount registrar --initialBalance 0.1

near tokens near send-near dao-testnet '1000 NEAR' network-config testnet sign-with-keychain send

 near --quiet account create-account fund-myself venear.dao-testnet '2.4 NEAR' autogenerate-new-keypair save-to-keychain sign-as dao-testnet network-config testnet sign-with-keychain send

 near --quiet account create-account fund-myself vote.dao-testnet '2.4 NEAR' autogenerate-new-keypair save-to-keychain sign-as dao-testnet network-config testnet sign-with-keychain send

---

```bash
export CONTRACTS_SOURCE=release

export VENEAR_ACCOUNT_ID=venear.dao-testnet
export VOTING_ACCOUNT_ID=vote.dao-testnet
export CHAIN_ID=testnet

export LOCKUP_DEPLOYER_ACCOUNT_ID=dao-testnet
# 5 minutes unlock for testing
export UNLOCK_DURATION_NS=300000000000
# 0.1N
export LOCAL_DEPOSIT=100000000000000000000000
export OWNER_ACCOUNT_ID=dao-testnet
# testnet whitelist
export STAKING_POOL_WHITELIST_ACCOUNT_ID=whitelist.f863973.m0

export MIN_LOCKUP_DEPOSIT=100000000000000000000000

export GUARDIAN_ACCOUNT_ID=dao-testnet

# growth_config - 6% annual

## command

 near contract deploy $VENEAR_ACCOUNT_ID use-file res/$CONTRACTS_SOURCE/venear_contract.wasm with-init-call new json-args '{
  "config": {
    "unlock_duration_ns": "'$UNLOCK_DURATION_NS'",
    "staking_pool_whitelist_account_id": "'$STAKING_POOL_WHITELIST_ACCOUNT_ID'",
    "lockup_code_deployers": ["'$LOCKUP_DEPLOYER_ACCOUNT_ID'"],
    "local_deposit": "'$LOCAL_DEPOSIT'",
    "min_lockup_deposit": "'$MIN_LOCKUP_DEPOSIT'",
    "owner_account_id": "'$OWNER_ACCOUNT_ID'",
    "guardians": ["'$GUARDIAN_ACCOUNT_ID'"]
  },
  "venear_growth_config": {
    "annual_growth_rate_ns": {
      "numerator": "1902587519026",
      "denominator": "1000000000000000000000000000000"
    }
  }
}' prepaid-gas '10.0 Tgas' attached-deposit '0 NEAR' network-config $CHAIN_ID sign-with-keychain send

## actually executed

near contract deploy venear.dao-testnet use-file res/release/venear_contract.wasm with-init-call new json-args '{
  "config": {
    "unlock_duration_ns": "300000000000",
    "staking_pool_whitelist_account_id": "whitelist.f863973.m0",
    "lockup_code_deployers": ["dao-testnet"],
    "local_deposit": "100000000000000000000000",
    "min_lockup_deposit": "100000000000000000000000",
    "owner_account_id": "dao-testnet",
    "guardians": ["dao-testnet"]
  },
  "venear_growth_config": {
    "annual_growth_rate_ns": {
      "numerator": "1902587519026",
      "denominator": "1000000000000000000000000000000"
    }
  }
}' prepaid-gas '10.0 Tgas' attached-deposit '0 NEAR' network-config testnet sign-with-keychain send
```
---

```bash
export REVIEWER_ACCOUNT_ID=testmewell.testnet

# voting period 10min
export VOTING_DURATION_NS=600000000000
# 0.1N
export BASE_PROPOSAL_FEE=100000000000000000000000
# 0.00125N
export VOTE_STORAGE_FEE=1250000000000000000000

# command

near contract deploy $VOTING_ACCOUNT_ID use-file res/$CONTRACTS_SOURCE/voting_contract.wasm with-init-call new json-args '{
  "config": {
    "venear_account_id": "'$VENEAR_ACCOUNT_ID'",
    "reviewer_ids": ["'$REVIEWER_ACCOUNT_ID'"],
    "owner_account_id": "'$OWNER_ACCOUNT_ID'",
    "voting_duration_ns": "'$VOTING_DURATION_NS'",
    "max_number_of_voting_options": 16,
    "base_proposal_fee": "'$BASE_PROPOSAL_FEE'",
    "vote_storage_fee": "'$VOTE_STORAGE_FEE'",
    "guardians": ["'$GUARDIAN_ACCOUNT_ID'"]
  }
}' prepaid-gas '10.0 Tgas' attached-deposit '0 NEAR' network-config $CHAIN_ID sign-with-keychain

## actually executed

near contract deploy vote.dao-testnet use-file res/release/voting_contract.wasm with-init-call new json-args '{
  "config": {
    "venear_account_id": "venear.dao-testnet",
    "reviewer_ids": ["testmewell.testnet"],
    "owner_account_id": "dao-testnet",
    "voting_duration_ns": "600000000000",
    "max_number_of_voting_options": 16,
    "base_proposal_fee": "100000000000000000000000",
    "vote_storage_fee": "1250000000000000000000",
    "guardians": ["dao-testnet"]
  }
}' prepaid-gas '10.0 Tgas' attached-deposit '0 NEAR' network-config testnet sign-with-keychain send

# upload lockup contract

near contract call-function as-transaction $VENEAR_ACCOUNT_ID prepare_lockup_code file-args res/$CONTRACTS_SOURCE/lockup_contract.wasm prepaid-gas '100.0 Tgas' attached-deposit '1.98 NEAR' sign-as $LOCKUP_DEPLOYER_ACCOUNT_ID network-config $CHAIN_ID sign-with-keychain send


```


# TO FIX

## Instructions

Fix astrodao mentions
Instructions menthion build_all vs build_release???
Add proper upgrade instructions
Modern ledger can sign deployment transactions

## Update parameters

Apparently venear growth config was updated. 
Why this number?
7.5%

  "venear_growth_config": {
    "annual_growth_rate_ns": {
      "numerator": "2378234398782",
      "denominator": "1000000000000000000000000000000"
    }
  }

## Repos

https://github.com/fastnear/house-of-stake-contracts/ and this repo must be consolidated under near/house-of-stake

