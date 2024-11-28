# Bridging

Some considerations around bridging

## Traditional Bridging Flow

- User locks tokens in Chain A bridge contract
- Original tokens are held in custody
- Wrapped version minted on Chain B
- Result: User has wrapped tokens on Chain B

## Multi-chain Native Token Flow (aimed approach)

- User initiates transfer from Chain A to Chain B
- Validators verify request
- On Chain A: Tokens are burned (or locked)
- SUI Coordinator updates global state
- On Chain B: Same amount of native tokens minted
- Result: User has native tokens on Chain B

## Comparative

| Aspect | Traditional Bridge | Multi-chain Native |
|---------|-------------------|------------------|
| **Token Type** | • Wrapped token<br>• Different contract<br>• Limited functionality | • Native token<br>• Same token standard<br>• Full chain features |
| **Custody** | • Tokens locked in pool<br>• Requires trust in custody | • Tokens burned/minted<br>• No custody needed |
| **Functionality** | • Limited to wrap features<br>• Extra approval needed | • Full native features<br>• Direct usage in DeFi |
| **User Experience** | • Multiple token versions<br>• Complex interactions<br>• Bridge approval needed | • Single token version<br>• Seamless usage<br>• Direct transfers |

Example Transaction:

```python
# Traditional Bridge
1. approve(bridge_address, amount)
2. bridge.lock_tokens(amount)
3. wait_for_validation()
4. receive_wrapped_tokens()
5. approve(dex, wrapped_amount)  # Extra step for DeFi

# Multi-chain Native
1. token.transfer_cross_chain(target_chain, amount)
2. wait_for_validation()
3. tokens_ready_on_target_chain()  # Native tokens
```

## Difficulties in implementing a non-wrapped token bridge

### 1. State Consistency Challenge

```
                Global State (SUI)
                ┌─────────────────┐
                │ Total Supply: X │
                └─────────────────┘
                        │
        ┌───────────────┴───────────────┐
        ▼                               ▼
    Chain A State                    Chain B State
    ┌─────────────────┐            ┌─────────────────┐
    │ Supply: X1      │            │ Supply: X2      │
    └─────────────────┘            └─────────────────┘
```

Problems:

- Race conditions in cross-chain state updates
- Network delays causing temporary inconsistencies
- Need atomic operations across multiple chains
- Must maintain at all times Total Supply = Supply Chain 1 + Supply Chain 2 + ...

### 2. Security Risks

```python
# Potential Attack Scenarios
class SecurityRisks:
    def double_spend():
        # 1. Start transfer on Chain A
        # 2. Exploit validation delay
        # 3. Use tokens on Chain A before burn
        # 4. Receive tokens on Chain B
        # Result: Tokens exist on both chains

    def replay_attack():
        # 1. Capture valid bridge transaction
        # 2. Replay same transaction later
        # Result: Unauthorized minting
```

### 3. Chain-Specific Standards Compliance

```solidity
// Need to implement for each chain
interface EthereumToken {
    function transfer(address to, uint256 amount) external returns (bool);
    // ERC20 standards
}

interface SolanaToken {
    // SPL token standards
}

interface CosmosToken {
    // Cosmos token standards
}
```

### 4. Complex Validation Logic

```python
class ValidatorNode:
    async def validate_cross_chain_transfer(self, transfer):
        # Must verify:
        # 1. Source chain balance
        # 2. Destination chain capacity
        # 3. Global state consistency
        # 4. User signatures
        # 5. Previous pending operations
        
        if not self.verify_all_conditions():
            raise ValidationError()
```

### 5. Recovery Mechanisms

```python
class BridgeRecovery:
    def handle_failed_transfer():
        # What happens if:
        # - Burn succeeds but mint fails?
        # - Network partition during transfer?
        # - Chain fork occurs?
        
    def emergency_shutdown():
        # Need safe shutdown mechanism
        # Must preserve user funds
```

### 6. Implementation Complexity

```python
class NativeBridge:
    def __init__(self):
        self.chains = {}
        self.validators = []
        self.global_state = GlobalState()
        
    async def process_transfer(self, transfer):
        # Need to coordinate:
        # 1. Source chain operation
        await self.burn_tokens(transfer.source_chain)
        
        # 2. Validator consensus
        consensus = await self.get_validator_consensus()
        
        # 3. State update on SUI
        await self.update_global_state()
        
        # 4. Destination chain operation
        await self.mint_tokens(transfer.dest_chain)
        
        # 5. Confirmation and finality
        await self.ensure_finality()
```
