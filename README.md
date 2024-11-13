# Avalanche Subnet - Module 01 Metacrafters

This project demonstrates the creation and deployment of a custom Avalanche subnet based on Polygon, integrating a sample ERC20 token and Vault contract. It includes steps to set up and deploy the subnet locally, followed by deploying contracts like an ERC20 token and a Vault contract for token deposits and withdrawals.

## Table of Contents

- [Overview](#overview)
- [Prerequisites](#prerequisites)
- [Setup](#setup)
- [Commands](#commands)
  - [Create Subnet](#create-subnet)
  - [Deploy Subnet](#deploy-subnet)
  - [Deploy Contracts](#deploy-contracts)
- [Contracts](#contracts)
  - [ERC20 Token](#erc20-token)
  - [Vault Contract](#vault-contract)
- [Usage](#usage)

## Overview

This repository walks you through the process of setting up an Avalanche subnet using the `avalanche-cli` tool. After creating and configuring the subnet, the repository deploys an ERC20 token (MetaToken) and a Vault contract for managing deposits and withdrawals of the token. 

The key steps include:
1. Creating a custom subnet for Polygon.
2. Deploying the subnet locally for testing.
3. Deploying ERC20 and Vault smart contracts.
4. Using a Relayer to handle cross-chain communications between the Avalanche C-chain and the Polygon subnet.

## Prerequisites

To get started with this project, ensure you have the following installed and configured:

- [Avalanche CLI](https://github.com/ava-labs/avalanche-cli)
- [Go](https://golang.org/dl/)
- [Node.js](https://nodejs.org/)
- [Solidity Compiler](https://soliditylang.org/)
- A basic understanding of Avalanche subnets, Ethereum-based smart contracts (ERC20), and Solidity.

## Setup

### 1. Install Avalanche CLI

Follow the installation steps from the [official Avalanche CLI repository](https://github.com/ava-labs/avalanche-cli). Ensure you have version `v1.11.12` or above.

```bash
curl -sSL https://github.com/ava-labs/avalanche-cli/releases/download/v1.11.12/avalanche-cli-linux-x86_64-v1.11.12.tar.gz | tar xz -C /usr/local/bin
```

### 2. Create and Configure Avalanche Subnet

Once the CLI is set up, you'll create a new Polygon-based subnet. Use the following commands to generate a new subnet configuration and deploy it locally.

```bash
avalanche subnet create polygon
```

This will create a subnet with default settings suitable for testing, including a chain ID of `1001` and a token symbol of `POL`.

### 3. Deploy the Subnet

After the subnet is created, deploy it to the local network:

```bash
avalanche subnet deploy polygon
```

This command sets up a local network and boots the subnet. It will also deploy key contracts, such as the Teleporter Messenger and Teleporter Registry, allowing cross-chain interactions between the Avalanche C-chain and your Polygon subnet.

## Commands

### Create Subnet

```bash
avalanche subnet create polygon
```

This will create the Polygon subnet with default settings:
- Chain ID: `1001`
- Token Symbol: `POL`
- Prefunding address: `0x8db97C7cEcE249c2b98bDC0226Cc4C2A57BF52FC`

### Deploy Subnet

```bash
avalanche subnet deploy polygon
```

This command deploys the configured subnet to a local network, initializing necessary network components, including:
- Teleporter Messenger and Registry for cross-chain communication.

## Contracts

### ERC20 Token

The ERC20 token contract (MetaToken) follows the standard ERC20 implementation, allowing for minting, transferring, and burning of tokens. Here’s the code for the token contract:

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.17;

contract ERC20 {
    uint public totalSupply;
    mapping(address => uint) public balanceOf;
    mapping(address => mapping(address => uint)) public allowance;
    string public name = "MetaToken";
    string public symbol = "MTK";
    uint8 public decimals = 18;

    event Transfer(address indexed from, address indexed to, uint value);
    event Approval(address indexed owner, address indexed spender, uint value);

    function transfer(address recipient, uint amount) external returns (bool) {
        balanceOf[msg.sender] -= amount;
        balanceOf[recipient] += amount;
        emit Transfer(msg.sender, recipient, amount);
        return true;
    }

    function approve(address spender, uint amount) external returns (bool) {
        allowance[msg.sender][spender] = amount;
        emit Approval(msg.sender, spender, amount);
        return true;
    }

    function transferFrom(address sender, address recipient, uint amount) external returns (bool) {
        allowance[sender][msg.sender] -= amount;
        balanceOf[sender] -= amount;
        balanceOf[recipient] += amount;
        emit Transfer(sender, recipient, amount);
        return true;
    }

    function mint(uint amount) external {
        balanceOf[msg.sender] += amount;
        totalSupply += amount;
        emit Transfer(address(0), msg.sender, amount);
    }

    function burn(uint amount) external {
        balanceOf[msg.sender] -= amount;
        totalSupply -= amount;
        emit Transfer(msg.sender, address(0), amount);
    }
}
```

### Vault Contract

The Vault contract allows users to deposit ERC20 tokens and mint shares in return, while also allowing them to withdraw their tokens in exchange for burning their shares. Here's the code for the Vault contract:

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.17;

interface IERC20 {
    function totalSupply() external view returns (uint);
    function balanceOf(address account) external view returns (uint);
    function transfer(address recipient, uint amount) external returns (bool);
    function allowance(address owner, address spender) external view returns (uint);
    function approve(address spender, uint amount) external returns (bool);
    function transferFrom(address sender, address recipient, uint amount) external returns (bool);

    event Transfer(address indexed from, address indexed to, uint value);
    event Approval(address indexed owner, address indexed spender, uint value);
}

contract Vault {
    IERC20 public immutable token;
    uint public totalSupply;
    mapping(address => uint) public balanceOf;

    constructor(address _token) {
        token = IERC20(_token);
    }

    function _mint(address _to, uint _shares) private {
        totalSupply += _shares;
        balanceOf[_to] += _shares;
    }

    function _burn(address _from, uint _shares) private {
        totalSupply -= _shares;
        balanceOf[_from] -= _shares;
    }

    function deposit(uint _amount) external {
        uint shares;
        if (totalSupply == 0) {
            shares = _amount;
        } else {
            shares = (_amount * totalSupply) / token.balanceOf(address(this));
        }

        _mint(msg.sender, shares);
        token.transferFrom(msg.sender, address(this), _amount);
    }

    function withdraw(uint _shares) external {
        uint amount = (_shares * token.balanceOf(address(this))) / totalSupply;
        _burn(msg.sender, _shares);
        token.transfer(msg.sender, amount);
    }
}
```

## Usage

1. **Deploy the Subnet**:
   - Use the commands mentioned above to create and deploy your Avalanche subnet locally.
   
2. **Deploy Contracts**:
   - Deploy the ERC20 and Vault contracts on your local subnet via Remix, Truffle, or any Solidity IDE of your choice.
   
3. **Interact with Contracts**:
   - Once deployed, users can interact with the ERC20 token and Vault contract using Web3, ethers.js, or similar libraries.
