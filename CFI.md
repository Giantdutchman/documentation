# CFI Token (Chain-Fi Token) - README

## 🔒 Securing the Future of Decentralized Finance

CFI is the native utility token powering the Chain-Fi ecosystem - a revolutionary blockchain security platform that combines advanced cryptographic methods and cutting-edge decentralized authentication to deliver unmatched asset protection for digital finance.

---

## 📊 Token Overview

| **Attribute** | **Details** |
|---------------|-------------|
| **Token Name** | Chain-Fi Token |
| **Symbol** | CFI |
| **Standard** | ERC-20 Compatible |
| **Total Supply** | 1,000,000,000 CFI (1 Billion) |
| **Decimals** | 18 |
| **Initial Price** | TBD (Private Sale) |
| **Regulatory Status** | Non-Security Utility Token |
| **Primary Network** | Base (Trading Hub) |
| **Multi-Chain Support** | Ethereum, Arbitrum, Optimism, Polygon |

---

## 🌐 Multi-Chain Native Architecture

### Revolutionary Token Design
- **Native Deployment**: CFI is natively deployed on each supported blockchain
- **No Bridging Required**: Direct native token usage across all networks
- **Unified Supply**: 1B CFI total supply managed across all chain deployments
- **Cross-Chain Utility**: Full functionality available on each supported chain
- **Guardian Network Coordination**: Secure cross-chain balance verification via ECDSA signatures

### Supported Networks
- **Base Network** 🎯 *Primary trading hub with concentrated liquidity*
- **Ethereum** 🔷 *Institutional and staking operations*
- **Arbitrum** ⚡ *Gaming and NFT ecosystems*
- **Optimism** 🔴 *DeFi protocol integrations*
- **Polygon** 🟣 *High-frequency micro-transactions*

---

## 💰 Token Distribution & Allocation

### Effective Circulating Supply: 550M CFI (55%)

| **Category** | **Amount** | **Percentage** | **Description** |
|-------------|------------|----------------|------------------|
| **Development & Ecosystem** | 300M CFI | 30% | Core development, rewards, grants |
| **Team & Advisors** | 150M CFI | 15% | Core team and strategic advisors |
| **Private Sale** | 100M CFI | 10% | Strategic and private investors |
| **Treasury & Expansion** | 300M CFI | 30% | 🔒 *Non-circulating treasury* |
| **Guardian Network Reserve** | 150M CFI | 15% | 🔒 *Non-circulating reserve* |

### 🔥 Critical Supply Distinction
- **Real Circulating Supply**: 550M CFI (not 1B CFI)
- **Treasury-Controlled**: 450M CFI held in non-circulating reserves
- **Governance Required**: Community voting needed for treasury/guardian token distribution
- **Enhanced Scarcity**: 45% supply reduction from day one

---

## 🛠 Core Utility Functions

### 1. **Membership Access & Tiered Services**
- **Basic Tier**: $2.99/month - Essential security features
- **Normal Tier**: $9.99/month - Advanced vault management
- **Premium Tier**: $49.99/month - Enterprise-grade security

### 2. **Platform Operations & Fees**
- **Gasless Architecture**: Platform absorbs all blockchain fees
- **Quota Management**: Tier-based transaction and API quotas
- **Fee Extensions**: CFI payments for additional resources
- **Emergency Recovery**: Secure asset recovery services

### 3. **Ecosystem Payment Token**
- **NFT Marketplace**: Universal payment for digital assets
- **Gaming Platforms**: In-game purchases and virtual goods
- **DEX Trading**: Base Network exclusive trading
- **Subscription Services**: Recurring payments for dApps
- **Security Services**: Bug bounties and audit payments

### 4. **Governance Participation**
- **Protocol Parameters**: Security thresholds and fee structures
- **Multi-Chain Voting**: Holdings across all chains count
- **Guardian Network**: 1.5x voting weight for network participants
- **Non-Corporate Matters**: Utility token classification compliant

---

## 🔥 Revolutionary Deflationary Framework

### Dynamic Multi-Stream Burning Mechanism

CFI implements a sophisticated deflationary model with **dynamic burning rates** that start at 25% and gradually decline to 10% as supply decreases, creating optimal deflationary pressure while maintaining ecosystem sustainability.

#### **Dynamic Burn Rate Schedule**
```
Supply Remaining → Burn Rate
1B CFI (100%) → 25% (Initial)
750M CFI (75%) → 22%
500M CFI (50%) → 18%
300M CFI (30%) → 15%
200M CFI (20%) → 12%
100M CFI (10%) → 10% (Minimum)
```

