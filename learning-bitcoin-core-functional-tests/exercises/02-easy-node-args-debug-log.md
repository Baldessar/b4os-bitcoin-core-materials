# Easy 02: Node Arguments + Debug Log with Prune Mode

Welcome! This exercise is about restarting nodes with startup arguments and checking logs.

## Concepts You Need

This exercise teaches you how to check node logs. While running, Bitcoin Core prints various logs, and checking them helps confirm whether the code path you expect is actually being executed.

The test framework node class `TestNode` provides the function `assert_debug_log(expected_msgs=[], unexpected_msgs=[])`, which checks whether certain strings are present or absent in node logs.

We usually pair `assert_debug_log(...)` with a Python `with` block. The `with` block means "apply this helper only to the indented code block."

So with `assert_debug_log(...)`, the flow is:

1. Start watching logs.
2. Run the action inside the block.
3. When the block ends, verify that expected messages were seen.

You will see this `with` + `assert_debug_log` pattern often in functional tests:

```python
# Make sure that 'Hello world' is in the debug log
with self.nodes[0].assert_debug_log(["Hello world"]):
    # action that should produce the log message
    # ...

# Make sure that 'I like donuts' is not in the debug log
with self.nodes[0].assert_debug_log([], unexpected_msgs=["I like donuts"]):
    # action that shouldn't produce the log message
    # ...
```

In this exercise, you will restart the node with specific startup flags and check whether specific log messages are emitted.

You will use prune mode, which means the node will not keep the entire blockchain on disk, only the last N MiB (configured by the user).

When prune mode is enabled, the Bitcoin Core node will log:

```
Prune configured to target <N> MiB on disk for block and undo files.
```

You can also check whether prune mode is enabled through `getblockchaininfo()`, where `info["pruned"]` is set to `True`.

To restart a node, use `self.restart_node`. The method takes the index of the node to restart and a list of extra args to pass on restart. For example, to restart node 0 (`self.nodes[0]`) with the startup argument `-txindex`:

```
self.restart_node(0, extra_args=["-txindex"])
```

## What You Will Build

Create a functional test that:

1. Uses one node.
2. Restarts with default args.
3. Asserts that debug logs do not show prune mode configuration.
4. Asserts that pruning is not enabled through RPC.
5. Restarts with pruning enabled (for example, `-prune=550`).
6. Asserts debug logs show prune mode configuration.
7. Asserts pruning is enabled through RPC.

Suggested filename in `b4os-bitcoin/`:

- `test/functional/feature_prune_debug_log.py`

Do not forget to add the test to [`test/functional/test_runner.py`](https://github.com/danielabrozzoni/b4os-bitcoin/blob/master/test/functional/test_runner.py).

## Hints

### Step 1 - Set up test params

<details>
<summary>Small hint</summary>
Set `self.num_nodes = 1`.
</details>

### Step 2 - Restart with default args

<details>
<summary>Small hint</summary>

Use `restart_node` with `extra_args=[""]`. This restarts the node without any additional configuration.
</details>

### Step 3 - Check debug logs

<details>
<summary>Small hint</summary>

You will need to wrap the restart call with `with assert_debug_log():`.
</details>

<details>
<summary>Small hint</summary>
Make sure the node did not print pruning logs on restart.
</details>

<details>
<summary>Big hint</summary>

```
with self.nodes[0].assert_debug_log([], unexpected_msgs=['Prune configured to target']):
```
</details>

### Step 4 - Check with RPC

<details>
<summary>Small hint</summary>
Call `getblockchaininfo` and assert `pruned` is `False`.
</details>

### Step 5, 6, 7

Same hints as steps 2, 3, and 4 :)

## Next Up

Continue with [`03-easy-miniwallet-support.md`](./03-easy-miniwallet-support.md).
