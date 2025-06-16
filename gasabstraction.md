# Gas Abstraction System Setup

This document outlines the structure, components, integration workflow, and UI modal interaction for the gas abstraction and membership management system within the Chain-Fi ecosystem.

## Overview

The gas abstraction system allows users to perform blockchain transactions without directly paying for gas fees, significantly reducing entry barriers and enhancing user adoption. Payment integration via Stripe and crypto ensures flexibility and ease of use for users across different preference types.

## Components

### 1. Centralized Payment Server

The centralized server interfaces directly with Stripe and blockchain smart contracts:

* Receives Stripe webhook events:

  * `invoice.payment_succeeded`
  * `customer.subscription.created`
  * `customer.subscription.updated`
  * `customer.subscription.deleted`
* Maintains dedicated treasury wallet for fee collections.
* Updates on-chain fee system contract accordingly.

### 2. Fee System Smart Contract

Responsible for membership status, expiration, tiers, and quotas:

```solidity
struct Member {
    bool isMember;
    uint8 tier;          // 1: Basic, 2: Medium, 3: Premium
    uint256 expiration;  // Unix timestamp
    uint256 quota;       // transactions, API calls, storage limits
}

mapping(address => Member) public members;

function updateMembership(
    address user, 
    bool isActive, 
    uint8 tier, 
    uint256 expirationTimestamp
) external onlyAuthorizedServer;

function isMemberActive(address user) public view returns (bool) {
    return (members[user].isMember && block.timestamp < members[user].expiration);
}
```

### 3. Vault Interaction Logic

Vault smart contracts query the membership status dynamically:

```solidity
require(feeSystem.isMemberActive(msg.sender), "Membership expired or inactive");
require(members[msg.sender].quota > 0, "Quota exhausted");

// After each transaction or ecosystem interaction
feeSystem.reduceQuota(msg.sender, usedAmount);
```

## UI Modal Integration

The UI modal component interacts directly with the centralized membership server, providing critical functionalities:

* **Subscription and Payment:** Options for Stripe and crypto payments.
* **Membership Status:** Active, expired, tier level, and expiration date clearly displayed.
* **Quota Usage:** Real-time quota tracking (transactions, API calls, storage).
* **Renewal and Upgrade:** Smooth tier management and proactive renewal prompts.
* **Membership History:** Accessible logs of previous payments, tiers, and expiration timestamps.

### UI Modal Workflow

* User subscribes via modal, selecting payment method and tier.
* Payment confirmed via centralized server, updating blockchain status.
* Membership and quota details dynamically shown and updated in the modal.

### Example UI Modal Display

* **Subscription View:**

  ```
  [Basic ($3.50/month)] [Medium ($7.50/month)] [Premium ($15/month)]
  ```

* **Membership Status:**

  ```
  Status: ✅ Active (Medium)
  Expires on: 2025-07-02
  Remaining Transactions: 145
  ```

* **Renew/Upgrade Prompt:**

  ```
  Your membership expires in 7 days.
  [Renew Now] [Upgrade Tier]
  ```

## Quota Rollover Policy

To provide a user-friendly and transparent experience, unused quota from the previous membership period is carried over into the next month during renewal.

### How It Works

When a subscription is renewed (via Stripe or crypto), the smart contract simply adds the base quota for the tier to the user's existing quota, up to the maximum cap. This avoids modifying or resetting the `isMember` boolean if it is still valid, and no explicit deletion of unused quota is needed.

When a subscription is renewed (via Stripe or crypto), the existing quota is added to the new monthly quota up to a defined maximum per tier:

```solidity
function renewSubscription(address user) external onlyAuthorizedServer {
    Member storage current = members[user];
    uint256 rollover = current.quota;

    uint256 newQuota = quotaByTier[current.tier];
    uint256 maxQuota = maxQuotaByTier[current.tier];

    current.quota = min(rollover + newQuota, maxQuota);
    current.expiration = block.timestamp + 30 days;
    current.isMember = true;
}
```

### Max Quotas Per Tier

\| Tier    | Monthly Quota | Max Rollover Cap | |## Decentralization Philosophy

While the Chain-Fi system leverages a centralized server for Stripe and crypto payment integration, it maintains decentralized principles in its core logic and access control:

* **Smart contracts enforce access control and quotas** on-chain using verifiable logic.
* **Crypto payments are fully supported**, allowing users to bypass traditional payment rails.
* **The system remains functional without Stripe**, ensuring censorship resistance.
* **Membership validation is dynamic and trustless**, with expiration and quota logic embedded in the smart contract.

