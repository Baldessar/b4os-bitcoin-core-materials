# Easy 01: Basic Wallet RPC Flow

Welcome! In this exercise, you will learn how to use the wallet RPC interface.

## Concepts You Need

In this exercise, you will use the Bitcoin Core wallet RPC interface.

If you have never used wallet RPCs before, it can help to spin up your own node, run `bitcoin-cli help | less`, and browse the wallet and rawtransaction commands.

### Wallet module checks in tests

To run wallet tests, Bitcoin Core must be configured with wallet support. If you followed `SETUP.md`, this is already the case. Even so, we still want to skip the test when Bitcoin Core is compiled without wallet support; otherwise, someone using a no-wallet build would see a bunch of test failures.

For this reason, when working on wallet tests, call `self.skip_if_no_wallet()` in `skip_test_if_missing_module()`.

As an optional experiment, remove that skip in a wallet test and run it in an environment without wallet support to see the failure mode. To compile Bitcoin Core without wallet support, add `-DENABLE_WALLET=OFF` during configuration. Remember to compile Bitcoin Core with wallet support again afterward.

### Default wallet vs explicitly created wallets

When `skip_if_no_wallet()` is used, the framework enables wallet usage for the test and creates/loads a default wallet per node.

However, you can still create wallets manually, and that is exactly what you will do in this exercise. Use the `createwallet` RPC to create one, then `get_wallet_rpc` to get the wallet object you will work with.

```python
self.nodes[0].createwallet("node0_wallet")
node0_wallet = self.nodes[0].get_wallet_rpc("node0_wallet")
```

If only one wallet is loaded, you can call wallet RPCs directly on `self.nodes[i]`. However, when multiple wallets are loaded, those commands are ambiguous, so you should use `get_wallet_rpc(...)`:

```python
self.nodes[0].getbalance() # Only one wallet is loaded, so this is OK
self.nodes[0].createwallet("node0_wallet")
self.nodes[0].getbalance() # Oops! This raises an error because multiple wallets are loaded.
node0_wallet = self.nodes[0].get_wallet_rpc("node0_wallet")
default_wallet = self.nodes[0].get_wallet_rpc(self.default_wallet_name)
node0_wallet.getbalance() # This is OK!
default_wallet.getbalance() # This is OK!
```

### Wallet RPCs you will use

For this exercise, you will call the following RPCs:

- `getnewaddress`
- `getbalance`
- One sending RPC (`sendtoaddress` is a simple option)

If you have never used them, you can either spin up a node and check the help output, or inspect it directly from the test framework, for example:

```python
self.log.info(self.nodes[1].help("send"))
```

### Mining blocks in framework tests

To mine blocks, use the framework helpers. There are several (`self.generate`, `self.generateblock`, `self.generatetoaddress`, `self.generatetodescriptor`), and the most common is `self.generate`. You *could* use the node RPC `generatetoaddress`, but it is better to use framework helpers because they ensure all nodes in the framework receive the generated block before continuing.

```python
# Let's create 10 blocks!
# self.nodes[0] mines 10 blocks, and the reward is sent to self.nodes[0]'s default wallet
self.generate(self.nodes[0], 10)
# self.nodes[0] mines 15 blocks, but sends the reward to a specific address
address = "tb1..."
self.generatetoaddress(self.nodes[0], 15, address)
```
### Coinbase maturity

