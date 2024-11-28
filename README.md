# Multichain token research

## Token specs

See related [issue](https://git.ourworld.tf/hero/hero_server_python/src/branch/development/examples/vypertest)

## Architecture

See related [issue](https://git.ourworld.tf/hero/circle_herodev/issues/15)

```
                      SUI Coordinator (Master Contract)
                        ┌────────────────────────────┐
                        │ - Global state             │
                        │ - Cross-chain registry     │
                        │ - Validator management     │
                        └────────────────────────────┘
                                    ▲
                                    │
                    ┌───────────────┴───────────────┐
                    │         Validator Pool        │
                    │     (Minimum 5 Validators)    │
                    └───────────────┬───────────────┘
                                    │
            ┌───────────────┬───────┴────────┬───────────────┐
            ▼               ▼                 ▼               ▼
    Chain A Contract  Chain B Contract  Chain C Contract  Chain N Contract
```

## Token implementation strategy

```
Native Multi-Chain Approach
├── SUI Coordinator
│   ├── Master registry of token holdings
│   └── Bridge contract for cross-chain operations
├── Target Chains
│   ├── Chain-specific token contracts
│   └── Standard-compliant implementations
└── Unified Key Management
    ├── Same public/private key across chains
    └── Address derivation standardization
```

## Cross-Chain operation flow

```
User Request → Chain A Contract → Event Emission
    ↓
Validators Monitor Event
    ↓
Each Validator Signs Operation
    ↓
Signatures Submitted to SUI Coordinator
    ↓
When Threshold Reached (e.g., 3/5 validators)
    ↓
Validators Execute Operation on Target Chain
```

## Technical components

- Primary token contract on SUI → [coordinator](./coordinator.md)
- Secondary token contracts on target chains → [token](./token.md)
- Bridge contract on SUI → [bridge](./bridge.md)
- Validator/Monitor service (V tool) → [validator](./validator.md)

## Implementation phases

### Phase 1: Core infrastructure

- SUI coordinator contract
- Basic bridge functionality
- Single chain integration test

### Phase 2: Multi-chain support

- Additional chain implementations
- Validator network setup
- Cross-chain testing

### Phase 3: DeFi integration

- Standard compliance verification
- DeFi protocol testing
- Security audits

### Phase 4: Production release

- Mainnet deployment
- Monitoring setup
- Emergency procedures

## Other considerations

### Key management solution

For unified key management across chains:

```solidity
contract MultiChainToken {
    mapping(address => bool) public registeredAddresses;
    
    function registerAddress(bytes memory signature) public {
        // Verify signature matches across chains
        // Register address for token operations
    }
}
```

### DeFi compatibility layer

```solidity
interface IStandardToken {
    // Standard token interface
    function balanceOf(address account) external view returns (uint256);
    function transfer(address recipient, uint256 amount) external returns (bool);
    // Additional standard methods
}

contract ChainSpecificToken is IStandardToken {
    // Implementation of standard interface
    // Additional chain-specific features
}
```

### Security

- Multi-signature requirements
- Timelock mechanisms
- Oracle dependency management
- Bridge attack vectors
