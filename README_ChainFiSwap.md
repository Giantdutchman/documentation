# ChainFiSwap Contract

## Overview

The **ChainFiSwap** contract is a dedicated cross-chain coordinator for CFI token swaps within the Chain-Fi ecosystem. It acts as an intermediary between user vaults and the CFI token contract, providing secure validation and coordination for cross-chain token transfers.

## Key Features

- ✅ **Cross-Chain Native**: Supports multi-chain CFI token swaps
- 🛡️ **Triple Security Validation**: Registry + Whitelist + Signature verification
- 🤖 **Automatic Supply Management**: Integrates with CFI token's auto-burn/mint system
- 👨‍💼 **Guardian Coordination**: Works with Guardian Network for cross-chain execution
- 📊 **Complete Tracking**: Comprehensive event system for monitoring and validation
- 🔒 **Access Control**: Multiple permission levels for different operations

## Architecture Overview

```
┌─────────────┐    ┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│    User     │    │   ChainFiVault  │    │  ChainFiSwap    │    │   CFI Token     │
│             │    │                 │    │                 │    │                 │
│             │────┤                 │────┤                 │────┤                 │
│   Wallet    │    │ Asset Storage   │    │  Coordinator    │    │ Supply Manager  │
│             │    │ + Security      │    │ + Validation    │    │ + Auto Burn/Mint│
└─────────────┘    └─────────────────┘    └─────────────────┘    └─────────────────┘
                                                  │
                                                  │
                                         ┌─────────────────┐
                                         │ Guardian Network│
                                         │                 │
                                         │ Cross-Chain     │
                                         │ Coordinator     │
                                         └─────────────────┘
```

## Party Interactions

### 1. 👤 **Users**
**Role**: Initiate cross-chain swaps
**Interactions**:
- Call `initiateSwap()` with vault signature
- Provide source vault, target vault, amount, and target chain
- Must own the source vault to authorize transfers

### 2. 🏦 **ChainFi Vaults** 
**Role**: Secure asset storage with transfer authorization
**Interactions**:
- Validate swap contract is whitelisted
- Verify user signatures for token transfers
- Transfer CFI tokens to swap contract when authorized
- Enforce security checks (cooldowns, nonce, etc.)

### 3. 🪙 **CFI Token Contract**
**Role**: Token processing and supply management
**Interactions**:
- Receive tokens from swap contract via `receiveSwapTokens()`
- Auto-burn excess tokens if they exceed circulating supply
- Add valid tokens to swap treasury
- Process Guardian-executed swaps on target chains
- Auto-mint tokens when swap treasury insufficient

### 4. 🛡️ **Guardian Network**
**Role**: Cross-chain coordination and validation
**Interactions**:
- Monitor `SwapInitiated` events across all chains
- Validate token reception with CFI contracts
- Execute swaps on target chains
- Record completion on source chains

### 5. 📋 **Vault Registry**
**Role**: Vault validation and ownership tracking
**Interactions**:
- Validate that source vaults are registered
- Provide vault ownership information
- Ensure only legitimate vaults can initiate swaps

## Complete Swap Workflow

### Phase 1: Initiation (Source Chain)

```solidity
// 1. User calls initiateSwap()
chainFiSwap.initiateSwap(
    sourceVault,     // User's vault address
    targetVault,     // Destination vault on target chain
    amount,          // CFI tokens to swap
    targetChain,     // Target chain ID
    signature        // User's authorization signature
);
```

**Security Validations**:
- ✅ Source vault must be registered in VaultRegistry
- ✅ Target chain must be supported
- ✅ Swap contract must be whitelisted in vault's listing registry
- ✅ User must provide valid signature

**Token Flow**:
```
1. Vault → SwapContract (via approveAndTransferToken)
2. SwapContract approves CFI contract
3. CFI contract pulls tokens via receiveSwapTokens()
4. CFI contract auto-burns excess tokens (if > circulating supply)
5. CFI contract adds remainder to swapTreasuryBalance
```

**Events Emitted**:
```solidity
// From ChainFiSwap
event SwapInitiated(swapId, sourceVault, targetVault, amount, sourceChain, targetChain);

// From CFI Token
event SwapTokensReceived(sourceVault, targetVault, amount, targetChain, swapId);
event AutoBurnForCirculatingSupply(excessAmount, actualAmount, operation);
```

### Phase 2: Guardian Validation

The Guardian Network performs comprehensive validation:

