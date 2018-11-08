## Zero Knowledge Range Proof and Set Membership

This repository contains ING's Zero Knowledge Range Proof (ZKRP) and Zero Knowledge Set Membership (ZKSM) protocols. The current implementation of the Ethereum smart contract is based on the paper "Efficient Proofs that a Committed Number Lies in an Interval" by Fabrice Boudot. The repository also contains the implementation of the paper "Efficient protocols for set membership and range proofs", by Jan Camenisch, Rafik Chaabouni and abhi shelat. 

Among the main benefits of using ZKRP and ZKSM it is possible remark the following:

* they are significantly more efficient than generic Zero Knowledge Proofs, like is the case of zkSNARKS. 
* they have being used to provide private transactions on Monero, zkLedger, Confidential Transactions and many others.    
* Although ZKRP allows to construct private transactions, the size of the proof would make the size of transactions too big. Therefore in order to have practical solutions it is important to research efficient implementations of ZKRPs, and this is the main purpose of this repository. 
* Since ZKRP is a subcase of ZKSM, we have that ZKSM may be used in replacement of ZKRP, and this is interesting because for certain scenarios we have that it leads to better performance. 

## ZKSM use cases:

ZKSM allows to prove that some secret value is an element of a determined set, without disclosing which value. Namely, we can do the following:

* Prove that we live in a country that belongs to the European Union. 
* Validation of KYC private data. For example, proving that a postcode is valid, without revealing it. 
* Private Identity Management Systems.
* Other interesting applications like: Anti-Money Laundering (AML) and Common Reference Standard (CRS).

## Range Proof in Ethereum 

One fundamental concern in blockchain technology is the confidentiality of the data on the blockchain. In order to reach consensus between all independent nodes in a blockchain network, each node must be able to validate all transactions (for instance against double-spend), in most cases this means that the content of the transactions is visible to all nodes. Fortunately several solutions exist that preserve confidentiality on a blockchain (private transactions, HyperLedger Fabric Channels, Payment Channels, Homomorphic encryption, transaction-mixing, zero knowledge proofs etc.). This article describes the implementation of a Zero Knowledge Range Proof in Ethereum.

The Zero Knowledge Range Proof allows the blockchain network to validate that a secret number is within known limits without disclosing the secret number. This is useful to reach consensus in a variety of use cases:

 * Validate that someone's age is between 18 and 65 without disclosing the age.
 * Validate that someone is in Europe without disclosing the exact location.
 * Validate that a payment-amount is positive without disclosing the amount (as done by Monero).

