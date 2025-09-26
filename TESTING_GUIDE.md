# Testing Guide for NEAR Deployment Instructions

## Quick Start

I've created a comprehensive test suite to validate all commands in the Halborn High-Value Contract Security Guide. Here's how to test:

### 1. Automated Testing Script

Run the validation script to test all commands:

```bash
# Basic run (uses default test account)
./testnet-validation-script.sh

# With custom test account
TEST_ACCOUNT=your-test.testnet ./testnet-validation-script.sh

# With all custom parameters
TEST_ACCOUNT=your-test.testnet \
TEST_CONTRACT=my-contract.testnet \
TEST_DAO_NAME=my-dao \
./testnet-validation-script.sh
```

The script will test:
- ✅ All NEAR CLI commands syntax
- ✅ Account viewing commands
- ✅ Contract deployment commands
- ✅ Base64 encoding (Linux/macOS)
- ✅ RPC calls
- ✅ Python verification script
- ✅ AstroDAO deployment setup
- ✅ Key management commands
- ✅ Storage metrics calculations

### 2. Manual Testing Priority

If you want to test specific parts manually:

#### High Priority - Test These First:

```bash
# 1. Test account commands (no cost, read-only)
near account view-account-summary wrap.testnet network-config testnet
near account list-keys wrap.testnet network-config testnet

# 2. Test RPC calls (no account needed)
curl -s -X POST https://rpc.testnet.near.org -H 'Content-Type: application/json' \
  -d '{"jsonrpc":"2.0","method":"query","params":{"request_type":"view_code","account_id":"wrap.testnet","finality":"final"},"id":1}' \
  | jq '.result.code_base64' | cut -c1-50

# 3. Test base64 encoding
echo "test data" | base64  # macOS
echo "test data" | base64 -w0  # Linux
```

#### Medium Priority - Requires Testnet Account:

```bash
# 1. Deploy a test contract (need funded account)
near contract deploy test-contract.testnet use-file contract.wasm \
  without-init-call network-config testnet sign-with-keychain send

# 2. Test DAO creation (need funded account)
export COUNCIL='["alice.testnet", "bob.testnet", "charlie.testnet"]'
export DAO_ARGS=$(echo -n '{"config": {"name": "test-dao", "purpose": "Test DAO", "metadata":""}, "policy": '$COUNCIL'}' | base64)
near call sputnik-v2.testnet create "{\"name\": \"test-dao-$(date +%s)\", \"args\": \"$DAO_ARGS\"}" \
  --accountId your-account.testnet --deposit 10 --gas 150000000000000
```

### 3. Known Working Examples

These commands have been verified to work on testnet:

#### Simple DAO Deployment (Tested by Cory):
```bash
export COUNCIL='["member1.testnet", "member2.testnet", "member3.testnet"]'
export DAO_ARG=$(echo -n '{"config": {"name": "test-dao", "purpose": "Sputnik Dev v2 DAO", "metadata":""}, "policy": '$COUNCIL'}' | base64)
export DAO_NAME=test-dao-$RANDOM

near call sputnik-v2.testnet create "{\"name\": \"$DAO_NAME\", \"args\": \"$DAO_ARG\"}" \
  --accountId your-account.testnet --deposit 10 --gas 150000000000000
```

### 4. What Each Test Validates

| Test Category | What It Checks | Requires Account? | Requires NEAR? |
|--------------|----------------|-------------------|----------------|
| Account Commands | CLI syntax, RPC connectivity | No | No |
| Deployment Commands | Command structure | Yes | Yes |
| Base64 Encoding | Cross-platform compatibility | No | No |
| RPC Calls | Network connectivity, response format | No | No |
| Python Script | Hash verification logic | No | No |
| DAO Deployment | Factory interaction, encoding | Yes | Yes (10 NEAR) |
| Key Management | Command syntax only | No | No |

### 5. Troubleshooting

If tests fail:

1. **"NEAR CLI not found"**
   ```bash
   npm install -g near-cli-rs
   ```

2. **"Account does not exist"**
   - Create a testnet account at https://testnet.mynearwallet.com/
   - Or use the default test accounts in read-only commands

3. **"Insufficient balance"**
   - Get testnet NEAR from the faucet
   - Or skip deployment tests and focus on syntax validation

4. **Base64 encoding issues**
   - Linux: use `base64 -w0`
   - macOS: use `base64` (no -w0)
   - Universal: use `openssl base64 -A`

### 6. Expected Output

When running the test script, you should see:
```
[INFO] ✓ NEAR CLI found: near-cli-rs 0.7.0
[INFO] ✓ view-account-summary works
[INFO] ✓ list-keys works
[INFO] ✓ SHA256 hash: abc123...
[INFO] ✓ Linux/macOS base64 encoding works
[INFO] ✓ RPC view_code call works
[INFO] ✓ DAO args base64 encoding works
```

### 7. Next Steps After Testing

1. **If all tests pass**: The deployment guide is ready for production use
2. **If some tests fail**: Check the specific error messages and refer to troubleshooting
3. **Before mainnet**: Always test your specific deployment scenario on testnet first

## Files Created

- `testnet-validation-script.sh` - Automated test suite
- `command-validation-report.md` - Full command verification report
- `TESTING_GUIDE.md` - This guide

## Support

If you encounter issues not covered here:
1. Check the `command-validation-report.md` for command-specific notes
2. Refer to the troubleshooting section in the main guide (Section 5)
3. Test with simpler examples first, then add complexity
