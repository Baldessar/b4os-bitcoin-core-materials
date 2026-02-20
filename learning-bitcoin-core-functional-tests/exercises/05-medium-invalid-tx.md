# Exercise 05 (Medium): Submitting an Invalid Transaction Over P2P

Let's step into a more realistic P2P validation scenario where multiple tools come together.

## Concepts You Need

The typical transaction relay pattern is:

1. A peer announces transaction hashes with `INV`.
2. Another peer that wants the transaction requests it with `GETDATA`.
3. The announcing peer replies with `TX` (if found) or `NOTFOUND` (if missing).

However, it is also valid to send a `TX` message directly (unsolicited). Modern peers usually relay via `INV`/`GETDATA`, but unsolicited `TX` is still accepted by the protocol and will be processed by normal mempool/validation logic.

## What You Will Build

Create a functional test that:

1. Creates an invalid transaction - there are various ways to do this, so try `git grep bad-txns` in Bitcoin Core if you need inspiration :)
2. Submits the invalid transaction through a P2P message to `self.nodes[0]`. You can use the `TX` message unsolicited instead of `INV` + `GETDATA`.
3. Checks that `self.nodes[0]` logged that the transaction is invalid (you can look for `"bad-txns"` in the logs).
4. Checks that the transaction is not in `self.nodes[0]` mempool.
5. Asks `self.nodes[0]` for the transaction through a `GETDATA` message.
6. Checks that `self.nodes[0]` replies with a `NOTFOUND` message.

Suggested filename in `b4os-bitcoin/`:

- `test/functional/p2p_invalid_submission.py`

Do not forget to add the test to [`test/functional/test_runner.py`](https://github.com/danielabrozzoni/b4os-bitcoin/blob/master/test/functional/test_runner.py) so you can run it through the harness.

When you complete this exercise, submit your work as a PR in your fork of [`b4os-bitcoin`](https://github.com/danielabrozzoni/b4os-bitcoin).

## Hints

### Step 1 - Create an invalid transaction

<details>
<summary>Small hint</summary>
To create the transaction, you can either use Bitcoin Core wallet RPCs or MiniWallet, whichever you prefer.
One possible approach is to create a normal, valid transaction first and then make it invalid. How would you do that? There are many ways. Try `git grep bad-txns` for ideas.
</details>

<details>
<summary>Medium hint</summary>
Too many possible errors and unsure which one to pick? Modify your transaction so it spends an input that does not exist.
</details>

<details>
<summary>Big hint</summary>
For example, change your first input's outpoint to a non-existent txid:vout.
```python
tx = miniwallet.create_self_transfer(...)['tx']
tx.vin[0].prevout.n = 15
```
</details>

### Step 2 - Submit the transaction through P2P

<details>
<summary>Small hint</summary>

We want the transaction to be submitted through P2P, so you will need to use a `P2PInterface` and manually construct the needed message.
</details>

<details>
<summary>Small hint</summary>

For this step, you *could* send an `INV`, wait for `GETDATA`, and then send a `TX` message. Or, you can send a `TX` message directly.
If you need inspiration for building and sending a `TX` P2P message, search in-tree for `msg_tx` usage.
</details>

<details>
<summary>Big hint</summary>

`msg_tx` wraps a `CTransaction`. You can construct the `CTransaction` path first, then wrap and send it.
</details>

### Step 3 - Check logs for invalidation path

<details>
<summary>Big hint</summary>

Use `assert_debug_log(...)` around the send path so you assert on the validation reason, not only final mempool state.
</details>

### Step 4 - Check mempool rejection

<details>
<summary>Small hint</summary>

Is there an RPC that can help you check whether a tx is in the mempool? Or that might return the entire mempool for you to check?
</details>

<details>
<summary>Big hint</summary>

`getrawmempool` is the easiest way to assert the invalid transaction was not accepted.
</details>

### Step 5 - Request the transaction via GETDATA

<details>
<summary>Small hint</summary>

You need to construct a `msg_getdata`, and then put a `CInv` inside `message.inv`. `CInv` can use either `txid` or `wtxid`; both will work.
</details>

### Step 6 - Verify NOTFOUND response

<details>
<summary>Small hint</summary>

`P2PInterface` sent a `GETDATA` message to `self.nodes[0]` in step 5. Is there a way to check whether `self.nodes[0]` replied with a message? Is this information available in `P2PInterface`?
</details>

<details>
<summary>Medium hint</summary>

The `notfound` response is asynchronous. Use `wait_until` with a timeout instead of an immediate assertion.
</details>

<details>
<summary>Big hint</summary>

`P2PInterface.last_message` tracks the latest message by type. Wait for `notfound`, then inspect the announced inventory item.
</details>