The Zero Knowledge Range Proof requires a commitment on a number by a trusted party (for instance a government committing on someone's age), an Ethereum user can use this commitment to generate a range proof. The Ethereum network will verify this proof.


## Precompiled contract

This version is based on the Byzantium precompiles. The range proof consists of 2 parts:
 * Generating the proof that a number is within an interval (outside the blockchain by the client that submits that proof).
 * Validating the proof that this number is within that interval (executed by each validating node on the blockchain).

On Ethereum validation of transactions in smart contract logic is typically done in the Ethereum Virtual Machine. However the operations involved in the validation of this range-proof are too computationally expensive to run on the EVM. Therefore we call a precompiled contract during verification. A precompiled contract is written in the native language of the Ethereum-client (in our case in Golang) and is preconfigured to live at a specific address (with a low number). In our case we use the precompile bigModExp at address 0x5, which is available in Ethereum since the Byzantium release.

 ## Gas consumption

 Ethereum uses the concept of gas which means the sender of a transaction needs to pay (i.e. Eth or Etc) for the computational steps executed by the smart-contract that is invoked by the transaction. The more complex computations the smart contract executes, the more gas will be consumed. Therefore the transaction specifies a gas limit and a gas price.

 Gas limit is there to protect you from buggy code running until your funds are depleted. The product of gasPrice and gas represents the maximum amount of Wei that you are willing to pay for executing the transaction. What you specify as gasPrice is used by miners to rank transactions for inclusion in the blockchain. It is the price in Wei of one unit of gas, in which VM operations are priced.

 The current implementation is not yet optimized for gas usage, the verification costs are around 3 million gas. On private networks this is usually not an issue, but it may be too expensive in the Ethereum mainnet.

## Usage

The usage of this go-ethereum library consists of four parts:
1. Running an Ethereum client with a chain that enables the Byzantium precompiles;
2. Generating a commitment (in Java);
3. Generating a range proof (in Java);
4. Validating range proofs with the Solidity smart contract (connecting from Java, uPort, or via console).

We will describe each step in more detail below. We have tried to make these instructions accessible for a broad audience.

#### Running the Ethereum client

Make sure that you have `geth` installed. If not, follow the instructions on https://github.com/ethereum/go-ethereum/wiki/Building-Ethereum

From the zkrangeproof working directory, you can initialize a private chain using the provided `genesis.json` file, for example:
```
geth init ./data/genesis.json
geth --targetgaslimit 99900000000 --networkid 15997 --rpc --rpcaddr 0.0.0.0 --rpcport 8545 --rpccorsdomain "*"
```

#### Generating commitments

Open a project in your favourite IDE and make sure that you have an IDE (plugin) that supports Gradle, or another dependency manager.
Note that this is a go-ethereum library; you cannot 'run' anything directly.

You can now i.e. add a new java package to the project.
The input for the commitment is a 'secret' `BigInteger` (`java.math.BigInteger`) and a `SecretOrderGroup`. You can generate the `SecretOrderGroup` with:
```java
// import com.ing.blockchain.zk.components.SecretOrderGroupGenerator
// import com.ing.blockchain.zk.dto.SecretOrderGroup
new SecretOrderGroupGenerator(512).generate();
```
where `512` is the bitlength (higher is safer, but also reduces the efficiency).

You can now generate the TTPMessage, which contains the commitment, using:
```java
// import com.ing.blockchain.zk.TTPGenerator
// import com.ing.blockchain.zk.dto.TTPMessage
TTPMessage ttpMessage = TTPGenerator.generateTTPMessage(x, group);
```
where `x` is the 'secret' `BigInteger` and `group` the `SecretOrderGroup`.

This TTPMessage contains six attributes: the first four together describe the commitment, whereas the latter two are private variables for the user that are also needed to generate range proofs.
The commitment can be retrieved with the `getCommitment()` method, and the private variables can be retrieved with the `getX()` and `getY()` methods.
The four different segments of the commitment can then be retrieved by calling the methods `getCommitmentValue()`, `getGroup().getN()`, `getGroup().getG()`, and `getGroup().getH()` on the result of `getCommitment()`.

In the precompiled smart contract we expect the commitment to be (only) the first four variables of the TTPMessage, separated by commas.

#### Generating range proofs

Given the commitment and the user's private variables, we can generate proofs for specific ranges. Hence, we need the TTPMessage here.
Other inputs are a lower bound and an upper bound for the range proof. The bounds need to match with the bounds that are being checked by the relying party.
The range must be supplied as a `ClosedRange`:
```java
// import com.ing.blockchain.zk.dto.*
ClosedRange range = ClosedRange.of(lower, upper);
```
where `lower` and `upper` are BigIntegers.

A range proof should only be generated if this range contains the secret value:
```java
if (range.contains(ttpMessage.getX()))
```

The range proof can now be generated by calling:
```java
// import com.ing.blockchain.zk.RangeProof
BoudotRangeProof rangeProof = RangeProof.calculateRangeProof(ttpMessage, range);
```

And can directly be verified by:
```java
RangeProof.validateRangeProof(rangeProof, ttpMessage.getCommitment(), range);
```

The range proof can be exported for the Solidity contract using the provided ExportUtils:
```java
ExportUtil.exportForEVM(commitment))

        System.out.println("Proof = ");
        System.out.println(DatatypeConverter.printHexBinary(ExportUtil.exportForEVM(rangeProof, commitment, range)));
```

In the precompiled smart contract we expect the range proof to be the above variables (in this order), separated by commas.

#### Validating proofs by calling the precompiled smart contract

Another way to verify a range proof is by calling the precompiled smart contract that is embedded in our modified Geth client. Make sure that the modified Geth is running (see the first section of this instruction).

You first need to set up an Ethereum client in Java (i.e. via web3j). Once set up, you can call the precompiled contract at address `0x0000000000000000000000000000000000000009`. You can call the following method to validate the range proof:
```java
validate(lower, upper, commitment, rangeProof);
```
which returns `true` iff the range proof is valid for this range and commitment.

It is also possible to call the smart contract from uPort by instantiating another web3 instance that connects to the local modified Geth client. Please refer to the uPort documentation for calling smart contracts. Once instantiated by providing an ABI (embedded in the code snippet below), you can call the `validate` method on the smart contract as illustrated in the snippet above.

For testing purposes you can also call the precompiled smart contract from the command line. Start the modified Geth client with the argument `console`, or open a new terminal using `geth attach` to be able to execute commands from the command line.
First, instantiate the smart contract:
```
var precompiled = eth.contract([{"constant":true,"inputs":[{"name":"lower","type":"uint64"},{"name":"upper","type":"uint64"},{"name":"commitment","type":"string"},{"name":"proof","type":"string"}],"name":"validate","outputs":[{"name":"","type":"bool"}],"payable":false,"stateMutability":"view","type":"function"}]).at("0x0000000000000000000000000000000000000009");
```
this always returns `undefined`. Now, call the contract by filling in the variables below:
```
precompiled.validate(lower, upper, "commitment", "rangeProof");
```
which returns `true` iff the range proof is valid for this range and commitment.

Please note that the variables `commitment` and `rangeProof` should be the comma separated combination of the `BigInteger` variables specified in this instruction. The commitment is only the first four variables of the TTPMessage.


