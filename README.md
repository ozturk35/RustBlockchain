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

## Generic Blockchains (with PoW support)

Blockchain = chronological, sequential list of blocks

Block contain this information:
* Index: this block's location within the list of blocks
* Payload: any relevant information or events that have occured for/in the block
* Timestamp: gives our blockchain a sense of time
* Nonce: special number used for mining (for PoW verification)
* Previous block hash: cryptographic fingerprint of previous block
* Hash: cryptographic fingerprint of all of the above data concatenated together

## Concept: Hashing

### What is hashing?

In a nutshell, a hash algorithm consists of a set of irreversible computations that can be performed on a datum to generate a (usually) unique byte sequence.

MD5("cryptocurrency")     = edb17c209161dc46762f32e6ef842f7f

SHA-1("cryptocurrency")   = 8548d20d051479841ac25ded8b86b1dab70299bb

SHA-256("cryptocurrency") = a81246023e3f6c6167a08ba224409026f88bb8e98ed1431cd53cb63a328c6e84

## Concept: Mining

### Block Hashing: Review

1. Concatenate together all the bytes composing the block's fields (aside from the hash field)
2. Generate unique data fingerprint: hash

The current version of Bitcoin uses SHA-256^2 (hash of hash of data) for its proof-of-work algorithm "hashcash".

### Difficulty

SHA-256 generates a 32-byte hash. Difficulty (in our case) specifies the unsigned 128-bit integer value that the most significant 16 bytes of the hash of a block must be less than before it is considered "valid" (if those bytes are interpreted as a single number instead of a series of bytes). Difficulty will be stored as a field of the Block struct.

Difficulty could also be expressed as:
* The first n bytes of the hash that must be zero.
* The number of bits or bytes at the beginning of the hash that must be zero.

These options are essentially different ways of expressing the same thing.

Bitcoin stores the difficulty value more compactly than this, but this is simpler and we don't have to worry about space efficiency.

