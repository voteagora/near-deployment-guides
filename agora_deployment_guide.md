# HoS Contract Deployment Bootstrap

### Before getting started

GM! Welcome to the House of Stake. If you are reading this, that means that you will deploying the House of Stake contracts for the NEAR ecosystem. This is a big responsability given that the Hosue of Stake contracts will eventually hold a lot of NEAR. We want to make sure that the processess you and your security council are preforming are done with the utmost security in mind. 

As part of this deployment guide, Agora has received an approved list of security concerns and best practices for serious NEAR contracts from the auditing firm [Halborn](https://www.halborn.com/). It is the author's expectation that before you go and deploy these contracts on Mainnet that you and your team have read and understand the security concerns in this document: [Deploying and Securing High Value Contracts in the NEAR Ecosystem](https://github.com/voteagora/near-runbooks-temp/blob/main/halborn_runbook.md). This document contains many best practices and considerations around how one should deploy high value NEAR contracts. By continuing on with this deployment guide, you are acknoweldging that you have read and understood these security concerns. 

With that, let's get started deploying the House of Stake.

### Introduction

The following guide is a comprehensive set of instructions to enable the NF to bootstrap the governance contracts.

This document is expected to be followed twice, once for the testnet deployment and then again for the mainnet deployment, with the following macro steps:

1. Setup the AstroDAO Multisig on Testnet “Step 1”
2. Deploy HoS Contracts to Testnet “Step 2”
3. Notify Agora about the deployment, Agora will perform UAT on the provided addresses
4. Setup the AstroDAO Multisig on Testnet “Step 1”
5. Deploy HoS Contracts to Testnet “Step 2”
6. Notify Agora about the deployment, Agora will Integrate the provided addresses on the production HoS site

### Step 1 - Deploying a AstroDAO Multisig for the Security Council

Using the GUI: [https://astrodao.com/](https://astrodao.com/)

Follow the full setup instructions for deployment here: https://github.com/near-daos/sputnik-dao-contract/blob/main/README.md

Configure with the roles outlined in the Halborn Runbook (cite **5. DAO Tooling Comparison (AstroDAO vs Custom Multisig))**:

```bash
# Create DAO with initial council
near call factory.astrodao.near create '{
  "name": "nf-security-council",
  "args": {
    "config": {
      "name": "NF Security Council",
      "purpose": "High-value contract governance",
      "metadata": ""
    },
    "policy": {
      "roles": [{
        "name": "council",
        "kind": {"Group": ["alice.near", "bob.near", "charlie.near"]},
        "permissions": ["*:*"],
        "vote_policy": {"weight_kind": "RoleWeight", "quorum": "70", "threshold": [7, 10]}
      }],
      "default_vote_policy": {
        "weight_kind": "RoleWeight",
        "quorum": "70",
        "threshold": [7, 10]
      },
      "proposal_bond": "1000000000000000000000000",
      "proposal_period": "604800000000000",
      "bounty_bond": "0",
      "bounty_forgiveness_period": "0"
    }
  }
}' --accountId deployer.near --deposit 10
```

Security council member account IDs should be supplied here when following the instructions outlined in the Sputnik DAO repo:
`export COUNCIL='["council-member.testnet", "YOUR_ACCOUNT.testnet"]'`

Alternatively, use the configuration example above to start defining roles as advised in the Halborn Runbook. 

### Step 2 - Deploying the HoS Contracts

Prerequisites: 

- NEAR JS CLI installed on a secure machine
- Root accountID - NF Foundation account with keys owned by the security council
- https://github.com/fastnear/house-of-stake-contracts checked out locally
1. Build Contract sources by running `build_all.sh` in the root directory
2. Pre-fund addresses: 
    1. VENEAR_ACCOUNT_ID="v.$ROOT_ACCOUNT_ID"
    2. VOTING_ACCOUNT_ID="vote.$ROOT_ACCOUNT_ID"

Run the commands:

```bash
near --quiet account create-account fund-myself $VENEAR_ACCOUNT_ID '2.4 NEAR' autogenerate-new-keypair save-to-keychain sign-as $ROOT_ACCOUNT_ID network-config $CHAIN_ID sign-with-keychain send
near --quiet account create-account fund-myself $VOTING_ACCOUNT_ID '2.3 NEAR' autogenerate-new-keypair save-to-keychain sign-as $ROOT_ACCOUNT_ID network-config $CHAIN_ID sign-with-keychain send
```

1. Deploy the veNEAR contract, populating all the parameters based on recommendations below.

```bash
near --quiet contract deploy $VENEAR_ACCOUNT_ID use-file res/$CONTRACTS_SOURCE/venear_contract.wasm with-init-call new json-args '{
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
```

**Configuration**

See the original documentation: https://github.com/fastnear/house-of-stake-contracts/blob/main/README.md

- `$LOCKUP_DEPLOYER_ACCOUNT_ID` - Recommendation is for it to be a developer ledger. When you need to deploy a contract, you can add a full-access key to this account, then call method to add a new lockup, then remove the previous full-access key. This is because the ledger might not be able to sign the large binary transaction.
- `$UNLOCK_DURATION_NS` - is `90 * 24 * 60 * 60 * 1_000_000_000` 90 days for lockup unlocks
- `$LOCAL_DEPOSIT` - `0.1 * 10**24`, i.e. `0.1` NEAR to cover storage cost.
- `$OWNER_ACCOUNT_ID` - Given the Halborn runbook this should be the Sputnik DAO account (AstroDAO), for example:`hos-owner.sputnik-dao.near`. The DAO should be secured with multisig like `3/5` or `4/5` and be members of the security council. See the role configurations elaborated in the runbook. Agora as a developer, should have a seat in the multisig or have a separate role with ability to propose new actions.
- `staking_pool_whitelist_account_id` - Cannot be changed once set, it is fixed towards the mainnet `"lockup-whitelist.near"` that manages whitelist of mainnet pools for lockups. For testnet a new whitelist can be deployed or use the existing: `whitelist.f863973.m0`
- `$GUARDIAN_ACCOUNT_ID` - These must be the same list of trusted accounts from security council. It gives ability for every of these accounts to individually pause the HoS contract. Paused contracts stop providing merkle balance proofs and state snapshots, so new voting can't be started. I suggest to include individual accounts from the 5 members of security commission as well as Agora developer accounts. There should be strong intersection between members in the multisig and this role. See the runbook for the ceremonies for pausing the contract.
1. Deploy and configure the Voting Contract

```bash
near --quiet contract deploy $VOTING_ACCOUNT_ID use-file res/$CONTRACTS_SOURCE/voting_contract.wasm with-init-call new json-args '{
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
}' prepaid-gas '10.0 Tgas' attached-deposit '0 NEAR' network-config $CHAIN_ID sign-with-keychain send

near --quiet contract call-function as-transaction $VENEAR_ACCOUNT_ID prepare_lockup_code file-args res/$CONTRACTS_SOURCE/lockup_contract.wasm prepaid-gas '100.0 Tgas' attached-deposit '1.98 NEAR' sign-as $LOCKUP_DEPLOYER_ACCOUNT_ID network-config $CHAIN_ID sign-with-keychain send

CONTRACT_HASH=$(cat res/$CONTRACTS_SOURCE/lockup_contract.wasm | sha256sum | awk '{ print $1 }' | xxd -r -p | base58)
near --quiet contract call-function as-transaction $VENEAR_ACCOUNT_ID set_lockup_contract json-args '{
  "contract_hash": "'$CONTRACT_HASH'",
  "min_lockup_deposit": "'$MIN_LOCKUP_DEPOSIT'"
}' prepaid-gas '20.0 Tgas' attached-deposit '1 yoctoNEAR' sign-as $OWNER_ACCOUNT_ID network-config $CHAIN_ID sign-with-keychain send

```

**Configuration**

See the original configuration: [https://github.com/fastnear/house-of-stake-contracts/blob/main/README.md](https://github.com/fastnear/house-of-stake-contracts/blob/main/README.md)

- `$VENEAR_ACCOUNT_ID` - This should be production account of veNEAR contract
- `$REVIEWER_ACCOUNT_ID` - Configured to be one or more reviewers accounts from the NF Security Council. The role provides moderation of the proposals, to make sure there is no spam and they follow a guideline for new proposals. The list of accounts can be changed by the `OWNER_ACCOUNT_ID`. 

Since each account has ability to approve and reject the transaction, one option is to use Sputnik DAO contract. But that will require a set of reviewers to vote on every valid proposal from the DAO. Another option is to setup list of trusted moderators that would follow the predefined list of rules for reviewing proposals. When in doubt they can discuss in some off-chain channel to reach consensus. The accounts can be secured by a simple ledger. There are no much security risks except for rejecting a valid proposal or approving an invalid one, voting is still required regardless.
- `$OWNER_ACCOUNT_ID` - Should be the same as the veNEAR contract owner.
- `$VOTING_DURATION_NS` - Should be `7 * 24 * 60 * 60 * 1_000_000_000` - 7 days
- `$BASE_PROPOSAL_FEE` - This can be set to `0.1 * 10**24`, it's `0.1` NEAR initially but can be adjusted (increased) if there are too much spam and it's hard to moderate. It's non-refundable fee that the proposer pays in addition to added bytes fee.
- `$VOTE_STORAGE_FEE` - `0.00125 * 10**24`, it's enough for `125 bytes`. I think this contract is slightly less, but it's a safe start.
- `$GUARDIAN_ACCOUNT_ID` - Should be set as the same list as the `veNEAR` contract.