This hybrid model balances decentralization with usability, offering accessibility to both crypto-native and mainstream users without compromising integrity or autonomy.

\---------|----------------|------------------| | Basic   | 50             | 100              | | Medium  | 200            | 400              | | Premium | Unlimited\*     | N/A              |

\* Note: Premium tier may optionally implement soft caps for operational safety.

This rollover system:

* Prevents quota hoarding while still rewarding consistent subscribers.
* Encourages continuous membership without penalizing underuse.
* Requires no extra gas cost—quota rollover is applied during the server-triggered renewal transaction.

## Workflow

### Subscription Activation

1. User subscribes via Stripe or Crypto.
2. Payment server receives payment confirmation from Stripe.
3. Payment server updates Fee System Smart Contract with:

   * `isMember = true`
   * Appropriate `tier`
   * `expiration = block.timestamp + 30 days`
   * Allocates initial quota based on the tier.

### Transaction Execution

* User initiates a transaction.
* Vault checks membership validity via dynamic evaluation.
* Quota is decremented automatically after each transaction.

### Subscription Expiry

* No explicit transaction required.
* Membership status is dynamically determined by timestamp comparison, reducing gas costs and complexity.

## Security

* Webhook events from Stripe verified with signature.
* Payment server transactions signed securely (HSM recommended).
* Smart contracts utilize minimal and efficient logic for dynamic membership validation.

## Benefits

* Seamless user onboarding.
* Gasless transactions for users.
* Flexible payment methods (Stripe & crypto).
* Real-time dynamic membership validation reduces operational costs.
* Scalable and secure infrastructure for mass adoption.

---

For implementation details or assistance, please contact the Chain-Fi development team.

### ✅ Updated ChainFi Membership Flow (Gas Abstraction Model)

This is how the membership and quota logic works with the updated `MemberRegistry.sol`:

---

### 1. **User Subscribes**
- User initiates a Stripe or crypto payment via your frontend UI.
- PaymentServer (your backend) receives webhook confirmation.

### 2. **PaymentServer Resolves Vault Address**
- It resolves which vault belongs to the user via off-chain logic (using VaultRegistry, database, or user metadata).
- Example:
  ```js
  const vault = await registry.getVaultOf(userAddress);
  ```

### 3. **Server Calls MemberRegistry**
- PaymentServer calls `updateMembership()` or `renewMembership()` on `MemberRegistry.sol`, passing:
  - `vault`: vault address
  - `tier`: 1, 2, or 3 (Basic, Medium, Premium)
  - `expiration`: `block.timestamp + 30 days`
- Example call:
  ```solidity
  memberRegistry.updateMembership(vault, true, 2, block.timestamp + 30 days);
  ```

---

### 4. **User Initiates On-Chain Vault Action**
- When the user interacts with their vault (e.g., `deposit`, `swap`, `withdraw`), the vault:
  1. Calls `isMemberActive(vault)` to check expiration.
  2. Checks `quota` via `getMember(vault)`.
  3. Proceeds with action only if quota is sufficient.
  4. Calls `reduceQuota(usedAmount)` on `MemberRegistry`.

---

### 🔐 Security Principle
Only the vault can call `reduceQuota()`, because `msg.sender` must equal the vault address in `members[msg.sender]`.

---

### 🔁 Example Vault Integration
```solidity
require(memberRegistry.isMemberActive(address(this)), "Membership expired");
MemberRegistry.Member memory m = memberRegistry.getMember(address(this));
require(m.quota >= 1, "Quota exceeded");

// Execute vault logic

memberRegistry.reduceQuota(1);
```

---

### ⚙️ Result
- PaymentServer handles registration & logic off-chain.
- Vault contracts enforce quota enforcement on-chain.
- No registry lookup inside the contract, clean separation of concerns.

Let me know if you want a mock vault or unit test setup next.

// SPDX-License-Identifier: MIT
pragma solidity 0.8.23;

/**
 * @title MemberRegistry 
 * @notice Tracks vault membership state, subscription tiers, and usage quotas.
 * @dev Quotas can only be reduced by a trusted Forwarder. Membership and tier configuration can only be updated by the PaymentServer.
 */
