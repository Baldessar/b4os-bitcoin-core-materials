# Easy 04: Send a Ping and Verify Pong

Welcome! In this exercise, you will use a small P2P scenario to get comfortable with asynchronous message testing :)

## Concepts You Need

Before doing the exercise, it helps to understand how the P2P test tools work on their own.

### The P2P Interface and P2P messages

In Bitcoin Core functional tests, `P2PInterface` is the programmable test peer. You connect it to a node with `add_p2p_connection(...)`, then send protocol messages (`msg_*` objects) and inspect what the node sends back.

`P2PInterface` keeps internal state such as message counters and `last_message`, which stores the latest received message for each message type. If you need custom behavior, you can subclass `P2PInterface` and override `on_*` handlers. See [`example_test.py`](https://github.com/danielabrozzoni/b4os-bitcoin/blob/master/test/functional/example_test.py) with the `BaseNode` class for more info.

### Bitcoin ping/pong

`ping` and `pong` are protocol-level liveness messages. A `ping` includes a nonce, and the peer replies with `pong` using that same nonce. The nonce lets the sender match each `pong` reply to the correct `ping` request, even when multiple pings are in flight.

### Sending messages from `P2PInterface`

`P2PInterface` gives you helpers for sending messages to the peer it is connected to. There are two helpers with different goals:

- `send_without_ping(message)`: sends only the message you provide and returns.
- `send_and_ping(message)`: sends your message and then does an extra ping/pong sync round. This ensures that the peer did receive and process the message, since nodes process messages in order.

You can read more in:

- [Using the P2P interface](https://github.com/danielabrozzoni/b4os-bitcoin/tree/master/test/functional#using-the-p2p-interface)

### Using `wait_until` in P2P tests

P2P messaging is asynchronous. Your test thread sends a message, but message handling runs in a network thread, and message processing is not immediate. Instead of doing immediate assertions right after send calls, we use `wait_until` to wait for the expected state:

```python
peer.send_without_ping(msg_getaddr())
peer.wait_until(lambda: "addr" in peer.last_message, timeout=5)
```

### Using `p2p_lock`

`P2PInterface` state is shared between threads, so direct reads/writes should be protected with `p2p_lock`.

Use the lock when you directly access internal P2P state in normal test code:

```python
from test_framework.p2p import p2p_lock

with p2p_lock:
    got_pong = "pong" in peer.last_message
```

If you use `peer.wait_until(...)`, it already checks your condition under `p2p_lock`, so you do not need to lock manually inside that condition.

## What You Will Build

Create a functional test that:

1. Uses one node.
2. Creates one P2P connection to `self.nodes[0]` using `P2PInterface`.
3. Uses `P2PInterface` to send a `msg_ping` message to the node.
4. Checks that a `pong` is received.
5. Checks that the `pong` nonce matches the `ping` nonce.

Suggested filename in `b4os-bitcoin/`:

- `test/functional/p2p_ping_pong.py`

Do not forget to add the test to [`test/functional/test_runner.py`](https://github.com/danielabrozzoni/b4os-bitcoin/blob/master/test/functional/test_runner.py) so you can run it through the harness.

## Hints

### Step 1 - Set up a minimal test

<details>
<summary>Big hint</summary>

Set `self.num_nodes = 1` in `set_test_params()`.
</details>

### Step 2 - Create a P2P connection

<details>
<summary>Small hint</summary>

Create a `P2PInterface` instance and attach it to `self.nodes[0]`.
</details>

### Step 3 - Send a ping message

<details>
<summary>Small hint</summary>

Import `msg_ping` from `test_framework.messages` and send it through your peer.
</details>

<details>
<summary>Big hint</summary>

Use `send_without_ping(msg_ping(nonce=your_nonce))`. Save `your_nonce` so you can compare it with the received `pong` nonce.
</details>

### Step 4 - Verify pong is received and nonce matches

<details>
<summary>Small hint</summary>

`P2PInterface.last_message` stores the latest message by type.
</details>

<details>
<summary>Medium hint</summary>

Use `wait_until` and check both:

- `"pong" in peer.last_message`
- `peer.last_message["pong"].nonce == your_nonce`
</details>

<details>
<summary>Troubleshooting hint</summary>

If nonces do not match, check whether `send_and_ping(...)` was used. Extra sync pings can replace `last_message["pong"]`.
</details>

## Next Up

Nice work, you completed the core easy path! Time to move on to medium exercises in [`learning-bitcoin-core-functional-tests/exercises/`](./).