#### **Burn Sources**
1. **Fee-Based Burns**: 25%→10% of all platform fees
2. **Membership Burns**: 25%→10% of subscription payments
3. **Ecosystem Payment Burns**: 25%→10% of connected dApp payments
4. **Milestone Burns**: Achievement-based treasury burns
5. **Scheduled Treasury Burns**: Quarterly governance-approved burns

### 🎯 Scheduled Treasury Burns (Post-Governance)
- **Quarterly Burns**: 2.5% of eligible treasury (7.5M CFI initially)
- **Community Governance**: Required for all treasury burns
- **Maximum Per Quarter**: 5% of eligible treasury (15M CFI)
- **Protected Reserves**: Guardian Network tokens never burned
- **Transparency**: All burns publicly tracked and reported

---

## 🏛 Regulatory Compliance

### Non-Security Utility Token Classification
- **UK FSMA Compliant**: Non-security utility token status
- **EU MiCA Aligned**: Utility token (non-ART, non-EMT)
- **FATF Standards**: AML/KYC integration with transaction monitoring
- **No Investment Features**: No equity rights, profit-sharing, or redemption guarantees
- **Pure Utility Focus**: Platform access, fee payments, and governance only

### Compliance Framework
```javascript
const ComplianceFramework = {
  UK_FSMA: {
    classification: "Non-security utility token",
    requirements: ["Risk disclosure", "No investment promotion", "Utility focus"],
    status: "Compliant"
  },
  EU_MiCA: {
    classification: "Utility token (non-ART, non-EMT)",
    requirements: ["Technical whitepaper", "Issuer disclosure", "Market integrity"],
    status: "Aligned"
  },
  FATF_Standards: {
    requirements: ["Travel Rule compliance", "AML/KYC integration", "Transaction monitoring"],
    implementation: "CipherTrace Chainalysis integration",
    status: "Implemented"
  }
};
```

---

## 🔧 Technical Architecture

### Smart Contract Features
- **ERC-20 Compatible**: Standard token interface
- **Multi-Chain Native**: Deployed natively on each network
- **Guardian Network Integration**: ECDSA signature validation
- **CREATE2 Vaults**: Deterministic cross-chain addresses
- **Dynamic Fee Management**: Tier-based fee calculations
- **Burn Mechanism**: Multiple deflationary streams

### Security Features
- **Hardware Security Module**: TPM and HSM integration
- **ChainGuard 2FA**: Decentralized authentication
- **Guardian Network**: Cross-chain coordination and validation
- **Emergency Recovery**: Secure asset recovery mechanisms
- **Audit Trail**: Comprehensive transaction logging

---

## 📜 CFI Token Contract (First Draft)

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.19;

import "@openzeppelin/contracts/token/ERC20/ERC20.sol";
import "@openzeppelin/contracts/access/Ownable.sol";
import "@openzeppelin/contracts/security/ReentrancyGuard.sol";
import "@openzeppelin/contracts/utils/cryptography/ECDSA.sol";

/**
 * @title CFI Token (Chain-Fi Token)
 * @dev Multi-chain native utility token with Guardian Network coordination
 * @notice Non-security utility token for Chain-Fi ecosystem
 */
