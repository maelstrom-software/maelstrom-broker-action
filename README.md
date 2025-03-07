![Maelstrom Logo (Dark Compatible)](https://github.com/maelstrom-software/maelstrom/assets/146376379/7b46a1c1-e67f-412a-b618-42f7e2c25139)

# Maelstrom Broker GitHub Action

This GitHub action installs and runs the Maelstrom broker.
[Maelstrom](https://github.com/maelstrom-software/maelstrom) is a suite of
tools for running tests in isolated micro-containers locally on your machine or
distributed across arbitrarily large clusters. Maelstrom currently has test
runners for Rust, Go, and Python, with more on the way. For more information
about Maelstrom in general, please see the [main
repository](https://github.com/maelstrom-software/maelstrom), the
[web site](https://maelstrom-software.com), or the
[documentation](https://maelstrom-software.com/doc/book/latest).

# Usage

```yml
jobs:
  run-tests:
    name: Run Tests
    runs-on: ubuntu-24.04

    steps:
    - name: Install and Run Maelstrom Broker
      uses: maelstrom-software/maelstrom-broker-action@v1

    - name: Check Out Repository
      uses: actions/checkout@v4

    - name: Install cargo-maelstrom
      uses: maelstrom-software/maelstrom-cargo-maelstrom-action@v1

    - name: Run Tests
      run: cargo maelstrom
```

# How to Use

Run multiple Maelstrom worker jobs in parallel with Maelstrom test runners,
like `cargo-maelstrom`, `maelstrom-go-test`, or `maelstrom-pytest`. See [this
example workflow](https://github.com/maelstrom-software/maelstrom-examples/blob/main/.github/workflows/ci-base.yml).

# What it Does

This action installs
[`maelstrom-worker`](https://maelstrom-software.com/doc/book/latest/worker.html),
sets a [sysctl to allow unprivileged user
namespaces](https://ubuntu.com/blog/ubuntu-23-10-restricted-unprivileged-user-namespaces),
then runs `maelstrom-worker` until completion.

The worker will connect to the broker, execute tests as long as the broker
provides them, and then exit when the broker itself exits.

# Learn More

You can find documentation for running Maelstrom in GitHub
[here](https://maelstrom-software.com/doc/book/latest/github.html).

# Licensing

This project is available under the terms of either the Apache 2.0 license or the MIT license.