See: [How is difficulty stored in blocks?](https://en.bitcoin.it/wiki/Difficulty#How_is_difficulty_stored_in_blocks.3F)

### Little vs. Big Endian

Endianness: Order of bytes stored in the memory.

Example: 42u32

| Format | Representation |
| :- | :-: |
| Hex representation | 0x0000002a |
| Stored in big-endian order | 00 00 00 2a |
| Stored in little-endian order (most common) | 2a 00 00 00 |

If we treat it like a little endian representation of a number, the most significant 16 bytes of our hash will appear at the end of our hash's byte vector [16,32].

See: [Endianness](https://en.wikipedia.org/wiki/Endianness) and [byteorder](https://crates.io/crates/byteorder)

### Nonce

A hash is unique, reproducible fingerprint for some data. Therefore, to make a "valid" hash (per difficulty) we must somehow change the bytes we send to the function (the pre-image). Remember that even one small change to the input changes the resultant hash drastically. This effect is commonly called avalanching.

Of course, we can't actually change the information stored in a block willy-nilly. Therefore, we introduce an additional piece of data called __nonce__: an arbitrary (but not necessarily random) value added as a field to each block, and hashed along with the data. Since it has been declared arbitrary, we can change it as we please.

You can think of like this: generating the correct hash for a block is like the puzzle, and the nonce is the key to that puzzle. The process of finding that key is called __mining__.

### Mining Algorithm
1. Generate new nonce
2. Hash bytes (this is the computationally heavy step)
3. Check hash against difficulty
    * Insufficient? Go back to step 1
    * Sufficient? Continue to step 4
4. Add block to chain
5. Submit to peers, etc. Since this is out-of-scope for this project and we have no networking capabilities implemented (yet!), we'll just skip this step.

### Review: Mining

A block having been mined means that an amount of effort has been put into discovering a nonce "key" that "unlocks" the block's hash-based puzzle.

Mining has the property that it is a hard problem to solve while its solution is easy to check and verify.

It has a customizable difficulty that should adapt to the amount of effort being put forth by the miners on the network to maintain the average time it takes to mine a block.

Bitcoin adjusts its difficulty every 2016 blocks such that the next 2016 blocks should take two weeks to mine.

See: [What network hash rate results in a given difficulty?](https://en.bitcoin.it/wiki/Difficulty#What_network_hash_rate_results_in_a_given_difficulty.3F)

### Block Verification

We can implement a few rudimentary block verification tests. These steps would be executed whenever we receive a new block from a peer.

Each supposed valid block has a nonce attached to it that we assume took an approximately certain amount of effort to generate. This "approximately certain amount of effort" is described by the difficulty value.

We will verify four things now:
1. Actual index == stored index value (note that Bitcoin blocks don't store their index)
2. Block's hash fits stored difficulty value (we'll just trust the difficulty for now) (insecure)
3. Time is always increasing (IRL network latency/sync demands leniency here)
4. Actual previous block's hash == stored prev_block_hash value (except for genesis block)

The Bitcoin protocol describes [these 19 verification steps for blocks](https://en.bitcoin.it/wiki/Protocol_rules#.22block.22_messages).

## Transactions
### Transaction Verification Requirements

We have to protect against:
* Overspending (where did the money come from?)
* Double-spending(is the money available?)
* Impersonation (who owns the money and who is sending it?)
* ... (there are more, but we're just going to cover these three)

List of rules for a Bitcoin transaction: [Protocol rules](https://en.bitcoin.it/wiki/Protocol_rules#.22tx.22_messages)

### The Blockchain as a "Distributed Ledger"

What does it mean to "own a coin?"
| Block 123 | Block 124 | Block 125 | Block 126
| :- | :- | :- | :- |
| Jaime -> Andrew (15) | Francis -> Chris (34)   | <span style="border: 2px solid cornflowerblue; border-radius: 5px; padding: 0px 4px">Alice -> Bob (12)</span>      | Chris -> Zach (3)
| Chris -> Alice (50) | Michiko -> Bob (7)       | Zach -> Jaime (2)      | Zach -> Chris (2)
|                     | Terrence -> Georgia (87) | Chris -> Terrence (18) | Chris -> Zach (10)
|                     |                          |                        | Zach -> Sophia (18)

Consider the transaction in Block 125 that Alice sends Bob 12 coins.

Where does Alice get the coins?

Chris sent 50 coins to Alice in Block 123. But where does Chris get the coins? We can track the origin of every single coin all the way back to the first block in the blockchain.

### Structure of a Transaction

For us right now, transactions only contain two important pieces of information:
* Set of inputs (which are unused outputs from previous transactions)
* Set of outputs (new outputs that can be used in future transactions)

From here we can calculate:
* the value of the transaction: $\sum inputs$

* the value of the fee: $\sum inputs -  \sum outputs$

### Coinbase Transactions

Where it all starts!

Coinbase transactions:
* do not reuire inputs
* produce an output
* allow the miner to collect all the transaction fees in that block and that block's block reward (coin genesis!)

### Recap

A transaction will take a set of outputs as inputs, and generates a set of outputs in turn.

Example:

[50]->[12],[36]

50 -> 48

And does this mean that coins can get destroyed just like that? Nope! Miners take the leftovers (in this case 2 coins) as their fee. If a transaction does not have room in it for a fee for the miner, what incentive does the miner have to add the transaction to their block?

## Meeting Tx Verification Requirements
### Overspending

Simple: the sum of the values of the inputs must be greater than or equal to the sum of the values of the generated outputs.

I can't input 5 coins and be able to output 7.

### Double-Spending

Make sure that any one output is never used as an input more than once.

This can be done by maintaining a pool of unspent outputs and rejecting any transaction that tries to spend outputs that don't exist in the pool.

### Impersonation

This can be solved by adding a cryptographic signature to outputs to verify they are being spent by their owner.

We can't assume that whoever sent us the transaction over the network is also the person who created the transaction.

For now, we'll kind of ignore solving this problem. We might come back to it when we go over smart contracts.

## Transactions in Rust

### Iterators

Iterators help us process sequences of data. Rust's iterators are especially powerful.

To access the iterator of a vector, call its .iter() function. Then we have access to lazily-evaluated functions like map, flat_map, filter, for_each, any, all, etc.

Another powerful function is .collect::\<T>(). This will evaluate the iterator and form it into whatever type T you give it. The type T must be implement the FromIterator trait, but most of the standard library structures you'd expect this to work on do. For eample: String, Vec, HashMap, HashSet, LinkedList, etc.

### Errors

In Rust, there are essentially two types of errors: Result<T, E> and panic!. Result<T, E>s are just like any other data type, it’s just part of the standard library and recommended that you use it. They take the form of either Result::Ok(T) or Result::Err(E), and you can destructure them as necessary. panic!s, on the other hand, are not usable just like any other data type. They actually will halt the program, like it crashed. Ideally, we want to use Result<T, E>s over panic!s.

### Null

No such thing as null in Rust!

The closest we can get is the standard library's Option\<T>: an enum that communicates the existence of a value. It can take the form of Option::Some(T) or Option::None, where Option::None is probably the closest you will see to null in Rust programs.

### Updating our blockchain

Now we have to maintain a list of unspent outputs. This will just be a set of hashes of the unspent outputs. Note that this does not differentiate between two outputs that are to the same address for the same amount. This will be fixed later.

We have to validate three more conditions now:
* Can we spend the input?
* How many coins are in the output?
* Is the coinbase transaction valid? (We’re going to skimp a bit on this check for now.)