```javascript
// Guardian monitoring workflow
guardian.on('SwapInitiated', async (event) => {
    const { swapId, sourceVault, targetVault, amount, sourceChain, targetChain } = event;
    
    // 1. Validate swap was properly initiated
    const swapValid = await sourceSwapContract.validateSwapForExecution(
        swapId, sourceVault, amount, targetChain
    );
    
    // 2. Cross-validate with CFI contract
    const [isValid, actualProcessed, amountBurned] = await sourceCFIContract.validateSwapReception(
        swapId, sourceVault, targetVault, amount, targetChain
    );
    
    // 3. Check target chain status
    const alreadyProcessed = await targetCFIContract.isSwapProcessed(swapId);
    
    // 4. Execute if all validations pass
    if (swapValid && isValid && !alreadyProcessed) {
        await executeSwapOnTargetChain(swapId, event, actualProcessed);
    }
});
```

**Guardian Validation Points**:
- 🔍 **Swap Integrity**: Verify swap was initiated correctly
- 💰 **Token Reception**: Confirm CFI contract received tokens
- 🔥 **Auto-Burn Accounting**: Track any tokens burned for supply limits
- 🚫 **Duplicate Prevention**: Ensure swap not already executed
- 📊 **Supply Validation**: Verify circulating supply management

### Phase 3: Execution (Target Chain)

```solidity
// Guardian executes on target chain CFI contract
targetCFIContract.executeSwap(
    swapId,          // Unique swap identifier
    sourceVault,     // Original vault from source chain  
    targetVault,     // Destination vault on target chain
    actualAmount,    // Amount after any auto-burning
    sourceChain      // Source chain ID
);
```

**CFI Token Processing**:
```
1. Check swap not already processed
2. Try to use tokens from swapTreasuryBalance first
3. If insufficient treasury → auto-mint needed amount
4. Transfer/mint tokens to target vault
5. Update swap statistics and chain balances
```

**Events Emitted**:
```solidity
event SwapExecuted(sourceVault, targetVault, amount, sourceChain, swapId, treasuryUsed, minted);
event SwapCompleted(swapId, targetVault, amount, sourceChain);
event AutoMintForSwap(amount, targetVault, reason); // If minting was needed
```

### Phase 4: Completion Tracking

```solidity
// Guardian records completion on source chain
sourceSwapContract.recordSwapCompletion(swapId, targetVault, actualAmount);
```

## Guardian Network Role

### 🛡️ **Core Responsibilities**

1. **Event Monitoring**
   - Listen to `SwapInitiated` events across all supported chains
   - Monitor `SwapTokensReceived` events from CFI contracts
   - Track swap completion and failure states

2. **Multi-Source Validation**
   - Cross-validate swap data between swap contracts and CFI contracts
   - Verify token amounts match expected values
   - Account for auto-burned tokens in supply management

3. **Cross-Chain Execution**
   - Execute validated swaps on target chains
   - Ensure proper ordering and prevent duplicate executions
   - Handle edge cases and error recovery

4. **State Reconciliation**
   - Track swap completion across chains
   - Monitor total supply consistency
   - Perform periodic validation of cross-chain states

### 🔧 **Guardian Tools & Functions**

**Validation Functions**:
```solidity
// In ChainFiSwap
function validateSwapForExecution(swapId, vault, amount, targetChain) external view onlyGuardian;
function getSwapForValidation(swapId) external view onlyGuardian;

// In CFI Token  
function validateSwapReception(swapId, vault, targetVault, amount, targetChain) external view onlyGuardian;
function recordChainSnapshot() external onlyGuardian;
function validateCrossChainConsistency(...) external onlyGuardian;
```

**Administrative Functions**:
```solidity
function updateChainSupport(chainId, supported) external onlyGuardianOrOwner;
function recordSwapCompletion(swapId, targetVault, amount) external onlyGuardian;
```

### 📊 **Guardian Workflow Example**

```javascript
class GuardianNode {
    async monitorSwaps() {
        // Monitor all supported chains
        for (const chain of supportedChains) {
            chain.swapContract.on('SwapInitiated', this.handleSwapInitiated.bind(this));
        }
    }
    
    async handleSwapInitiated(event) {
        try {
            // 1. Validate swap initiation
            const isValidSwap = await this.validateSwapInitiation(event);
            if (!isValidSwap) return;
            
            // 2. Validate token reception
            const tokenReception = await this.validateTokenReception(event);
            if (!tokenReception.isValid) return;
            
            // 3. Check target chain readiness
            const targetReady = await this.checkTargetChain(event);
            if (!targetReady) return;
            
            // 4. Execute swap
            await this.executeSwap(event, tokenReception.actualAmount);
            
            // 5. Record completion
            await this.recordCompletion(event);
            
        } catch (error) {
            await this.handleSwapError(event, error);
        }
    }
}
```

