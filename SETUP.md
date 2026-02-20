# Workshop Setup

Run this setup before each workshop.

There are two repositories we will work on:

1. This repository (`b4os-bitcoin-core-materials`) for workshop instructions. You do not need to clone it for coding work; think of it as read-only workshop material (unless you spot something to improve :) ).
2. [`b4os-bitcoin`](https://github.com/danielabrozzoni/b4os-bitcoin) for Bitcoin Core code, coding exercises, and PR submissions.

## 0. Fork and clone `b4os-bitcoin`

Fork [`b4os-bitcoin`](https://github.com/danielabrozzoni/b4os-bitcoin) on GitHub, then clone your fork:

```bash
git clone https://github.com/<your_github_username>/b4os-bitcoin
```

`b4os-bitcoin` is not the upstream Bitcoin Core repository, so you can safely push exercise branches and open PRs there.

## 1. Build Bitcoin Core (pick your platform doc)

In `b4os-bitcoin/`, compile Bitcoin Core by following the build doc for your platform:

- Unix/Linux: [build-unix.md](https://github.com/danielabrozzoni/b4os-bitcoin/blob/master/doc/build-unix.md)
- macOS: [build-osx.md](https://github.com/danielabrozzoni/b4os-bitcoin/blob/master/doc/build-osx.md)
- Windows (mingw): [build-windows.md](https://github.com/danielabrozzoni/b4os-bitcoin/blob/master/doc/build-windows.md)
- Windows (MSVC): [build-windows-msvc.md](https://github.com/danielabrozzoni/b4os-bitcoin/blob/master/doc/build-windows-msvc.md)
- FreeBSD: [build-freebsd.md](https://github.com/danielabrozzoni/b4os-bitcoin/blob/master/doc/build-freebsd.md)
- NetBSD: [build-netbsd.md](https://github.com/danielabrozzoni/b4os-bitcoin/blob/master/doc/build-netbsd.md)
- OpenBSD: [build-openbsd.md](https://github.com/danielabrozzoni/b4os-bitcoin/blob/master/doc/build-openbsd.md)

## 2. Sanity Check the Functional Test Framework

Run a test once to confirm Python deps + binaries are usable:

```bash
build/test/functional/test_runner.py example_test.py
```

## 3. Ready for Workshop

After this passes, go to:

- [`learning-bitcoin-core-functional-tests/`](./learning-bitcoin-core-functional-tests/) for test-writing exercises
- [`practicing-bitcoin-core-code-review/`](./practicing-bitcoin-core-code-review/) for review exercises
