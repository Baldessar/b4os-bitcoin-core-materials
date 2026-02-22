# Hard 07: Compact Block Relay Fallback (`getblocktxn` / `blocktxn`)

This is a hard exercise! Hints are not provided, and the whole exercise is less guided. Have fun! :)

## Concepts You Need

Compact block relay (BIP152) speeds block propagation by sending compact representations (`cmpctblock`) instead of full tx lists.

Receiver flow:

1. receive `cmpctblock`
2. attempt reconstruction from local mempool
3. if missing txs, request them via `getblocktxn`
4. receive `blocktxn`
5. complete reconstruction and accept block

Useful references:

- [BIP152](https://github.com/bitcoin/bips/blob/master/bip-0152.mediawiki)
- [`test/functional/p2p_compactblocks.py`](https://github.com/danielabrozzoni/b4os-bitcoin/blob/master/test/functional/p2p_compactblocks.py)
- [`test/functional/test_framework/p2p.py`](https://github.com/danielabrozzoni/b4os-bitcoin/blob/master/test/functional/test_framework/p2p.py)
- [`test/functional/test_framework/messages.py`](https://github.com/danielabrozzoni/b4os-bitcoin/blob/master/test/functional/test_framework/messages.py)

## What You Will Build

Create a functional test that:

1. Builds a block that contains at least one non-coinbase transaction that `self.nodes[0]` doesn't know about.
2. Announces that block to `self.nodes[0]` using a custom P2P peer (subclass `P2PInterface`) and a `cmpctblock` message.
3. Ensures the node cannot fully reconstruct the block from mempool data and responds with `getblocktxn`.
4. Sends back a `blocktxn` message
5. Asserts that the node accepts the block and updates its tip to that block hash.
