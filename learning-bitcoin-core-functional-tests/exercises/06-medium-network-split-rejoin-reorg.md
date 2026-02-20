# Medium 06: Network Split, Rejoin, and Reorg

## Concepts You Need

Before writing the test logic, it helps to understand how to control topology and how Bitcoin Core chooses a winning tip.

### Disconnecting and reconnecting nodes in the framework

In Bitcoin Core functional tests, network links are controlled by the test framework itself. Use `self.disconnect_nodes(a, b)` to cut the link between two nodes, then use `self.connect_nodes(a, b)` to bring it back.

A useful detail: `disconnect_nodes(...)` does not just send one disconnect RPC and continue. It waits until both sides no longer report each other as peers. That makes your split deterministic and avoids flaky races.

```python
# Split node 0 and node 1
self.disconnect_nodes(0, 1)

# Later, reconnect them
self.connect_nodes(0, 1)
```

### Mining while nodes are disconnected: `self.generate(..., sync_fun=self.no_op)`

Framework `self.generate(...)` syncs all nodes by default after mining. That default is usually great, but it is wrong during intentional partitions.

If your nodes are disconnected, call `self.generate(node, n, sync_fun=self.no_op)` so the framework does not try to force global sync.

```python
# Mine independently while split
self.generate(self.nodes[0], 1, sync_fun=self.no_op)
self.generate(self.nodes[1], 1, sync_fun=self.no_op)
```

### Network split and competing tips

A network split means some nodes are temporarily disconnected from others, so information no longer flows across the whole network. Each disconnected group keeps operating on its own: it can receive transactions from its own peers, mine new blocks, and advance its own chain tip.

Because those groups are isolated, they can produce different valid blocks at the same height. That creates competing branches. Both branches may be valid; they just represent different histories after the split point.

### Verifying competing branches with `getchaintips`

`getbestblockhash` only shows the active tip. During split/rejoin tests, that is not enough context. `getchaintips` is better because it lists all known chain tips - both active, and side branches.

Use it to confirm both branches are known after reconnect, before the decisive next block is mined.

```python
def tip_hashes(node):
    return {tip["hash"] for tip in node.getchaintips()}

self.wait_until(lambda: tip0 in tip_hashes(self.nodes[0]) and tip1 in tip_hashes(self.nodes[0]))
self.wait_until(lambda: tip0 in tip_hashes(self.nodes[1]) and tip1 in tip_hashes(self.nodes[1]))
```

### What “deciding the tip” means

A node’s “tip” is the block at the end of the chain it currently considers best.
When people say a node “decides the tip,” they mean it selects which known branch should be the active chain.

This is a local decision made independently by each node, based on consensus and chain-selection rules. Nodes do not vote; they evaluate available branches and activate the best one.

### Why one fork wins: more work first, tie keeps first-seen branch

Bitcoin Core selects the chain with the most total accumulated work (`nChainWork`). If one branch has more work, nodes switch to it.

When work is equal, tie-breaking uses block arrival ordering (`nSequenceId`). In practice, this means the node sticks with the branch it can activate first (commonly the one it saw first).

From [`src/node/blockstorage.cpp`](https://github.com/danielabrozzoni/b4os-bitcoin/blob/master/src/node/blockstorage.cpp):

```cpp
bool CBlockIndexWorkComparator::operator()(const CBlockIndex* pa, const CBlockIndex* pb) const
{
    // First sort by most total work, ...
    if (pa->nChainWork > pb->nChainWork) return false;
    if (pa->nChainWork < pb->nChainWork) return true;

    // ... then by earliest activatable time, ...
    if (pa->nSequenceId < pb->nSequenceId) return false;
    if (pa->nSequenceId > pb->nSequenceId) return true;

    ...
}
```

And `nSequenceId` is documented as receive order in [`src/chain.h`](https://github.com/danielabrozzoni/b4os-bitcoin/blob/master/src/chain.h):

```cpp
//! (memory only) Sequential id assigned to distinguish order in which blocks are received.
int32_t nSequenceId{SEQ_ID_INIT_FROM_DISK};
```

So for this exercise: after reconnect, equal-work branches can coexist temporarily. A node does not switch just because it learns about another equal-work tip. Once one side mines the next block, that side has more work, and both nodes converge.

## What You Will Build

Create a functional test that:

1. Uses two nodes.
2. Creates two conflicting transactions, `tx0` and `tx1`. Both transactions spend the same input, so they cannot coexist in the blockchain.
3. Disconnects node 0 and node 1.
4. Broadcasts `tx0` from node 0 and `tx1` from node 1.
5. Checks that `tx0` is in node 0's mempool and `tx1` is in node 1's mempool, and that neither transaction is in the other node's mempool.
6. Mines one block on node 0 and one block on node 1 while disconnected.
7. Verifies that the tips are different.
8. Checks that `tx0` is confirmed on node 0 and `tx1` is confirmed on node 1.
9. Reconnects node 0 and node 1.
10. Uses `getchaintips` to check that both nodes see both competing blocks.
11. Uses `getbestblockhash` to check that neither node has switched tips yet.
12. Mines one block with node 0; this block builds on top of `tip0` and breaks the tie.
13. Asserts that both nodes now have the same tip.
14. Tries broadcasting `tx1` again and asserts that it raises an RPC error because the input is already spent.

Suggested filename in `b4os-bitcoin/`:

- `test/functional/feature_split_rejoin_reorg.py`

Do not forget to add the test to [`test/functional/test_runner.py`](https://github.com/danielabrozzoni/b4os-bitcoin/blob/master/test/functional/test_runner.py) so you can run it through the harness.

## Hints

### Step 2 - Create conflicting transactions

<details>
<summary>Small hint</summary>

Using `MiniWallet` is the easiest path. Get one utxo and build two different spends from it.
</details>

<details>
<summary>Big hint</summary>

Use `get_utxo(mark_as_spent=False)`, then pass that same utxo to two `create_self_transfer(...)` calls.
</details>

### Step 6 - Mine while partitioned

<details>
<summary>Troubleshooting hint</summary>

As explained above, when nodes are disconnected, you can't simply do `self.generate(...)`, because it will try to wait until both nodes have the same tip, and timeout.

Use `sync_fun=self.no_op` while the nodes are disconnected.
</details>

### Step 8 - Verify each tx is confirmed on its own side

<details>
<summary>Medium hint</summary>

Use each node's mined block hash and inspect `getblock(block_hash)["tx"]`.
</details>

### Step 10 - Verify both competing tips are known

<details>
<summary>Big hint</summary>

After reconnect, wait until each node's `getchaintips()` contains both `tip0` and `tip1` hashes.
</details>

### Step 14 - Assert rebroadcast fails

<details>
<summary>Small hint</summary>

If you try broadcasting tx1, `sendrawtransaction` will error, because tx1 spends an input already spent. This is good! How can you make the test assert that `sendrawtransaction` fails? Is there a function that does that in the test framework utils?
</details>

<details>
<summary>Big hint</summary>

Use `assert_raises_rpc_error(...)`.
</details>