contract MemberRegistry {
    address public immutable paymentServer;
    address public trustedForwarder;

    struct Member {
        bool isMember;
        uint8 tier; // 1 = Basic, 2 = Medium, 3 = Premium, 4-6 locked initially
        uint256 expiration;
        uint256 quota;
    }

    mapping(address => Member) public members;
    mapping(uint8 => uint256) public quotaByTier;
    mapping(uint8 => uint256) public maxQuotaByTier;
    mapping(uint8 => bool) public isTierUnlocked;

    event MembershipUpdated(address indexed vault, uint8 tier, uint256 expiration, uint256 quota);
    event QuotaUsed(address indexed vault, uint256 used);
    event TierQuotaUpdated(uint8 tier, uint256 baseQuota, uint256 maxQuota);
    event TierUnlocked(uint8 tier, uint256 baseQuota, uint256 maxQuota);
    event TrustedForwarderUpdated(address forwarder);

    modifier onlyPaymentServer() {
        require(msg.sender == paymentServer, "Only PaymentServer");
        _;
    }

    modifier onlyForwarder() {
        require(msg.sender == trustedForwarder, "Only Forwarder");
        _;
    }

    constructor(address _paymentServer) {
        paymentServer = _paymentServer;

        quotaByTier[1] = 50;
        quotaByTier[2] = 200;
        quotaByTier[3] = type(uint256).max;

        maxQuotaByTier[1] = 100;
        maxQuotaByTier[2] = 400;
        maxQuotaByTier[3] = type(uint256).max;

        isTierUnlocked[1] = true;
        isTierUnlocked[2] = true;
        isTierUnlocked[3] = true;
        isTierUnlocked[4] = false;
        isTierUnlocked[5] = false;
        isTierUnlocked[6] = false;
    }

    function setTrustedForwarder(address _forwarder) external onlyPaymentServer {
        trustedForwarder = _forwarder;
        emit TrustedForwarderUpdated(_forwarder);
    }

    function updateMembership(
        address vault,
        bool isActive,
        uint8 tier,
        uint256 expiration
    ) external onlyPaymentServer {
        require(isTierUnlocked[tier], "Tier locked");
        Member storage m = members[vault];
        m.isMember = isActive;
        m.tier = tier;
        m.expiration = expiration;
        emit MembershipUpdated(vault, tier, expiration, m.quota);
    }

    function renewMembership(address vault) external onlyPaymentServer {
        Member storage m = members[vault];
        require(isTierUnlocked[m.tier], "Tier locked");
        uint256 baseQuota = quotaByTier[m.tier];
        uint256 maxQuota = maxQuotaByTier[m.tier];
        m.quota = _min(m.quota + baseQuota, maxQuota);
        m.expiration = block.timestamp + 30 days;
        m.isMember = true;
        emit MembershipUpdated(vault, m.tier, m.expiration, m.quota);
    }

    function reduceQuotaFor(address vault, uint256 used) external onlyForwarder {
        Member storage m = members[vault];
        require(m.quota >= used, "Quota exceeded");
        m.quota -= used;
        emit QuotaUsed(vault, used);
    }

    function updateTierQuota(uint8 tier, uint256 baseQuota, uint256 maxQuota) external onlyPaymentServer {
        quotaByTier[tier] = baseQuota;
        maxQuotaByTier[tier] = maxQuota;
        emit TierQuotaUpdated(tier, baseQuota, maxQuota);
    }

    function unlockTier(uint8 tier, uint256 baseQuota, uint256 maxQuota) external onlyPaymentServer {
        require(tier >= 4 && tier <= 6, "Invalid tier to unlock");
        require(!isTierUnlocked[tier], "Tier already unlocked");
        quotaByTier[tier] = baseQuota;
        maxQuotaByTier[tier] = maxQuota;
        isTierUnlocked[tier] = true;
        emit TierUnlocked(tier, baseQuota, maxQuota);
    }

    function isMemberActive(address vault) external view returns (bool) {
        Member memory m = members[vault];
        return m.isMember && block.timestamp < m.expiration;
    }

    function getMember(address vault) external view returns (Member memory) {
        return members[vault];
    }

    function _min(uint256 a, uint256 b) internal pure returns (uint256) {
        return a < b ? a : b;
    }
}


// SPDX-License-Identifier: MIT
pragma solidity 0.8.23;

import "./ECDSA.sol";
import "./ReentrancyGuard.sol";
import "./interfaces/IChainFiVaultRegistry.sol";
import "./interfaces/IChainFiForwarder.sol";
import "./interfaces/IMetaTransaction.sol";