contract CFIToken is ERC20, Ownable, ReentrancyGuard {
    using ECDSA for bytes32;
    
    // ========== CONSTANTS ==========
    uint256 public constant MAX_SUPPLY = 1_000_000_000 * 10**18; // 1 billion CFI
    uint256 public constant INITIAL_BURN_RATE = 25; // 25% initial burn rate
    uint256 public constant MINIMUM_BURN_RATE = 10; // 10% minimum burn rate
    
    // Guardian Network address (hardcoded for security)
    address public constant GUARDIAN_ADDRESS = 0x742d35Cc6634C0532925a3b8D4C9db96590c6C89;
    
    // ========== STATE VARIABLES ==========
    uint256 private _totalBurned;
    uint256 private _currentBurnRate = INITIAL_BURN_RATE;
    
    // Internal vault for cross-chain liquidity
    uint256 public internalVaultBalance;
    
    // Membership tiers and fees
    enum MembershipTier { BASIC, NORMAL, PREMIUM }
    mapping(address => MembershipTier) public userTiers;
    mapping(address => uint256) public membershipExpiry;
    
    // Fee structure
    mapping(MembershipTier => uint256) public membershipFees;
    
    // Cross-chain coordination
    mapping(bytes32 => bool) public processedSwaps;
    mapping(uint256 => uint256) public chainInternalVaults; // chainId => balance
    
    // Governance
    mapping(address => uint256) public governanceWeight;
    bool public governanceActive = false;
    
    // ========== EVENTS ==========
    event TokensBurned(uint256 amount, string reason);
    event BurnRateAdjusted(uint256 newRate, uint256 totalSupply);
    event MembershipPurchased(address indexed user, MembershipTier tier, uint256 amount);
    event CrossChainSwapInitiated(bytes32 indexed swapId, address indexed user, uint256 amount, uint256 targetChain);
    event CrossChainSwapCompleted(bytes32 indexed swapId, address indexed user, uint256 amount);
    event GuardianMint(address indexed recipient, uint256 amount, bytes32 swapId);
    event InternalVaultTransfer(address indexed recipient, uint256 amount, bytes32 swapId);
    
    // ========== CONSTRUCTOR ==========
    constructor() ERC20("Chain-Fi Token", "CFI") {
        // Set membership fees (in CFI, adjusted for 18 decimals)
        membershipFees[MembershipTier.BASIC] = 299 * 10**16; // ~$2.99 worth
        membershipFees[MembershipTier.NORMAL] = 999 * 10**16; // ~$9.99 worth  
        membershipFees[MembershipTier.PREMIUM] = 4999 * 10**16; // ~$49.99 worth
        
        // Initial token distribution (will be adjusted based on final allocation)
        _mint(msg.sender, MAX_SUPPLY);
        
        // Initialize internal vault with 5% of supply for cross-chain liquidity
        internalVaultBalance = MAX_SUPPLY * 5 / 100;
    }
    
    // ========== MODIFIERS ==========
    modifier onlyGuardian() {
        require(msg.sender == GUARDIAN_ADDRESS, "Only Guardian Network");
        _;
    }
    
    modifier validSignature(bytes32 messageHash, bytes memory signature) {
        address recoveredAddress = messageHash.recover(signature);
        require(recoveredAddress == GUARDIAN_ADDRESS, "Invalid guardian signature");
        _;
    }
    
    // ========== MEMBERSHIP FUNCTIONS ==========
    
    /**
     * @notice Purchase membership tier with CFI tokens
     * @param tier The membership tier to purchase
     */
    function purchaseMembership(MembershipTier tier) external nonReentrant {
        uint256 fee = membershipFees[tier];
        require(balanceOf(msg.sender) >= fee, "Insufficient CFI balance");
        
        // Calculate burn amount based on dynamic rate
        uint256 burnAmount = (fee * _currentBurnRate) / 100;
        uint256 treasuryAmount = fee - burnAmount;
        
        // Transfer fee and burn portion
        _transfer(msg.sender, address(this), fee);
        _burn(address(this), burnAmount);
        _updateTotalBurned(burnAmount);
        
        // Update user membership
        userTiers[msg.sender] = tier;
        membershipExpiry[msg.sender] = block.timestamp + 30 days;
        
        emit MembershipPurchased(msg.sender, tier, fee);
        emit TokensBurned(burnAmount, "Membership purchase");
    }
    
    /**
     * @notice Get user's current membership tier
     */
    function getUserMembershipTier(address user) external view returns (MembershipTier, uint256) {
        if (membershipExpiry[user] < block.timestamp) {
            return (MembershipTier.BASIC, 0); // Expired membership defaults to BASIC
        }
        return (userTiers[user], membershipExpiry[user]);
    }
    
    // ========== CROSS-CHAIN FUNCTIONS ==========
    
    /**
     * @notice Initiate cross-chain swap
     * @param amount Amount to swap
     * @param targetChain Target chain ID
     */
    function initiateCrossChainSwap(uint256 amount, uint256 targetChain) external nonReentrant {
        require(amount > 0, "Amount must be positive");
        require(balanceOf(msg.sender) >= amount, "Insufficient balance");
        require(targetChain != block.chainid, "Cannot swap to same chain");
        
        // Generate unique swap ID
        bytes32 swapId = keccak256(abi.encodePacked(
            msg.sender, amount, targetChain, block.timestamp, block.number
        ));
        
        // Burn tokens on source chain
        _burn(msg.sender, amount);
        _updateTotalBurned(amount);
        
        emit CrossChainSwapInitiated(swapId, msg.sender, amount, targetChain);
        emit TokensBurned(amount, "Cross-chain swap");
    }
    
    /**
     * @notice Complete cross-chain swap from internal vault
     * @param swapId Unique swap identifier
     * @param recipient Token recipient
     * @param amount Amount to transfer
     * @param signature Guardian signature
     */
    function completeSwapFromInternalVault(
        bytes32 swapId,
        address recipient,
        uint256 amount,
        bytes memory signature
    ) external validSignature(
        keccak256(abi.encodePacked(swapId, recipient, amount, "VAULT_TRANSFER")),
        signature
    ) nonReentrant {
        require(!processedSwaps[swapId], "Swap already processed");
        require(internalVaultBalance >= amount, "Insufficient vault balance");
        
        processedSwaps[swapId] = true;
        internalVaultBalance -= amount;
        
        _mint(recipient, amount);
        
        emit InternalVaultTransfer(recipient, amount, swapId);
        emit CrossChainSwapCompleted(swapId, recipient, amount);
    }
    
    /**
     * @notice Guardian mint for cross-chain swaps when vault insufficient
     * @param swapId Unique swap identifier  
     * @param recipient Token recipient
     * @param amount Amount to mint
     * @param signature Guardian signature
     */
    function guardianMintTo(
        bytes32 swapId,
        address recipient, 
        uint256 amount,
        bytes memory signature
    ) external validSignature(
        keccak256(abi.encodePacked(swapId, recipient, amount, "MINT_TO")),
        signature
    ) nonReentrant {
        require(!processedSwaps[swapId], "Swap already processed");
        require(totalSupply() + amount <= MAX_SUPPLY, "Exceeds max supply");
        
        processedSwaps[swapId] = true;
        _mint(recipient, amount);
        
        emit GuardianMint(recipient, amount, swapId);
        emit CrossChainSwapCompleted(swapId, recipient, amount);
    }
    
    // ========== BURN MECHANISM ==========
    
    /**
     * @notice Calculate dynamic burn rate based on current supply
     */
    function calculateDynamicBurnRate() public view returns (uint256) {
        uint256 supplyRatio = (totalSupply() * 100) / MAX_SUPPLY;
        
        if (supplyRatio >= 75) return 25;      // 750M+ CFI remaining
        if (supplyRatio >= 50) return 22;      // 500M+ CFI remaining  
        if (supplyRatio >= 30) return 18;      // 300M+ CFI remaining
        if (supplyRatio >= 20) return 15;      // 200M+ CFI remaining
        if (supplyRatio >= 10) return 12;      // 100M+ CFI remaining
        return MINIMUM_BURN_RATE;              // <100M CFI remaining
    }
    
    /**
     * @notice Update burn rate and total burned tracking
     */
    function _updateTotalBurned(uint256 amount) internal {
        _totalBurned += amount;
        
        // Update dynamic burn rate
        uint256 newBurnRate = calculateDynamicBurnRate();
        if (newBurnRate != _currentBurnRate) {
            _currentBurnRate = newBurnRate;
            emit BurnRateAdjusted(newBurnRate, totalSupply());
        }
    }
    
    /**
     * @notice Manual burn function for treasury/milestone burns
     */
    function treasuryBurn(uint256 amount) external onlyOwner {
        require(amount > 0, "Amount must be positive");
        require(balanceOf(address(this)) >= amount, "Insufficient treasury balance");
        
        _burn(address(this), amount);
        _updateTotalBurned(amount);
        
        emit TokensBurned(amount, "Treasury burn");
    }
    
    // ========== GOVERNANCE FUNCTIONS ==========
    
    /**
     * @notice Activate governance system
     */
    function activateGovernance() external onlyOwner {
        require(!governanceActive, "Governance already active");
        governanceActive = true;
    }
    
    /**
     * @notice Calculate voting power for governance
     * @param voter Address to calculate voting power for
     */
    function calculateVotingPower(address voter) external view returns (uint256) {
        if (!governanceActive) return 0;
        
        uint256 baseVotingPower = balanceOf(voter);
        
        // Add membership tier multiplier
        uint256 tierMultiplier = 100; // 1.0x base
        if (membershipExpiry[voter] > block.timestamp) {
            if (userTiers[voter] == MembershipTier.PREMIUM) {
                tierMultiplier = 150; // 1.5x for premium
            } else if (userTiers[voter] == MembershipTier.NORMAL) {
                tierMultiplier = 125; // 1.25x for normal
            }
        }
        
        return (baseVotingPower * tierMultiplier) / 100;
    }
    
    // ========== VIEW FUNCTIONS ==========
    
    /**
     * @notice Get current burn rate
     */
    function getCurrentBurnRate() external view returns (uint256) {
        return _currentBurnRate;
    }
    
    /**
     * @notice Get total burned tokens
     */
    function totalBurned() external view returns (uint256) {
        return _totalBurned;
    }
    
    /**
     * @notice Get effective circulating supply (excluding treasury)
     */
    function circulatingSupply() external view returns (uint256) {
        // Treasury holds 30% + Guardian Reserve 15% = 45% non-circulating
        uint256 nonCirculating = (MAX_SUPPLY * 45) / 100;
        return totalSupply() - nonCirculating;
    }
    
    /**
     * @notice Get user's vault address (CREATE2 deterministic)
     */
    function getUserVault(address user) public view returns (address) {
        return calculateCreate2Address(user, block.chainid);
    }
    
    /**
     * @notice Calculate CREATE2 address for cross-chain consistency
     */
    function calculateCreate2Address(address user, uint256 chainId) public pure returns (address) {
        bytes32 salt = keccak256(abi.encodePacked(user, chainId));
        // This would be implemented with actual factory contract address
        return address(uint160(uint256(keccak256(abi.encodePacked(
            bytes1(0xff),
            address(0), // Factory contract address would go here
            salt,
            keccak256("VaultBytecode") // Actual vault bytecode hash
        )))));
    }
    
    /**
     * @notice Verify guardian signature for any operation
     */
    function verifyGuardianSignature(
        bytes32 messageHash,
        bytes memory signature
    ) public pure returns (bool) {
        address recoveredAddress = messageHash.recover(signature);
        return recoveredAddress == GUARDIAN_ADDRESS;
    }
    
    // ========== ADMIN FUNCTIONS ==========
    
    /**
     * @notice Update membership fees (only owner)
     */
    function updateMembershipFee(MembershipTier tier, uint256 newFee) external onlyOwner {
        membershipFees[tier] = newFee;
    }
    
    /**
     * @notice Add to internal vault balance
     */
    function addToInternalVault(uint256 amount) external onlyOwner {
        require(balanceOf(address(this)) >= amount, "Insufficient contract balance");
        internalVaultBalance += amount;
    }
    
    /**
     * @notice Emergency pause function (if needed for security)
     */
    function emergencyPause() external onlyOwner {
        // Emergency pause logic would go here
        // This would integrate with OpenZeppelin's Pausable if needed
    }
}
```

---

## 🚀 Roadmap & Development Timeline

### **Phase 1: Foundation (Current - Q2 2024)**
- ✅ Whitepaper completion
- 🔄 Testnet deployment and testing
- 🔄 Smart contract audits
- 🔄 Regulatory compliance framework

### **Phase 2: Token Launch (Q3-Q4 2024)**
- 🔜 Private sale completion
- 🔜 Multi-chain deployment
- 🔜 DEX launch on Base Network
- 🔜 Ecosystem partnerships

### **Phase 3: Expansion (2025)**
- 🔜 Guardian Network decentralization
- 🔜 Enterprise OAuth integration
- 🔜 Additional blockchain support
- 🔜 Institutional onboarding

### **Phase 4: Maturity (2026+)**
- 🔜 Full Guardian Network operation
- 🔜 Global regulatory compliance
- 🔜 Ecosystem token standard
- 🔜 Cross-platform integration

---

## ⚖️ Legal & Risk Disclosures

### Important Notices
- **Non-Security Token**: CFI is classified as a utility token, not a security
- **Regulatory Compliance**: Designed for long-term regulatory sustainability
- **No Investment Advice**: This document is for informational purposes only
- **Risk Warning**: All blockchain investments carry inherent risks
- **Utility Focus**: Token value derives from platform utility, not speculation

### Risk Factors
- **Regulatory Changes**: Future regulations may impact token operations
- **Technology Risks**: Smart contract and blockchain network risks
- **Market Volatility**: Token price may fluctuate significantly
- **Competition**: Other security platforms may impact adoption
- **Adoption Risk**: Platform success depends on user adoption

---

## 📞 Contact & Resources

### Official Links
- **Website**: [chain-fi.io](https://chain-fi.io)
- **Documentation**: [docs.chain-fi.io](https://docs.chain-fi.io)
- **Whitepaper**: [chain-fi.io/whitepaper](https://chain-fi.io/whitepaper)
- **GitHub**: [github.com/chain-fi](https://github.com/chain-fi)

### Community
- **Discord**: [discord.gg/chainfi](https://discord.gg/chainfi)
- **Telegram**: [t.me/chainfi](https://t.me/chainfi)
- **Twitter**: [@ChainFiProtocol](https://twitter.com/ChainFiProtocol)
- **LinkedIn**: [Chain-Fi](https://linkedin.com/company/chain-fi)

### Support
- **Email**: support@chain-fi.io
- **Technical**: dev@chain-fi.io
- **Business**: partnerships@chain-fi.io
- **Press**: media@chain-fi.io

---

**© 2024 Chain-Fi Protocol. All rights reserved.**

*This README serves as the foundational documentation for the CFI token. The smart contract code provided is a first draft and will undergo extensive testing, auditing, and refinement before mainnet deployment.*
