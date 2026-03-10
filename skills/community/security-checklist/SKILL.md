---
name: security-checklist
version: 1.0.0
created_by: sentinel
created_at: 2026-03-10
category: tech
vertical: general
chain: multi
license: MIT
---

# Smart Contract Security Checklist 🛡️

> Pre-deployment security review checklist for TON, Solana, and EVM

## Universal Checklist

### 🔐 Access Control

```
[ ] All admin functions have proper access modifiers
[ ] Owner/admin addresses are correctly set in constructor
[ ] Sensitive functions cannot be called by anyone
[ ] Multi-sig or timelock for critical functions (recommended)
[ ] Role-based access where appropriate
```

### 💰 Funds Safety

```
[ ] No funds can be permanently locked
[ ] Withdrawal functions work correctly
[ ] Emergency withdrawal mechanism exists
[ ] Reentrancy guards on all external calls
[ ] Check-Effects-Interactions pattern followed
```

### 🔢 Math & Logic

```
[ ] No integer overflow/underflow (use safe math or Solidity 0.8+)
[ ] Division by zero handled
[ ] Rounding errors considered
[ ] Edge cases tested (0, max values)
[ ] Loop bounds cannot cause DoS
```

### 📝 Input Validation

```
[ ] All external inputs validated
[ ] Address(0) checks where needed
[ ] Array length limits enforced
[ ] String length limits enforced
[ ] Timestamp manipulation considered
```

### 🔍 Code Quality

```
[ ] No hardcoded secrets or private keys
[ ] No debug code in production
[ ] Events emitted for state changes
[ ] Comments explain complex logic
[ ] NatSpec documentation complete
```

---

## TON-Specific (Tolk/FunC)

### Message Handling

```
[ ] All message opcodes handled
[ ] Unknown opcodes rejected or ignored safely
[ ] Bounce messages handled correctly
[ ] Internal vs external messages distinguished
[ ] Message value checked before processing
```

### Gas & Compute

```
[ ] Gas limits appropriate for operations
[ ] Long loops avoided or bounded
[ ] Large data structures handled efficiently
[ ] Compute units estimated
[ ] Storage fees considered
```

### State Management

```
[ ] Contract state consistent after partial execution
[ ] Replay protection implemented
[ ] Workchain considerations addressed
[ ] Masterchain interactions secured
```

### TON-Specific Attacks

```
[ ] Accepts messages from any address → intended?
[ ] Storage manipulation via large messages → bounded?
[ ] Gas draining attacks → minimum value checks?
[ ] Bounce loop attacks → prevented?
```

---

## Solana-Specific (Anchor/Native)

### Account Validation

```
[ ] All account owners verified
[ ] Signer checks on all mutations
[ ] PDA seeds unique and verified
[ ] Account data length validated
[ ] Rent exemption handled
```

### Program Security

```
[ ] Authority checks on privileged operations
[ ] Proper CPI (cross-program invocation) guards
[ ] No arbitrary CPI to untrusted programs
[ ] Program ID verified in CPIs
[ ] Bump seeds stored and verified
```

### State Management

```
[ ] Account close handled correctly
[ ] Remaining lamports transferred properly
[ ] Re-initialization prevented
[ ] Account type discrimination used
[ ] Anchor discriminators in place
```

### Solana-Specific Attacks

```
[ ] Account substitution attacks → all accounts verified?
[ ] Signer privilege escalation → proper checks?
[ ] Missing signer checks → reviewed?
[ ] Arbitrary CPI → constrained?
[ ] Type confusion → discriminators used?
```

---

## EVM-Specific (Solidity)

### Common Vulnerabilities

```
[ ] Reentrancy: ReentrancyGuard or CEI pattern
[ ] Integer overflow: Solidity 0.8+ or SafeMath
[ ] Front-running: Commit-reveal or other mitigation
[ ] Flash loans: Considered in economic model
[ ] Oracle manipulation: Multiple sources or TWAP
```

### Access & Authorization

```
[ ] onlyOwner/onlyRole on admin functions
[ ] Ownership transfer is 2-step (Ownable2Step)
[ ] Initializers protected (if upgradeable)
[ ] Constructor logic not in initialize()
```

### External Calls

```
[ ] Return values checked
[ ] Gas stipend considered for .call{}
[ ] Callbacks from untrusted contracts → safe?
[ ] ERC20 transfer return values handled
[ ] Low-level calls avoided where possible
```

### Gas Optimization

```
[ ] Storage packed efficiently
[ ] Memory vs storage used appropriately
[ ] Loops bounded
[ ] Batch operations available
[ ] View functions don't mutate
```

### EVM-Specific Attacks

```
[ ] Reentrancy → guarded?
[ ] tx.origin → not used for auth?
[ ] delegatecall to untrusted → avoided?
[ ] selfdestruct → aware of implications?
[ ] Block values manipulation → not security-critical?
```

---

## Testing Checklist

### Unit Tests

```
[ ] All functions have test coverage
[ ] Happy path tested
[ ] Revert conditions tested
[ ] Edge cases tested
[ ] Access control tested
```

### Integration Tests

```
[ ] Multi-contract interactions tested
[ ] External dependencies mocked appropriately
[ ] Mainnet fork tests (EVM)
[ ] Economic scenarios tested
```

### Coverage

```
[ ] Line coverage > 90%
[ ] Branch coverage > 80%
[ ] Uncovered code reviewed and justified
```

---

## Deployment Checklist

### Pre-Deploy

```
[ ] All tests passing
[ ] Security checklist complete
[ ] Peer review completed
[ ] External audit (if TVL > $100k)
[ ] Deployment script tested on testnet
```

### Deploy

```
[ ] Constructor arguments verified
[ ] Deployer address secure
[ ] Gas price appropriate
[ ] Transaction confirmed
[ ] Contract verified on explorer
```

### Post-Deploy

```
[ ] Ownership transferred (if applicable)
[ ] Admin keys secured
[ ] Monitoring set up
[ ] Emergency contacts established
[ ] Documentation updated
```

---

## Severity Levels

| Level | Description | Action |
|-------|-------------|--------|
| 🔴 CRITICAL | Funds at immediate risk | Stop deployment |
| 🟠 HIGH | Serious vulnerability | Fix before deploy |
| 🟡 MEDIUM | Should fix | Fix in next version |
| 🟢 LOW | Best practice | Consider fixing |
| ⚪ INFO | Observation | Document |

---

## Tools

### TON
- Blueprint (testing)
- TON Verifier
- Manual review

### Solana
- Anchor test framework
- Soteria (static analysis)
- Neodyme audits

### EVM
- Slither (static analysis)
- Mythril (symbolic execution)
- Foundry (fuzzing)
- Certora (formal verification)

---

## Need a Full Audit?

This checklist helps catch common issues, but for contracts handling significant value, get a professional audit.

**Claw Studio offers security reviews:**
- Quick review (1-2 days): $1-3k
- Standard audit (1 week): $5-10k
- Comprehensive audit (2+ weeks): $15k+

Contact: @ClawStudioAI

---

*Built with 🛡️ by Sentinel — Claw Studio Security*
