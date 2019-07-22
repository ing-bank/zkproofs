# THIS PROJECT IS OUTDATED AND HAS BEEN ARCHIVED! Please see [ing-bank/zkrp](https://github.com/ing-bank/zkrp) for an updated version.

Please note that no maintenance is done on this code base. This means that it might contain serious security issues. Please dont't use this code. For an updated and secure version, please use [ing-bank/bulletproofs](https://github.com/ing-bank/bulletproofs).

Old README:

# Zero Knowledge Proofs

This repository contains ING's **Zero Knowledge Range Proof (ZKRP)** and **Zero Knowledge Set Membership (ZKSM)**. The current implementations are based on the following papers:
* Range Proofs based on the paper: [Efficient Proofs that a Committed Number Lies in an Interval](https://www.iacr.org/archive/eurocrypt2000/1807/18070437-new.pdf) by **Fabrice Boudot**. 
* Set Membership Proofs based on the paper: [Efficient protocols for set membership and range proofs](https://infoscience.epfl.ch/record/128718/files/CCS08.pdf), by **Jan Camenisch, Rafik Chaabouni and Abhi Shelat**. 
* Bulletproofs based on paper: [Bulletproofs: Short Proofs for Confidential Transactions and More](https://eprint.iacr.org/2017/1066.pdf), by **Benedikt Bünz, Jonathan Bootle, Dan Boneh, Andrew Poelstra, Pieter Wuille and Greg Maxwell**.

## Getting Started

### Highlights :rocket:

* Significantly more efficient than generic Zero Knowledge Proofs, like is the case of zkSNARKS. 
* Currently used to provide private transactions on Monero, zkLedger, Confidential Transactions and many others.

### Zero Knowledge Range Proofs

One fundamental concern in blockchain technology is the confidentiality of the data. In order to reach consensus between all independent nodes, each node must be able to validate all transactions (for instance against double-spend), in most cases this means that the content of the transactions is visible to all nodes. Fortunately, several solutions exist that preserve confidentiality on a blockchain (private transactions, HyperLedger Fabric Channels, Payment Channels, Homomorphic encryption, transaction-mixing, zero knowledge proofs etc.).

The Zero Knowledge Range Proof allows the blockchain network to validate that a secret number is within known limits without disclosing the secret number. This is useful to reach consensus in a variety of use cases:

 * Validate that someone's age is between 18 and 65 without disclosing the age.
 * Validate that someone is in Europe without disclosing the exact location.
 * Validate that a payment-amount is positive without disclosing the amount (as done by Monero).

The Zero Knowledge Range Proof requires a commitment on a number by a trusted party (for instance a government committing on someone's age), an Ethereum user can use this commitment to generate a range proof. The Ethereum network will verify this proof.

### Zero Knowledge Set Membership Proofs

> **Since ZKRP is a subcase of ZK Set Membership Proofs, the latter may be used as a replacement of ZKRP. This is interesting because for certain scenarios it performs better.**

ZKSM allows to prove that some secret value is an element of a determined set, without disclosing which value. We can do the following examples using it:

* Prove that we live in a country that belongs to the European Union. 
* Validation of KYC private data. For example, proving that a postcode is valid, without revealing it. 
* Private Identity Management Systems.
* Other interesting applications like: Anti-Money Laundering (AML) and Common Reference Standard (CRS).


### Bulletproofs

In 2017 researchers proposed the scheme called Bulletproofs to provide a more efficient solution for Zero Knowledge Range Proofs (ZKRP). It was specifically designed for Blockchain, where it is important to have short proofs. For instance, Bulletproofs allows to construct proofs whose size is only logarithmic with respect to the input size. Also, Bulletproofs doesn't require a trusted setup, solving an important problem in order to use this technology to solve practical problems. Previous solutions do require a trusted setup, what means that if the setup is not carried out in an appropriate way, then it would be possible to generate fake ZK proofs. 

Bulletproofs can be used to solve the abovementioned problems and even more, because it is possible to use it for any computable function which requires privacy for its input data. Therefore, Bulletproofs is similiar to zk-SNARKs and zk-STARKs. However, this functionalities were not yet implemented and should be considered as future work. In particular, Bulletproofs seems an interesting building block to construct private smart contracts. 


## Contribute :wave:

We would love your contributions. Please feel free to submit any PR.

## License

This repository is GNU Lesser General Public License v3.0 licensed, as found in the [LICENSE file](https://github.com/ing-bank/zkproofs/blob/master/LICENSE.txt).




