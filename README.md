# RustBlockchain

## Cryptocurrency Blockchains

Two main data structures:
* The blocks in the blockchain
* The transactions within the blocks

Ancillary data:
* Wallets
* Addresses
* Balances
* Peers

### Generic Blockchains (with PoW support)

Blockchain = chronological, sequential list of blocks

Block contain this information:
* Index: this block's location within the list of blocks
* Payload: any relevant information or events that have occured for/in the block
* Timestamp: gives our blockchain a sense of time
* Nonce: special number used for mining (for PoW verification)
* Previous block hash: cryptographic fingerprint of previous block
* Hash: cryptographic fingerprint of all of the above data concatenated together

### Concept: Hashing

#### What is hashing?

In a nutshell, a hash algorithm consists of a set of irreversible computations that can be performed on a datum to generate a (usually) unique byte sequence.

MD5("cryptocurrency")     = edb17c209161dc46762f32e6ef842f7f

SHA-1("cryptocurrency")   = 8548d20d051479841ac25ded8b86b1dab70299bb

SHA-256("cryptocurrency") = a81246023e3f6c6167a08ba224409026f88bb8e98ed1431cd53cb63a328c6e84


