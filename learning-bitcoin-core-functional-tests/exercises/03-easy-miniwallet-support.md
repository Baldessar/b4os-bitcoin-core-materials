# Easy 03: Use MiniWallet as a Test Helper

Welcome! This exercise introduces MiniWallet as a practical helper for building test scenarios without using the Core wallet :)

## Concepts You Need

This exercise introduces MiniWallet as lightweight test support.

MiniWallet is not the full wallet RPC interface used in the [first exercise](./01-easy-wallet-rpc-basics.md). Instead, it is a helper in the functional test framework that makes it faster to create UTXOs and construct transactions for test scenarios. It is especially useful because many non-wallet tests still need spendable coins, and MiniWallet provides that without requiring that Bitcoin Core is compiled with wallet support.

The MiniWallet code is fairly short, and you can inspect its methods directly in [`test/functional/test_framework/wallet.py`](https://github.com/danielabrozzoni/b4os-bitcoin/blob/master/test/functional/test_framework/wallet.py).

A common pattern is:

```python
# Create a MiniWallet bound to node 0
wallet = MiniWallet(self.nodes[0])

# MiniWallet usually starts with mature UTXOs from the cached chain
assert_greater_than(wallet.get_balance(), 0)

# Build + broadcast a simple self-transfer transaction
tx = wallet.send_self_transfer(from_node=self.nodes[0])

# If needed, you can also create txs without broadcasting:
# tx = wallet.create_self_transfer()
# ...do stuff with tx...
# self.nodes[0].sendrawtransaction(tx["hex"])
```

## What You Will Build

Create a functional test that:

1. Uses one node.
2. Creates a `MiniWallet` connected to `self.nodes[0]`.
3. The MiniWallet starts with funds by default. Assert that its balance is greater than `0`.
4. Creates and broadcasts one self-transfer using MiniWallet.
5. Checks that the transaction is in the mempool.
6. Mines one block and checks that the transaction leaves the mempool.

Suggested filename in `b4os-bitcoin/`:

- `test/functional/feature_miniwallet.py`

Do not forget to add the test to [`test/functional/test_runner.py`](https://github.com/danielabrozzoni/b4os-bitcoin/blob/master/test/functional/test_runner.py) so you can run it through the harness.

## Hints

### Step 2 - Create MiniWallet and fund it

<details>
<summary>Big hint</summary>

Construct MiniWallet with `MiniWallet(self.nodes[0])`.
</details>

### Step 3 - Check balance

<details>
<summary>Small hint</summary>

Use `assert_greater_than` and MiniWallet's `get_balance()` method.
</details>

### Step 4 - Create and broadcast a self-transfer

<details>
<summary>Small hint</summary>
MiniWallet has a self-transfer helper that creates and sends the tx.
</details>

<details>
<summary>Big hint</summary>
Use `send_self_transfer`, and keep the returned transaction identifier so you can track its lifecycle.
</details>

### Step 5/6 - Assert mempool behavior

<details>
<summary>Small hint</summary>
Check that the transaction is in `getrawmempool()` before mining, then mine one block and check it is no longer in mempool.
</details>

## Next Up

Continue with [`04-easy-p2p-ping-pong.md`](./04-easy-p2p-ping-pong.md).
