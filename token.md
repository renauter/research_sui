# Chain-specific token contracts

- ERC20/BEP20 compatibility
- Native chain standard compliance
- Mint/burn functions controlled by bridge

## Base Token Structure (Cross-Chain Compatible)

```python
class TokenBase:
    # Core Properties
    struct TokenState:
        name: str
        symbol: str
        decimals: u8
        total_supply: u256
        burn_address: address
        
    # Admin Management
    struct AdminConfig:
        admins: Map<address, bool>
        required_signatures: u8
        
    # Account Management
    struct Account:
        balance: u256
        is_restricted: bool
        signing_policy: SigningPolicy
        
    struct SigningPolicy:
        required_signatures: u8
        signers: Map<address, bool>
```

```solidity
contract ChainToken {
    address public coordinator;
    mapping(address => uint256) public balances;
    
    // Only validated operations allowed
    modifier onlyValidator() {
        require(isValidator[msg.sender], "Not validator");
        _;
    }
    
    // Mint tokens on this chain
    function mint(address to, uint256 amount) 
        external 
        onlyValidator 
    {
        balances[to] += amount;
        emit Mint(to, amount);
    }
    
    // Burn tokens for cross-chain transfer
    function burn(address from, uint256 amount)
        external
        onlyValidator
    {
        require(balances[from] >= amount, "Insufficient balance");
        balances[from] -= amount;
        emit Burn(from, amount);
    }
}
```

## Chain-Specific Implementations

```
├── Ethereum Implementation
│   ├── Language: Solidity/Vyper
│   ├── Interface: ERC20
│   └── Features:
│       ├── Standard token functions
│       ├── Burning mechanism
│       ├── Minting (validator controlled)
│       └── Multi-signature support
│
├── SUI Implementation
│   ├── Language: Rust (Move)
│   ├── Interface: Sui token standard
│   └── Features:
│       ├── Native token functions
│       ├── Coordinator functions
│       └── State management
│
├── Solana Implementation
│   ├── Language: Rust
│   ├── Interface: SPL Token
│   └── Features: [Similar to above]
│
└── Cosmos Implementation
    ├── Language: Go
    ├── Interface: Cosmos SDK
    └── Features: [Similar to above]
```

### SUI

```rust
module token::our_token {
    use sui::object::{Self, UID};
    use sui::transfer;
    use sui::tx_context::{Self, TxContext};
    
    struct OurToken has key {
        id: UID,
        name: vector<u8>,
        symbol: vector<u8>,
        decimals: u8,
        total_supply: u64,
    }
    
    struct Account has store {
        balance: u64,
        is_restricted: bool,
        signing_policy: SigningPolicy,
    }
    
    struct SigningPolicy has store {
        required_signatures: u8,
        signers: vector<address>,
    }
    
    struct AdminCap has key {
        id: UID,
    }
    
    // Initialize token
    public fun init(ctx: &mut TxContext) {
        let admin_cap = AdminCap {
            id: object::new(ctx),
        };
        
        let token = OurToken {
            id: object::new(ctx),
            name: b"OurToken",
            symbol: b"OTK",
            decimals: 8,
            total_supply: 0,
        };
        
        transfer::transfer(admin_cap, tx_context::sender(ctx));
        transfer::share_object(token);
    }
    
    // Transfer with multisig support
    public fun transfer(
        token: &mut OurToken,
        from: &Account,
        to: &mut Account,
        amount: u64,
        signatures: vector<vector<u8>>,
        ctx: &mut TxContext
    ) {
        assert!(verify_signatures(from, signatures), 0);
        assert!(!from.is_restricted, 1);
        
        from.balance = from.balance - amount;
        to.balance = to.balance + amount;
    }
}
```

### Ethereum (Vyper)

```python
# SPDX-License-Identifier: MIT
# @version ^0.3.7

from vyper.interfaces import ERC20

implements: ERC20

# State Variables
name: public(String[32])
symbol: public(String[32])
decimals: public(uint8)
total_supply: public(uint256)

# Advanced State Management
accounts: public(HashMap[address, Account])
admins: public(HashMap[address, bool])
admin_threshold: public(uint8)

# Structs
struct Account:
    balance: uint256
    is_restricted: bool
    signing_policy: SigningPolicy
    
struct SigningPolicy:
    required_sigs: uint8
    signers: HashMap[address, bool]

# Events (ERC20 Compatible)
event Transfer:
    sender: indexed(address)
    receiver: indexed(address)
    amount: uint256

event AdminAction:
    action_type: uint8
    admin: indexed(address)
    target: indexed(address)
    data: bytes32

@external
def __init__():
    self.name = "OurToken"
    self.symbol = "OTK"
    self.decimals = 18
    self.admin_threshold = 1
    self.admins[msg.sender] = True

@external
def transfer(_to: address, _value: uint256, _signatures: DynArray[bytes, 5]) -> bool:
    assert not self.accounts[msg.sender].is_restricted, "Account restricted"
    assert self._verify_signatures(msg.sender, _signatures), "Invalid signatures"
    
    self.accounts[msg.sender].balance -= _value
    self.accounts[_to].balance += _value
    log Transfer(msg.sender, _to, _value)
    return True

@external
def admin_action(_action: uint8, _target: address, _data: bytes32, _signatures: DynArray[bytes, 5]) -> bool:
    assert self._verify_admin_signatures(_signatures), "Invalid admin signatures"
    
    if _action == 1:  # Add Admin
        self.admins[_target] = True
    elif _action == 2:  # Remove Admin
        self.admins[_target] = False
    # ... other admin actions
    
    log AdminAction(_action, msg.sender, _target, _data)
    return True
```