Coinbase outputs are not spendable immediately. They need 100 confirmations, as specified in [`COINBASE_MATURITY`](https://github.com/danielabrozzoni/b4os-bitcoin/blob/57a1a392c669bcab0c6a3218f0129ef7c7062547/src/consensus/consensus.h#L19).

For this reason, if you mine one block to your wallet, your balance will not increase by `50` BTC right away. You need to mine `101` blocks to get one mature coinbase output.

### Syncing mempools

With multiple nodes, relay is asynchronous. A transaction can appear on one node slightly before another.

Call `self.sync_mempools()` before cross-node mempool assertions.

### Assertions in the framework

Prefer framework helpers over bare `assert` when possible:

- `assert_equal(a, b)`
- `assert_greater_than(a, b)`

Definitions:

- [`assert_equal`](https://github.com/danielabrozzoni/b4os-bitcoin/blob/master/test/functional/test_framework/util.py#L75)
- [`assert_greater_than`](https://github.com/danielabrozzoni/b4os-bitcoin/blob/master/test/functional/test_framework/util.py#L87)

These helpers give clearer failures and are standard in Core tests.

### Asserting with `Decimal`

Wallet balances are decimal values. Use `Decimal` in assertions to avoid float precision issues.

```python
from decimal import Decimal
from test_framework.util import assert_equal

assert_equal(node1_wallet.getbalance(), Decimal("1.00000000"))
```

## What You Will Build

Create a functional test that:

1. Uses two nodes.
2. Creates one wallet on each node and gets the RPC handle for each one.
3. Funds node 0 wallet by generating blocks to it. How many blocks should be generated to have 50 spendable BTC in the wallet? :)
4. Checks that node 0 wallet balance is `50` BTC.
5. Sends `1` BTC from node 0 wallet to a node 1 wallet address (unconfirmed at this step).
6. Checks that the transaction is in both nodes' mempools.
7. Checks that node 0 wallet balance is now less than `49` BTC (because of amount + fee).
8. Mines one more block to node 0 wallet to confirm the transaction.
9. Checks that the transaction leaves the mempool.
10. Checks that node 1 wallet balance is `1` BTC.

Suggested filename in `b4os-bitcoin/`:

- `test/functional/wallet_rpc_basics.py`

Do not forget to add the test to [`test/functional/test_runner.py`](https://github.com/danielabrozzoni/b4os-bitcoin/blob/master/test/functional/test_runner.py) so you can run it through the harness.

## Hints

### Step 1 - Set up test params and wallet checks

<details>
<summary>Big hint</summary>

Set `self.num_nodes = 2` in `set_test_params()`, and call `self.skip_if_no_wallet()` in `skip_test_if_missing_module()`.
</details>

### Step 2 - Create wallets and RPC handles

<details>
<summary>Small hint</summary>

Use `createwallet` on each node and then call `get_wallet_rpc` for each created wallet.
</details>

<details>
<summary>Medium hint</summary>

```python
self.nodes[0].createwallet("node0_wallet")
node0_wallet = self.nodes[0].get_wallet_rpc("node0_wallet")

self.nodes[1].createwallet("node1_wallet")
node1_wallet = self.nodes[1].get_wallet_rpc("node1_wallet")
```
</details>

### Step 3 - Mine spendable coins

<details>
<summary>Small hint</summary>

Use the framework's `generate` method, passing `self.nodes[0]` as the `generator`. Mine 101 blocks so you have 100 immature 50 BTC outputs plus one mature 50 BTC output you can use.
</details>

<details>
<summary>Big hint</summary>

```python
self.generate(self.nodes[0], 101)
```
</details>

### Step 4 - Check node0 balance

<details>
<summary>Troubleshooting hint</summary>

When asserting, make sure to use `Decimal` to avoid floating point errors.

```python
assert_equal(node0_wallet.getbalance(), Decimal("50.00000000"))
```
</details>

### Step 5 - Send 1 BTC to node1

<details>
<summary>Small hint</summary>

Use either the `send` or the `sendtoaddress` RPC calls. If you've never used them, log and inspect the output of

```python
self.nodes[1].help("send")
# or
self.nodes[1].help("sendtoaddress")
```
</details>

### Step 6 - Check mempool

<details>
<summary>Troubleshoot hint</summary>

Do you see the transaction in both nodes' mempools? If not, make sure the nodes are synced.
</details>

<details>
<summary>Big hint</summary>

Save the txid from `sendtoaddress`, call `self.sync_mempools()`, and check that the txid is present in both nodes' `getrawmempool()`.
</details>

### Step 7 - Check node0 balance

<details>
<summary>Troubleshoot hint</summary>

Did you use `Decimal` for balance checks? :)

You do not need to assert the exact node 0 balance here. It is enough to assert that it is less than `49` BTC (accounting for the `1` BTC sent plus fee).
</details>

### Step 8 - Mine one more block

See hints in step 3.

### Step 9 - Check that the transaction leaves the mempool

See hints in step 6.

### Step 10 - Check balance on node 1

See hints in step 4 and in the `Decimal` section above.

## Next Up

Continue with [`02-easy-node-args-debug-log.md`](./02-easy-node-args-debug-log.md).
