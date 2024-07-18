# ZuLand
ZuLand - Private Space for Zuzalu Citizens

## Problem
Zuzalu citizens and villages currently don’t have any private and decentralized social space where they can communicate and organize autonomously while preserving their privacy. 

## Solution
We want to develop a private, decentralized solution where only Zu<City> attendees can read and write content.

We identified three components to make it happen:
- **ZuPass**: Software for storing and managing Proof-Carrying Data (like signatures, Merkle proofs, zk proofs, hash commitments, and keypairs). It allows users to store and organize this data and respond to queries from third-party applications about them.
- **Akasha Core**: Decentralized social network toolkit integrated with Ceramic ComposeDB.
- **Lit Protocol**: Decentralized key management system (KMS) so users can encrypt data and control who can access and use their data.

Our idea is to extend Akasha Core with Lit Protocol, allowing developers to use ready-made token-gated components for their social networks with customizable requirements and use ZuPass or ZuCitizenship to authenticate users in the social network and encrypt/decrypt content shared within the network.

ZuLand will be configured as a social extension of the Zuzalu.City platform, becoming part of a set of plug-in features from which each community can benefit.

## Team Members
- **drivenfast** - Core Zuzalu.City
- **limone.eth** - PM & Fullstack Dev at builders.garden
  - [warpcast.com/limone.eth](https://warpcast.com/limone.eth)
  - [github.com/limone-eth](https://github.com/limone-eth)
- **itsmide.eth** - Backend Dev at builders.garden
  - [warpcast.com/itsmide.eth](https://warpcast.com/itsmide.eth)
  - [github.com/mmatteo23](https://github.com/mmatteo23)
- **Bianc8** - Frontend Dev at builders.garden
  - [warpcast.com/bianc8.eth](https://warpcast.com/bianc8.eth)
  - [github.com/bianc8](https://github.com/bianc8)
- **Darph** - Designer at builders.garden
  - [warpcast.com/darph](https://warpcast.com/darph)
- **Martin Ezrodt** - Akasha
  - [github.com/etzm](https://github.com/etzm)
- **Sever Abibula** - Akasha
  - [x.com/Sever__S](https://x.com/Sever__S)
  - [github.com/SeverS](https://github.com/SeverS)
- **Marius Darila** - Akasha
  - [x.com/MariusDarila](https://x.com/MariusDarila)
  - [github.com/kenshyx](https://github.com/kenshyx)
 
## Details about the project
The parties involved in this project (Akasha & Zuzalu) already offer two key components to build the above described project:

- **Akasha Core - Social**: Akasha offers an open source framework for building and distributing composable web3 apps, with a strong focus on social networks.
- **ZuPass - Authentication**: the Zuzalu community developed software for storing and managing *Proof-Carrying Data* allowing trustless and private verifications on any well-formed/valid data

Right now, the two components are not connected/integrated, but there’s a clear opportunity to do so:

- ZuPass can empower a trustless, open, and privacy-preserving authentication method for people who want to participate in the social network
- Akasha Core and its ready-made components can speed up the development of the social network

The missing element in this story is token-gating, making the content private. Akasha Core, by design, is reading and writing data to Ceramic, which by default is public. We need that content to be readable/writable only by a specific subset of people (i.e., ZuGeorgia pass holders and future ZuCitiziens), and Lit Protocol seems the best shot we have to achieve this goal.

### Useful resources/links
- **Lit x Ceramic Integration: Storing Encrypted Data on ComposeDB:** https://spark.litprotocol.com/lit-x-composedb/
- **ZuPass repository**: https://github.com/proofcarryingdata/zupass
- **Akasha Core understanding apps, plugins, widgets**: https://blog.akasha.org/akasha-core-understanding-apps-widgets-plugins/
- **Akasha Core micro-frontends**: https://blog.akasha.org/akasha-core-micro-frontends/
- **Akasha Core repo**: https://github.com/AKASHAorg/akasha-core
- **Zupass and the Proof-Carrying Data SDK**: https://github.com/proofcarryingdata/zupass

## ZuPass 🤝 Lit Protocol
### Why this integration?

Lit Protocol is a decentralized key management system (KMS); users can encrypt data and control who can access and use their data.

The Lit network uses an identity-based encryption scheme, meaning decryption is permitted for those who satisfy a certain pre-determined identity parameter. 
To achieve this, we can create an **Access Control Condition** (ACC) and permit decryption to users who meet the condition we set.

That said, we would like to use **Zupass** as the **ACC**.

### Zupass as Access Control Condition

With Lit Protocol, we have some condition types:

- **Onchain**: conditions based on onchain data (ownership of an NFT or balance of a specific token)
    - EVM
    - Other chains (Solana and Cosmos)
- **Offchain**: conditions based on a **Lit Action Condition**. So, grant access whenever a given Lit Action meets the set conditions. It's a JS code that can be executed on the Lit Protocol Network. This allows create custom access control conditions.
The Lit Action code is the place where you choose your conditions. As said before, it's a Javascript code that you saved on IPFS and you refer to it using its CID. You can pass parameters to one of the functions you've written inside and the function can return any string (not just true or false), and you can compare it against any string using available comparators for strings (`=`, `!=` , `contains`, `!contains`).

Learn more about Lit Actions here here: https://developer.litprotocol.com/sdk/access-control/lit-action-conditions

#### Possible solutions

##### 1. Verification through Groth16Verifier smart contract

Looking at the starterkit released for DevConnect, ETHPrague [*1] and other events, one option is to send the **Proof-Carrying Data** (obtained from user's Zupass) directly to a deployed **Groth16Verifier** **smart contract** and use the output of the `verifyProof` method to verify the user.

The Access Control Condition will be something like:

```jsx
const accessControlConditions = [
  {
    contractAddress: '<our_verifier_contract_address>',
    functionName: 'verifyProof',
    functionParams: ['_pA', 'pB', '_pC', '_pubSignals'], // the proofArgs
    functionAbi: {...},
    chain: '<selected_chain>',
    returnValueTest: {
      comparator: '=',
      value: 'true'
    }
  }
]
```

Because it's a contract call from a contract type that is not standard (like ERC20, ERC721, and ERC1155) we need to pass a function ABI and call the method.

##### 2. Verification using an ERC721 smart contract

Another possibility is to release an NFT token to eligible users (e.g. using a smart contract similar to the one inside the starter [*2]), so we can use a Lit Protocol standard Access Control Condition and write something like:

```jsx
const accessControlConditions = [
  {
    contractAddress: '<our_erc721_contract>',
    standardContractType: 'ERC721',
    chain,
    method: 'balanceOf',
    parameters: [
      ':userAddress'
    ],
    returnValueTest: {
      comparator: '>',
      value: '0'
    }
  }
]
```

##### 3. Custom Lit Action on Zupass verification

Due to the presence of Lit Actions we can write a custom Lit Action condition and define a semplified Zupass verification.

For example, move the verification at step 3 into the Action.

![image](https://github.com/user-attachments/assets/33f0ff7e-339b-4768-9acc-19ad0722f696)

In this case the Access Control Condition would be:

```jsx
var accessControlConditions = [
  {
    contractAddress: "ipfs://QmcgbVu2sJS...S7eYtwoFY",
    standardContractType: "LitAction",
    chain: "ethereum",
    method: "pcdVerification",
    parameters: ["pcd_payload"],
    returnValueTest: {
      comparator: "=",
      value: "true",
    },
  },
];
```

The `contractAddress` is the Lit Action address saved on IPFS. The above condition will run the `pcdVerification()` function of the Lit Action, and check if the return value is true. It will pass the PCD payload as a parameter to the `pcdVerification()` function.

> **Note:** maybe for this integration we’ll need to use a bundler, and provide the bundle as Lit Action.
Here is an example: https://github.com/LIT-Protocol/js-serverless-function-test/tree/main/bundleTests/siwe
> 

### Is this the key to having a “condition-gated” Antenna on Akasha?

One of the most popular use cases of Lit Protocol is storing private data on **ComposeDB** (**Ceramic**), so this suggests a good opportunity to use Lit Protocol to transform a generic Akasha Antenna into a “condition-gated” version, where only specific users that have a specific Zupass credential have access.

### Resources

- **[1] ETHPrague Zupass: SE2 Starter Kit** - https://github.com/BuidlGuidl/ethprague-zupass-starterkit
- **[2] YourCollectible.sol** - https://github.com/BuidlGuidl/devconnect-zupass-se2/blob/main/packages/hardhat/contracts/YourCollectible.sol
- **[3] Zupass Github** - https://github.com/proofcarryingdata/zupass
- **[4] Lit Protocol Access Control** - https://developer.litprotocol.com/sdk/access-control/intro
