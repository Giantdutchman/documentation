# ChainFiForwarder: Gasless Meta-Transaction System

## Overview

ChainFiForwarder is an advanced gas abstraction system that enables users to execute vault operations without paying gas fees. It uses a meta-transaction pattern combined with payment server pre-validation to provide a truly gasless experience while maintaining security and preventing economic attacks.

## Table of Contents

- [Architecture Overview](#architecture-overview)
- [User Experience Magic](#user-experience-magic)
- [Meta-Transaction Pattern](#meta-transaction-pattern)
- [Payment Server Architecture](#payment-server-architecture)
- [Complete Flow Explanation](#complete-flow-explanation)
- [Function Call Encoding](#function-call-encoding)
- [Security Features](#security-features)
- [Code Examples](#code-examples)
- [Integration Guide](#integration-guide)
- [Security Considerations](#security-considerations)

## Architecture Overview

```
User → Payment Server → ChainFiForwarder → ChainFiVault
  ↓         ↓               ↓               ↓
Creates   Validates      Executes        Processes
Request   Off-chain      On-chain        Function
```

### Key Components

1. **ChainFiForwarder**: On-chain meta-transaction executor
2. **Payment Server**: Off-chain validation and access control
3. **ChainFiVault**: Target contract that receives forwarded calls
4. **Meta-Transaction Pattern**: Enables gasless execution

## User Experience Magic

### The Invisible Wrapper System

**From the user's perspective**: They simply call vault functions like `withdrawERC20()` or `transferNFT()` as if calling them directly.

**Behind the scenes**: The frontend performs sophisticated function call wrapping to enable gasless transactions.

```javascript
// What the user sees and thinks they're doing:
await vault.withdrawERC20(tokenAddress, amount);
// ↑ "I'm withdrawing tokens from my vault"

// What the frontend actually does (invisible to user):
const vaultCalldata = vault.interface.encodeFunctionData("withdrawERC20", [...]);
const forwardRequest = { from: user, to: vault, data: vaultCalldata };
const signature = await user._signTypedData(forwardRequest);
await paymentServer.execute(forwardRequest, signature);
// ↑ Frontend wraps the call for gasless execution
```

### Frontend Function Wrapping Flow

1. **User Initiates Action**: "Withdraw 100 CFI tokens"
2. **Frontend Intercepts**: Instead of calling vault directly, encodes the function call
3. **Meta-Transaction Creation**: Wraps the vault call in a ForwardRequest
4. **User Signs Once**: Signs the meta-transaction (still thinks they're just withdrawing)
5. **Gasless Execution**: Payment server validates and forwarder executes
6. **Vault Processes**: The actual `withdrawERC20` function runs on the vault

### The Magic: Seamless UX

```javascript
// Traditional approach (user pays gas):
const tx = await vault.withdrawERC20(token, amount, { gasPrice: "20000000000" });
await tx.wait(); // User pays ~$5-50 in gas

// ChainFi approach (gasless via forwarder):
await chainfi.vault.withdrawERC20(token, amount); // User pays $0 in gas
// ↑ Same interface, but completely gasless behind the scenes
```

**The user experience is identical, but the underlying execution is completely different!**

## Meta-Transaction Pattern

The forwarder uses EIP-712 typed data structures to enable users to sign transaction requests off-chain, which are then executed by the payment server on-chain.

### ForwardRequest Structure

```solidity
struct ForwardRequest {
    address from;        // User address (vault owner)
    address to;          // Target contract (user's vault)
    uint256 value;       // ETH value (usually 0)
    uint256 gas;         // Gas limit for execution
    uint256 nonce;       // User's current nonce
    uint256 maxGasPrice; // Maximum acceptable gas price
    bytes data;          // Encoded function call data
}
```

**The `data` field is crucial** - it contains the complete encoded function call that should be executed on the target vault.

## Payment Server Architecture

### Off-Chain Validation Benefits

The payment server provides multiple layers of validation before any transaction reaches the blockchain:

1. **Membership Validation**: Ensures user has active membership
2. **Quota Management**: Checks remaining transaction quota
3. **Rate Limiting**: Prevents spam attacks per IP/user
4. **Geographic Blocking**: Blocks requests from restricted regions
5. **Economic Protection**: Validates sufficient funds before execution

### Architecture Benefits

- **No Gas Drainage**: Invalid requests never reach blockchain
- **Efficient Validation**: Complex checks done off-chain
- **Attack Prevention**: Multiple security layers before execution
- **Cost Effective**: Only valid transactions consume gas

## Complete Flow Explanation

### Step 1: User Prepares Vault Function Call

The user (or frontend) creates the exact function call they want to execute:

```solidity
// Example: Withdrawing ERC20 tokens
uint256 vaultNonce = vault.getNonce();
bytes32 messageHash = keccak256(abi.encodePacked(
    tokenAddress,
    userAddress,
    withdrawAmount,
    vaultNonce
));
bytes memory vaultSignature = signMessage(messageHash, authPrivateKey);

bytes memory vaultCalldata = abi.encodeWithSelector(
    ChainFiVault.withdrawERC20.selector,
    tokenAddress,
    withdrawAmount,
    vaultSignature
);
```

### Step 2: Create ForwardRequest

Package the function call into a forward request:

```solidity
ForwardRequest memory request = ForwardRequest({
    from: userAddress,
    to: userVaultAddress,
    value: 0,
    gas: 300000,
    nonce: forwarder.nonces(userAddress),
    maxGasPrice: 20 gwei,
    data: vaultCalldata  // ← The encoded vault function call
});
```

### Step 3: User Signs Request

User signs the request using EIP-712:

```solidity
bytes32 digest = _hashTypedDataV4(structHash);
bytes memory signature = signMessage(digest, userPrivateKey);
```

### Step 4: Submit to Payment Server

User submits the signed request to the payment server API:

```javascript
const response = await fetch('/api/execute', {
    method: 'POST',
    body: JSON.stringify({
        request: forwardRequest,
        signature: userSignature
    })
});
```

### Step 5: Payment Server Validation

Payment server validates the request off-chain:

```javascript
// Pseudo-code for payment server validation
async function validateRequest(request, signature) {
    // 1. Verify user signature
    const isValidSignature = verifySignature(request, signature);
    
    // 2. Check membership status
    const hasMembership = await checkMembership(request.from);
    
    // 3. Verify quota availability
    const hasQuota = await checkQuota(userVault);
    
    // 4. Validate vault funds
    const hasFunds = await checkVaultBalance(request.to, request.data);
    
    // 5. Rate limiting check
    const withinRateLimit = checkRateLimit(userIP, request.from);
    
    return isValidSignature && hasMembership && hasQuota && hasFunds && withinRateLimit;
}
```

### Step 6: Forwarder Execution

If validation passes, payment server calls the forwarder:

```solidity
function execute(
    ForwardRequest calldata req,
    bytes calldata signature
) external onlyPaymentServer returns (bool, bytes memory) {
    // Verify user signature
    require(verify(req, signature), "Invalid signature");
    
    // Extract function selector
    bytes4 selector = bytes4(req.data[:4]);
    
    // Append meta-transaction context
    bytes memory encodedData = abi.encodePacked(
        selector,        // Original function selector
        req.data[4:],   // Original parameters
        abi.encode(
            req.from,    // Real sender
            address(0),  // Gas token (ETH)
            nonce        // Forwarder nonce
        )
    );

    // Execute the call on the target vault
    // This directly calls the actual vault function (withdrawERC20, transferNFT, etc.)
    (bool success, bytes memory returndata) = req.to.call{
        gas: req.gas,
        value: req.value
    }(encodedData);
    
    // Handle gas payment and quota reduction
    _handleGasPayment(gasUsed);
    _reduceUserQuota(req.from);
    
    return (success, returndata);
}
```

### Step 7: Vault Processes Function Call

The vault receives the call and extracts the real sender:

```solidity
function _msgSender() internal view returns (address) {
    if (msg.sender == trustedForwarder) {
        // Extract real sender from appended calldata
        // Last 96 bytes: [realSender][gasToken][forwarderNonce]
        return abi.decode(msg.data[msg.data.length-96:msg.data.length-64], (address));
    }
    return msg.sender;
}

function withdrawERC20(address token, uint256 amount, bytes memory signature) 
    external onlyOwnerViaForwarder {
    // _msgSender() returns the original user, not the forwarder
    address realUser = _msgSender();
    
    // Process the withdrawal...
}
```

## Key Understanding Summary

### What Actually Happens (Your Correct Understanding!)

✅ **Frontend Function Wrapping**: Most of the magic happens on the frontend  
✅ **User Perspective**: User thinks they're calling vault functions directly  
✅ **Data Collection**: Frontend collects the needed function call data  
✅ **Forwarder Validation**: Payment server validates, then forwarder executes  
✅ **Direct Vault Calls**: Forwarder calls the actual vault functions (`withdrawERC20`, `transferNFT`, etc.)  

### What Does NOT Happen

❌ **No Generic Execute Function on Vault**: There's no `execute()` function on the vault  
❌ **No Double Wrapping**: Forwarder doesn't call another wrapper that calls vault functions  
❌ **No Complex Routing**: Direct call from forwarder to specific vault function  

### The Flow You Understood Perfectly:

```
User Action → Frontend Wrapping → Payment Server → Forwarder → Actual Vault Function
    ↓              ↓                  ↓             ↓              ↓
 "Withdraw"    Encode call        Validate      Execute      withdrawERC20()
```

## Function Call Encoding

The forwarder can execute any vault function by encoding it in the `data` field:

### ERC20 Token Operations

```solidity
// Withdraw tokens to owner
bytes memory withdrawCall = abi.encodeWithSelector(
    ChainFiVault.withdrawERC20.selector,
    tokenAddress,
    amount,
    authSignature
);

// Transfer tokens to another address
bytes memory transferCall = abi.encodeWithSelector(
    ChainFiVault.transferERC20.selector,
    tokenAddress,
    recipientAddress,
    amount,
    authSignature
);
```

### NFT Operations

```solidity
// Transfer ERC721 NFT
bytes memory nftCall = abi.encodeWithSelector(
    ChainFiVault.transferERC721.selector,
    nftContract,
    recipientAddress,
    tokenId,
    authSignature
);

// Transfer ERC1155 tokens
bytes memory multiTokenCall = abi.encodeWithSelector(
    ChainFiVault.transferERC1155.selector,
    nftContract,
    recipientAddress,
    tokenId,
    amount,
    authSignature
);
```

### Ether Operations

```solidity
// Withdraw ETH to owner
bytes memory ethWithdrawCall = abi.encodeWithSelector(
    ChainFiVault.withdrawEther.selector,
    amount,
    authSignature
);

// Transfer ETH to another address
bytes memory ethTransferCall = abi.encodeWithSelector(
    ChainFiVault.transferEther.selector,
    recipientAddress,
    amount,
    authSignature
);
```

### Vault Management Operations

```solidity
// Update vault owner
bytes memory updateOwnerCall = abi.encodeWithSelector(
    ChainFiVault.modifyOwnerAddress.selector,
    newOwnerAddress,
    authSignature
);

// Update fallback address
bytes memory updateFallbackCall = abi.encodeWithSelector(
    ChainFiVault.modifyFallbackAddress.selector,
    newFallbackAddress,
    authSignature
);
```

## Security Features

### ChainGuard Integration

The forwarder integrates with ChainGuard for enterprise-grade security:

```solidity
// Multi-signature admin functions
function updatePaymentServer(address newServer, bytes[] memory signatures) external;
function emergencyPauseWithTeam(bool pause, bytes[] memory signatures) external;
function emergencyWithdraw(uint256 amount, address dest, bytes[] memory signatures) external;
```

### Access Control

- **Payment Server Only**: Only the designated payment server can call `execute()`
- **Emergency Pause**: Can be paused by ChainGuard team in emergencies
- **Signature Verification**: All requests must be properly signed by users

### Economic Protection

```solidity
// Project ETH pays for gas, not users
function _handleGasPayment(uint256 gasUsed) internal {
    uint256 gasCost = gasUsed * tx.gasprice;
    if (projectEthBalance >= gasCost) {
        projectEthBalance -= gasCost;
        emit GasPaidByProject(user, gasCost);
    }
}
```

## Code Examples

### Frontend Integration

```javascript
// 1. Create vault function call
const vaultCalldata = vault.interface.encodeFunctionData("withdrawERC20", [
    tokenAddress,
    amount,
    authSignature
]);

// 2. Create forward request
const forwardRequest = {
    from: userAddress,
    to: vaultAddress,
    value: 0,
    gas: 300000,
    nonce: await forwarder.nonces(userAddress),
    maxGasPrice: ethers.utils.parseUnits("20", "gwei"),
    data: vaultCalldata
};

// 3. Sign the request
const signature = await user._signTypedData(domain, types, forwardRequest);

// 4. Submit to payment server
const response = await submitToPaymentServer(forwardRequest, signature);
```

### Payment Server Implementation

```javascript
app.post('/api/execute', async (req, res) => {
    const { request, signature } = req.body;
    
    try {
        // Validate request off-chain
        const isValid = await validateRequest(request, signature);
        if (!isValid) {
            return res.status(400).json({ error: 'Invalid request' });
        }
        
        // Execute on-chain
        const tx = await forwarder.execute(request, signature);
        const receipt = await tx.wait();
        
        res.json({ success: true, txHash: receipt.transactionHash });
    } catch (error) {
        res.status(500).json({ error: error.message });
    }
});
```

### Smart Contract Integration

```solidity
contract MyVaultManager {
    IChainFiForwarder public forwarder;
    
    function executeVaultOperation(
        address vault,
        bytes memory functionCall,
        uint256 gas
    ) external {
        ForwardRequest memory req = ForwardRequest({
            from: msg.sender,
            to: vault,
            value: 0,
            gas: gas,
            nonce: forwarder.nonces(msg.sender),
            maxGasPrice: 20 gwei,
            data: functionCall
        });
        
        // This would normally go through payment server
        // Shown here for illustration only
    }
}
```

## Integration Guide

### For Frontend Developers

1. **Install Dependencies**:
   ```bash
   npm install ethers @chainfi/sdk
   ```

2. **Initialize Forwarder**:
   ```javascript
   import { ChainFiForwarder } from '@chainfi/sdk';
   
   const forwarder = new ChainFiForwarder(forwarderAddress, provider);
   ```

3. **Create and Sign Requests**:
   ```javascript
   const request = await forwarder.createRequest(vault, functionCall);
   const signature = await user.signTypedData(request);
   ```

4. **Submit to Payment Server**:
   ```javascript
   const result = await forwarder.execute(request, signature);
   ```

### For Backend Developers

1. **Implement Payment Server**:
   - Set up validation endpoints
   - Implement membership checking
   - Add rate limiting and security
   - Connect to blockchain for execution

2. **Handle Validation**:
   ```javascript
   const validator = new PaymentServerValidator({
       membershipContract: memberRegistryAddress,
       vaultRegistry: vaultRegistryAddress,
       forwarder: forwarderAddress
   });
   ```

### For Smart Contract Developers

1. **Implement Forwarder Support**:
   ```solidity
   contract MyContract {
       address public trustedForwarder;
       
       modifier onlyViaForwarder() {
           require(msg.sender == trustedForwarder, "Must use forwarder");
           _;
       }
       
       function _msgSender() internal view returns (address) {
           if (msg.sender == trustedForwarder) {
               return abi.decode(msg.data[msg.data.length-96:msg.data.length-64], (address));
           }
           return msg.sender;
       }
   }
   ```

## Security Considerations

### Signature Security

- **Nonce Protection**: Prevents replay attacks
- **EIP-712**: Uses typed data signing for security
- **Domain Separation**: Prevents cross-contract signature reuse

### Economic Security

- **Payment Server Validation**: Prevents invalid transactions
- **Quota System**: Limits usage per user
- **Gas Payment**: Project pays gas, eliminating user cost

### Access Control

```solidity
modifier onlyPaymentServer() {
    require(msg.sender == paymentServer, "Only payment server");
    _;
}

modifier whenNotEmergencyPaused() {
    require(!emergencyPaused, "Contract is emergency paused");
    _;
}
```

### Emergency Controls

- **Emergency Pause**: Can halt all operations
- **ChainGuard MultiSig**: Requires team approval for critical changes
- **Emergency Withdrawal**: Can recover funds if needed

## Benefits Summary

1. **Truly Gasless**: Users pay no gas fees
2. **Universal**: Can forward any function call
3. **Secure**: Multiple validation layers
4. **Efficient**: Off-chain validation reduces costs
5. **Scalable**: Payment server handles high load
6. **Attack Resistant**: Economic attacks prevented off-chain

## Deployment Addresses

- **ChainFiForwarder**: `[TO_BE_DEPLOYED]`
- **Payment Server**: `[CONFIGURED_ENDPOINT]`
- **ChainGuard Core**: `[DEPLOYED_ADDRESS]`

## API Documentation

### Core Functions

```solidity
// Execute a meta-transaction
function execute(ForwardRequest calldata req, bytes calldata signature) 
    external returns (bool, bytes memory);

// Verify a request signature
function verify(ForwardRequest calldata req, bytes calldata signature) 
    external view returns (bool);

// Get user's current nonce
function nonces(address user) external view returns (uint256);
```

### Admin Functions

```solidity
// Update payment server (requires ChainGuard signatures)
function updatePaymentServer(address newServer, bytes[] memory signatures) external;

// Emergency pause (requires ChainGuard signatures)
function emergencyPauseWithTeam(bool pause, bytes[] memory signatures) external;
```

## Troubleshooting

### Common Issues

1. **"Invalid signature"**: Check nonce and EIP-712 domain
2. **"Only payment server"**: Must call through payment server
3. **"Quota exceeded"**: User has no remaining quota
4. **"User does not own a vault"**: Vault not found in registry

### Debug Steps

1. Verify user signature with `verify()` function
2. Check user nonce with `nonces()` function
3. Validate membership status
4. Confirm vault ownership in registry

---

For more information, see the main [ChainFi Documentation](./README.md) or contact the development team. 