## Zero Knowledge Range Proof

ING's zero-knowledge range proof precompiled contract for the go-ethereum client.

## Important note:

The current version of this library implements the whitepaper “An Efficient Range Proof Scheme” by Kun Peng and Feng Bao. As discovered by Madars Virza, Research Scientist MIT Media Lab, this protocol contains a potential security vulnerability. 

*“The publicly computable value y/t is roughly the same magnitude (in expectation) as w^2 \* (m-a+1)(b-m+1). However, w^2 has fixed bit length (again, in expectation) and thus for a fixed range, this value leaks the magnitude of the committed value.”*

Therefore, the proof is not zero knowledge. We’re currently evaluating which protocol to use instead in order to provide a secure Zero Knowledge Proof protocol.

## Intro

One fundamental concern in blockchain technology is the confidentiality of the data on the blockchain. In order to reach consensus between all independent nodes in a blockchain network, each node must be able to validate all transactions (for instance against double-spent), in most cases this means that the content of the transactions is visible to all nodes. Fortunately several solutions exist that preserve confidentiality on a blockchain (private transactions, HyperLedger Fabric Channels, Payment Channels, Homomorphic encryption, transaction-mixing, zero knowledge proofs etc.). This article describes the implementation of a zero-knowledge range-proof in Ethereum.

The zero knowledge range proof allows the blockchain network to validate that a secret number is within known limits without disclosing the secret number. This is useful to reach consensus in a variety of use cases:

 * Validate that someone's age is between 18 and 65 without disclosing the age.
 * Validate that someone is in Europe without disclosing the exact location.
 * Validate that a payment-amount is positive without disclosing the amount (as done by Monero).

The zero-knowledge range-proof requires a commitment on a number by a trusted party (for instance a government committing on someone's age), an Ethereum-user can use this commitment to generate a range-proof. The Ethereum network will verify this proof.


## Fiat–Shamir

Though the original 'Efficient range-proof' by Kun Peng required interaction between the prover and the validator, we adjusted the protocol to become non-interactive so that it would become usable on a blockchain (where each node needs to be able to verify autonomously without interaction with the client). We made the protocol non-interactive using the Fiat–Shamir heuristic.

## Precompiled contract

The range proof consists of 2 parts:
 * Generating the proof that a number is within an interval (outside the blockchain by the client that submits that proof)
 * Validating the proof that this number is within that interval (executed by each validating node on the blockchain)

On Ethereum validation of transactions in smart contract logic is typically done in the Ethereum Virtual Machine. However the operations involved in the validation of this range-proof are too computationally expensive to run on the EVM. Therefore we validate the range proof in a precompiled contract. We added this precompiled contract to the Ethereum Go Client (Geth).  A precompiled contract is written in the native language of the Ethereum-client (in our case in Golang) and is preconfigured to live at a specific address (with a low number). The precompiled contract can be called from Solidity in 2 ways:

 * By referring to the address with a Solidity interface (works until Solidity 0.3.6. and requires the address to have a balance of at least one wei (preconfigured in the genesis block)).
 * By extending the Solidity language to include additional functions in which case the Solidity code will be compiled to call the precompiled smart contract at the same preconfigured address.

 ## Gas consumption

 Ethereum uses the concept of gas which means the sender of a transaction needs to pay (i.e. Eth or Etc) for the computational steps executed by the smart-contract that is invoked by the transaction. The more complex computations the smart contract executes, the more gas will be consumed. Therefore the transaction specifies a gas limit and a gas price.

 Gas limit is there to protect you from buggy code running until your funds are depleted. The product of gasPrice and gas represents the maximum amount of Wei that you are willing to pay for executing the transaction. What you specify as gasPrice is used by miners to rank transactions for inclusion in the blockchain. It is the price in Wei of one unit of gas, in which VM operations are priced.



 Determining the right gas-consumption is crucial for correct functioning of Ethereum. Too low gas introduces a DOS vulnerability, attackers can make the network slow by calling computationally hard functions while paying relatively little. Too high gas wastes people’s money.


 We benchmarked the zkRangeProof verification against various other built-in Ethereum functions which resulted in a gas-consumption of 180,000.
