# Validator service

- Watch for bridge events
- Cross-chain verification
- Handle token minting/burning

```python
class ValidatorNode:
    def __init__(self):
        self.chains = {}  # Connection to each chain
        self.sui_coordinator = None  # Connection to SUI
        self.private_key = None  # Validator's key
        
    async def monitor_chains(self):
        while True:
            # Monitor all chains for token operations
            for chain in self.chains.values():
                events = await chain.get_events()
                for event in events:
                    await self.process_event(event)
                    
    async def process_event(self, event):
        # Validate event
        if self.verify_event(event):
            # Sign operation
            signature = self.sign_operation(event)
            # Submit to coordinator
            await self.sui_coordinator.submit_signature(
                event.op_id, signature)
            
    async def execute_operation(self, op):
        # When enough signatures collected
        if len(op.signatures) >= THRESHOLD:
            if op.action == "MINT":
                await self.execute_mint(op)
            elif op.action == "BURN":
                await self.execute_burn(op)

class ValidatorProtocol:
    async def broadcast_signature(self, op_id: str, signature: bytes):
        # Broadcast to other validators
        for validator in self.peer_validators:
            await validator.receive_signature(op_id, signature)
            
    async def achieve_consensus(self, operation: CrossChainOp):
        signatures = []
        # Collect signatures until threshold
        while len(signatures) < THRESHOLD:
            sig = await self.wait_for_signature(operation.op_id)
            if self.verify_signature(sig):
                signatures.append(sig)
```