contract ChainFiForwarder is IChainFiForwarder, ReentrancyGuard, IMetaTransaction {
    using ECDSA for bytes32;

    mapping(address => uint256) public _nonces;
    mapping(address => bool) public whitelistedContracts;
    mapping(address => uint256) public internalEthBalance;
    
    address public owner;
    IChainFiVaultRegistry public vaultRegistry;

    event MetaTransactionExecuted(
        address indexed from,
        address indexed to,
        uint256 gasUsed,
        bytes data,
        bool success
    );
    event EthDeposited(address indexed user, uint256 amount);
    event EthWithdrawn(address indexed user, uint256 amount);

    modifier onlyOwner() {
        require(msg.sender == owner, "Only owner");
        _;
    }

    constructor(address vaultRegistry_) {
        require(vaultRegistry_ != address(0), "Invalid vault registry");
        vaultRegistry = IChainFiVaultRegistry(vaultRegistry_);
    }

    function nonces(address user) external view override returns (uint256) {
        return _nonces[user];
    }

    function execute(
        ForwardRequest calldata req,
        bytes calldata signature
    ) external payable nonReentrant returns (bool, bytes memory) {
        _validateRequest(req);
        require(verify(req, signature), "Invalid signature");
        
        uint256 initialGas = gasleft();
        uint256 currentNonce = _nonces[req.from];

        bytes4 selector = bytes4(req.data[:4]);
        bytes memory encodedData = abi.encodePacked(
            selector,
            req.data[4:],
            abi.encode(
                req.from,
                address(0), // Always use ETH for gas
                currentNonce
            )
        );

        (bool success, bytes memory returndata) = req.to.call{
            gas: req.gas,
            value: req.value
        }(encodedData);

        _updateNonce(req.from);
        
        uint256 gasUsed = initialGas - gasleft() + 21000;
        _handleGasPayment(req.from, gasUsed);

        emit MetaTransactionExecuted(req.from, req.to, gasUsed, req.data, success);
        return (success, returndata);
    }

    function _validateRequest(ForwardRequest calldata req) internal view {
        require(vaultRegistry.getVaultOwner(req.to) != address(0), "Target not a registered vault");
        require(vaultRegistry.getUserVault(req.from) != address(0), "User has no registered vault");
        require(vaultRegistry.getUserVault(req.from) == req.to, "Not vault owner");
        require(gasleft() >= req.gas, "Insufficient gas");
        require(tx.gasprice <= req.maxGasPrice, "Gas price too high");

        uint256 estimatedGas = req.gas + 21000;
        uint256 maxGasCost = estimatedGas * tx.gasprice;
        require(internalEthBalance[req.from] >= maxGasCost, "Insufficient internal ETH balance");
    }

    function _handleGasPayment(address from, uint256 gasUsed) internal {
        uint256 totalCost = gasUsed * tx.gasprice;
        internalEthBalance[from] -= totalCost;
    }

    function _updateNonce(address from) internal {
        uint256 newNonce = uint256(keccak256(abi.encodePacked(
            block.timestamp,
            block.prevrandao,
            from
        )));
        _nonces[from] = newNonce;
    }

    function verify(
        ForwardRequest calldata req,
        bytes calldata signature
    ) public view returns (bool) {
        require(this.nonces(req.from) == req.nonce, "Invalid nonce");
        
        bytes32 digest = _hashTypedDataV4(
            keccak256(abi.encode(
                keccak256("ForwardRequest(address from,address to,uint256 value,uint256 gas,uint256 nonce,uint256 maxGasPrice,bytes data)"),
                req.from,
                req.to,
                req.value,
                req.gas,
                req.nonce,
                req.maxGasPrice,
                keccak256(req.data)
            ))
        );

        address signer = digest.recover(signature);
        return signer == req.from;
    }

    function _hashTypedDataV4(bytes32 structHash) internal view returns (bytes32) {
        return keccak256(abi.encodePacked("\x19\x01", _domainSeparatorV4(), structHash));
    }

    function _domainSeparatorV4() internal view returns (bytes32) {
        return keccak256(abi.encode(
            keccak256("EIP712Domain(string name,string version,uint256 chainId,address verifyingContract)"),
            keccak256("ChainFiForwarder"),
            keccak256("1"),
            block.chainid,
            address(this)
        ));
    }

    receive() external payable {
        internalEthBalance[msg.sender] += msg.value;
        emit EthDeposited(msg.sender, msg.value);
    }

    function depositEth() external payable {
        internalEthBalance[msg.sender] += msg.value;
        emit EthDeposited(msg.sender, msg.value);
    }

    function withdrawEth(uint256 amount) external nonReentrant {
        require(internalEthBalance[msg.sender] >= amount, "Insufficient balance");
        internalEthBalance[msg.sender] -= amount;
        (bool success,) = msg.sender.call{value: amount}("");
        require(success, "ETH transfer failed");
        emit EthWithdrawn(msg.sender, amount);
    }

    function getInternalEthBalance(address user) external view returns (uint256) {
        return internalEthBalance[user];
    }
} 