## Security Features

### 🔒 **Access Control Levels**

- **Owner/DAO**: Update Guardian address, transfer ownership
- **Guardian**: Execute swaps, validate cross-chain state, update chain support
- **Dev Signer**: Update technical parameters (in CFI contract)
- **Public**: Initiate swaps (with proper vault authorization)

### 🛡️ **Validation Layers**

1. **Registry Validation**: Only registered vaults can initiate swaps
2. **Whitelist Validation**: Only whitelisted swap contracts can receive tokens
3. **Signature Validation**: Users must sign all transfer authorizations
4. **Guardian Validation**: Multi-source cross-validation before execution
5. **Supply Validation**: Automatic supply management prevents inflation

### 🚨 **Emergency Controls**

```solidity
function emergencyRecover(token, amount) external onlyOwner; // Recover stuck tokens
function updateChainSupport(chainId, false) external onlyGuardianOrOwner; // Disable chains
function updateGuardian(newGuardian) external onlyOwner; // Change Guardian
```

## Events & Monitoring

### 📊 **Swap Events**
```solidity
event SwapInitiated(bytes32 indexed swapId, address indexed sourceVault, address indexed targetVault, uint256 amount, uint256 sourceChain, uint256 targetChain);
event SwapCompleted(bytes32 indexed swapId, address indexed targetVault, uint256 amount);
```

### 🔧 **Administrative Events**
```solidity
event GuardianUpdated(address indexed oldGuardian, address indexed newGuardian);
event ChainSupportUpdated(uint256 indexed chainId, bool supported);
event VaultRegistrationChecked(address indexed vault, bool isRegistered, address vaultOwner);
```

### 📈 **Integration Events**
Monitor these events from CFI Token contract for complete picture:
```solidity
event SwapTokensReceived(address indexed from, address indexed to, uint256 amount, uint256 indexed targetChain, bytes32 indexed swapId);
event SwapExecuted(address indexed from, address indexed to, uint256 amount, uint256 indexed sourceChain, bytes32 indexed swapId, uint256 treasuryUsed, uint256 minted);
event AutoBurnForCirculatingSupply(uint256 excessAmount, uint256 actualAmount, string operation);
event AutoMintForSwap(uint256 amount, address indexed recipient, string reason);
```

## Deployment Considerations

### 🔧 **Constructor Parameters**
```solidity
constructor(
    address cfiToken,           // CFI token contract address
    address vaultRegistry,      // Vault registry for validation
    address guardianAddress,    // Guardian Network address
    address initialOwner,       // DAO/Owner address
    uint256[] memory initialSupportedChains  // Supported chain IDs
)
```

### 📋 **Pre-Deployment Setup**

1. **Deploy CFI Token** on each chain with appropriate supply limits
2. **Deploy Vault Registry** and register legitimate vaults
3. **Deploy ChainFiSwap** with proper Guardian and registry addresses
4. **Whitelist Swap Contract** in vault listing registries
5. **Configure Guardian Network** to monitor all chains

### 🔄 **Post-Deployment Configuration**

1. **Test Swaps**: Perform test swaps between chains
2. **Guardian Validation**: Verify Guardian can monitor and execute
3. **Supply Monitoring**: Confirm auto-burn/mint working correctly
4. **Emergency Procedures**: Test emergency controls and recovery

## Best Practices

### 👨‍💼 **For Guardian Operators**
- Monitor all chains continuously
- Validate cross-chain state consistency regularly
- Implement robust error handling and retry logic
- Maintain secure key management for Guardian address
- Monitor swap completion times and failure rates

### 👤 **For Users**
- Ensure vault has sufficient CFI tokens before swapping
- Verify target vault address on destination chain
- Account for potential auto-burn reducing swapped amount
- Monitor swap completion via events or Guardian APIs

### 🏗️ **For Developers**
- Use proper error handling for all external calls
- Implement comprehensive event monitoring
- Test cross-chain scenarios thoroughly
- Plan for Guardian network upgrades and maintenance

## Conclusion

The **ChainFiSwap** contract provides a secure, efficient, and automated system for cross-chain CFI token transfers. By combining vault security, automatic supply management, and Guardian Network coordination, it enables seamless cross-chain operations while maintaining strict security and supply controls.

The system's **defense-in-depth** approach ensures that multiple validation layers protect against various attack vectors, while the **automatic supply management** prevents inflation and maintains healthy tokenomics across all supported chains. 