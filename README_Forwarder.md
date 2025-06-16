# ChainFi Protocol Documentation

This repository contains comprehensive documentation for the ChainFi protocol ecosystem.

## Core Documentation

### System Components

- **[ChainFiForwarder](./README_ChainFiForwarder.md)** - Gasless Meta-Transaction System
  - Meta-transaction pattern explanation
  - Payment server architecture  
  - Complete flow from user request to vault execution
  - Integration guides for developers

- **[Gas Abstraction](./gasabstraction.md)** - Gas Abstraction Architecture
  - Payment server validation system
  - Economic protection mechanisms
  - User experience improvements

- **[CFI Token](./CFI.md)** - ChainFi Token Documentation
  - Token economics and mechanics
  - Cross-chain functionality
  - Governance features

- **[ChainFi Swap](./README_ChainFiSwap.md)** - Cross-Chain Bridge System
  - Cross-chain token transfers
  - Security mechanisms
  - Integration documentation

- **[Payment Validation](./PaymentValidation.md)** - Payment System Architecture
  - Validation mechanisms
  - Security protocols
  - API documentation

### Security & Operations

- **[Director of Security](./directorofsecurity.md)** - Security Framework
  - Security protocols
  - Emergency procedures
  - Risk management

- **[HSM Integration](./HSM.md)** - Hardware Security Module
  - Key management
  - Secure operations
  - Enterprise security

## Quick Start

### For Users
1. Create a ChainFi vault
2. Set up gasless transactions via the forwarder
3. Enjoy fee-free asset management

### For Developers
1. Review the [ChainFiForwarder documentation](./README_ChainFiForwarder.md)
2. Implement meta-transaction support
3. Integrate with the payment server API

### For Integrators
1. Study the gas abstraction architecture
2. Implement payment server validation
3. Deploy with ChainGuard security

## Development Tools

## Foundry

**Foundry is a blazing fast, portable and modular toolkit for Ethereum application development written in Rust.**

Foundry consists of:

-   **Forge**: Ethereum testing framework (like Truffle, Hardhat and DappTools).
-   **Cast**: Swiss army knife for interacting with EVM smart contracts, sending transactions and getting chain data.
-   **Anvil**: Local Ethereum node, akin to Ganache, Hardhat Network.
-   **Chisel**: Fast, utilitarian, and verbose solidity REPL.

## Documentation

https://book.getfoundry.sh/

## Usage

### Build

```shell
$ forge build
```

### Test

```shell
$ forge test
```

### Format

```shell
$ forge fmt
```

### Gas Snapshots

```shell
$ forge snapshot
```

### Anvil

```shell
$ anvil
```

### Deploy

```shell
$ forge script script/Counter.s.sol:CounterScript --rpc-url <your_rpc_url> --private-key <your_private_key>
```

### Cast

```shell
$ cast <subcommand>
```

### Help

```shell
$ forge --help
$ anvil --help
$ cast --help
```
