## RFP: ZKPassport

### Abstract

This proposal introduces a standard protocol for generating identity-related attestations from a Mina wallet. It aims to enhance the trust and integrity of digital identity systems by allowing users to prove the validity of their identity attributes without revealing the underlying data. This approach not only safeguards user privacy but also provides a robust framework for decentralized identity verification, making it applicable across various domains such as finance, healthcare, and online services. Through the implementation of ZKPassport, Mina wallet users can achieve higher levels of security and privacy in their digital interactions while ZKApps achieve a higher level of defense against sybil attacks and the ability to make their applications compliant with local laws.

### How will users provide the correct documents?

Most modern passports are Biometric passports or ePassports and contain a chip that can be scanned with NFC. These passports contain information regarding the passport such as country, age, date of birth, etc.

When scanned via a trusted mobile app, users can confidently produce a ZK Proof attesting to specific data on their passport chip. To gain a better understanding of which data can be accessed, please read the [ICAO specification](https://www.icao.int/publications/pages/publication.aspx?docnum=9303). Additionally, the public keys for checking the validity of this data can be found [here](https://download.pkd.icao.int/).

It's important to note that different issuer countries use different signature algorithms.

### Specification

Generating a proof will involve multiple steps:

#### Register Circuit

The register circuit is used for the following:

- Verify the signature of the passport
- Verify that the public key which signed the passport is part of the registry Merkle tree (a check of the Merkle roots will be performed on-chain)
- Generate commitment = H (secret + passportData + some other data)

Once the proof is generated, the user can register on-chain and their commitment will be added to the Lean Merkle tree.

As the hash function and signature algorithm differ upon the issuer country, there will be a need for a single circuit which must support many different fucntions or sperate circuits per country. However one verifier for each register circuit will be deployed on-chain, all of them committing to the same Merkle tree.

### Storing the Merkle Tree Using Offchain Storage

To ensure the Merkle tree containing key-value pairs of users' public keys and commitments is publicly accessible, we must maintain the following:

1. Very high uptime
2. Free read/write access
3. Decentralization to ensure trustlessness and resilience

We can achieve this using [Mina's Offchain Storage API](https://docs.minaprotocol.com/zkapps/writing-a-zkapp/feature-overview/offchain-storage). Below is a solution utilizing this API to store and manage the Merkle tree.

#### Utilizing Offchain Storage

1. **Set up Offchain Storage**:
   First, import the necessary components from the `o1js` library.

   ```javascript
   import { Experimental } from 'o1js';

   const { OffchainState, OffchainStateCommitments } = Experimental;
   ```

2. **Define Offchain State Configuration**:
   Define the configuration for the Offchain state, including the key-value map for the Merkle tree.

   ```javascript
   const offchainState = OffchainState({
     merkleTree: OffchainState.Map(PublicKey, Commitment),
   });

   class StateProof extends offchainState.Proof {}
   ```

3. **Set up the Smart Contract**:
   Initialize the smart contract and assign it to the Offchain storage. This will compile the recursive Offchain zkProgram in the background and assign the Offchain state to the smart contract instance.

   ```javascript
   let contract = new MyContract(contractAddress);
   offchainState.setContractInstance(contract);

   // Compile Offchain state program
   await offchainState.compile();
   // Compile smart contract
   await MyContract.compile();
   ```

4. **Settle Offchain State**:
   To settle the offchain state, generate an Offchain storage proof and provide it to the smart contract's `settle` method.

   ```javascript
   let proof = await offchainState.createSettlementProof();

   await Mina.transaction(sender, () => {
     // Settle all outstanding state changes
     contract.settle(proof);
   })
     .sign([sender.key])
     .prove()
     .send();
   ```

5. **Configure Smart Contract**:
   The smart contract requires a field containing a commitment to the offchain state. This field is used internally by the OffchainState methods and should not be written to by your smart contract logic.

   ```javascript
   class MyContract extends SmartContract {
     @state(OffchainStateCommitments) offchainState = State(
       OffchainStateCommitments.empty()
     );

     @method
     async settle(proof: StateProof) {
       await offchainState.settle(proof);
     }
   }
   ```

6. **Using Offchain Storage**:
   Now, developers can utilize Offchain storage in any of their smart contract methods. Below is an example of how to use the Offchain storage to update the Merkle tree.

   ```javascript
   class MyContract extends SmartContract {
     @method
     async updateMerkleTree(publicKey: PublicKey, commitment: Commitment) {
       // Retrieve the current state of the Merkle tree entry for the public key
       let commitmentOption = await offchainState.fields.merkleTree.get(publicKey);

       // Update the Merkle tree with the new commitment
       offchainState.fields.merkleTree.update(publicKey, {
         from: commitmentOption,
         to: commitment,
       });
     }
   }
   ```


### Disclose Circuit

The disclose circuit is used for the following:

- Verify that a user knows a secret, e.g., they can reconstruct one leaf of the Merkle tree (a check of the Merkle roots will be performed on-chain)
- Verify passport expiry
- Perform a range check over the age of the user
- Multiply the output by an input bitmap to allow the user to disclose only what they want to disclose
- Pack the final output

Any application that wants to use Proof of Passport can actually build its own disclose circuit.

### Circuit Specification

### Use Case Scenarios

1. **Financial Services**: A user can prove their identity to a bank without revealing their actual age, nationality, or other sensitive data, thereby complying with KYC requirements while maintaining privacy.

2. **Healthcare**: Patients can prove their eligibility for age-specific healthcare services without disclosing their exact date of birth.

3. **Online Services**: Users can verify their identity to access age-restricted online services without exposing their full identity details.
