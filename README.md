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
    # Both ubuntu-22.04 and ubuntu-24.04 are supported.
    # Both x86-64 and ARM (AArch64) are supported.
    # The architecture needs to be the same as the workers (but not the broker).
    runs-on: ubuntu-24.04

    steps:

    # This action installs and configures (via environment variables)
    # cargo-maelstrom so it can be run simply with `cargo maelstrom`.
    - name: Install and Configure cargo-maelstrom
      uses: maelstrom-software/cargo-maelstrom@v1

    # This action installs and configures (via environment variables)
    # maelstrom-go-test so it can be run simply with `maelstrom-go-test`.
    - name: Install and Configure maelstrom-go-test
      uses: maelstrom-software/maelstrom-go-test-action@v1

    - name: Check Out Repository
      uses: actions/checkout@v4

      # You can now run `cargo maelstrom` however you want.
    - name: Run Tests
      run: cargo maelstrom

      # You can now run `maelstrom-go-test` however you want.
    - name: Run Tests
      run: maelstrom-go-test

  # This is the broker job. It runs in parallel to the worker jobs and the #
  # client jobs.
  maelstrom-broker:
    name: Maelstrom Broker
    runs-on: ubuntu-24.04 # Can be ARM or AMD. It doesn't matter.
    steps:
    - name: Install and Run Maelstrom Broker
      uses: maelstrom-software/maelstrom-broker-action@v1

  # These are the worker jobs. Tests will execute on one of these workers.
  maelstrom-worker:
    strategy:
      matrix:
        worker-number: [1, 2, 3, 4]
    name: Maelstrom Worker ${{ matrix.worker-number }}
    runs-on: ubuntu-24.04 # Must be the same as the test-running job(s).
    steps:
    - name: Install and Run Maelstrom Worker
      uses: maelstrom-software/maelstrom-worker-action@v1

  # This job stops the cluster. It needs to depend on the test-running job(s).
  stop-maelstrom:
    runs-on: ubuntu-24.04 # Can be ARM or AMD. It doesn't matter.
    if: ${{ always() }}
    needs: [run-tests]
    steps:
    - name: Install and Configure maelstrom-go-test
      uses: maelstrom-software/maelstrom-admin-action@v1
    - name: Stop Maelstrom Cluster
      run: maelstrom-admin stop
```

# How to Use

Run on its own job in parallel with the Maelstrom workers and jobs that use
Maelstrom test runners, like `cargo-maelstrom`, `maelstrom-go-test`, or
`maelstrom-pytest`. Maelstrom workers should be run on separate jobs. See [this
example
workflow](https://github.com/maelstrom-software/maelstrom-examples/blob/main/.github/workflows/ci-base.yml).

After all test-running jobs are done, use `maelstrom-admin` to stop the job.
See
[`maelstrom-admin-action`](https://github.com/maelstrom-software/maelstrom-admin-action).

# What it Does

This action installs
[`maelstrom-broker`](https://maelstrom-software.com/doc/book/latest/broker.html),
then runs it until completion, either by being sent a signal or being told to
stop by `maelstrom-admin stop`.

Workers and test runners will connect to the broker. The broker will send
test-runners' tests to the workers for executio, then send the results back to
the test runners.

# Learn More

You can find documentation for running Maelstrom in GitHub
[here](https://maelstrom-software.com/doc/book/latest/github.html).

# Licensing

This project is available under the terms of either the Apache 2.0 license or the MIT license.
