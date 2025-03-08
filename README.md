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
    # The architecture needs to be the same as the workers.
    runs-on: ubuntu-24.04

    steps:
      # The broker needs to be installed and started before running the
      # cargo-maelstrom-action or the maelstrom-go-test-action. The broker must
      # be run in the same job as the test runners (cargo-maelstrom or
      # maelstrom-go-test).
    - name: Install Maelstrom Broker
      uses: maelstrom-software/maelstrom-broker-action@v1

      # Start the broker as a background process in this job.
    - name: Start Maelstrom Broker
      run: |
        TEMPFILE=$(mktemp maelstrom-broker-stderr.XXXXXX)
        maelstrom-broker 2> >(tee "$TEMPFILE" >&2) &
        PID=$!
        PORT=$( \
          tail -f "$TEMPFILE" \
          | awk '/\<addr: / { print $0; exit}' \
          | sed -Ee 's/^.*\baddr: [^,]*:([0-9]+),.*$/\1/' \
        )
        echo "MAELSTROM_BROKER_PID=$PID" >> "$GITHUB_ENV"
        echo "MAELSTROM_BROKER_PORT=$PORT" >> "$GITHUB_ENV"
      env:
        MAELSTROM_BROKER_ARTIFACT_TRANSFER_STRATEGY: github

      # Schedule a post-job handler to kill the broker. Killing the broker with
      # signal 15 (SIGTERM) allows it to tell the workers to shut themselves
      # down.
    - name: Schedule Post Handler to Kill Maelstrom Broker
      uses: gacts/run-and-post-run@v1
      with:
        post: kill -15 $MAELSTROM_BROKER_PID

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

  # These are the worker jobs. Tests will execute on one of these workers.
  maelstrom-worker:
    strategy:
      matrix:
        worker-number: [1, 2, 3, 4]

    name: Maelstrom Worker ${{ matrix.worker-number }}

    # This must be the same architecture as the test-running job.
    runs-on: ubuntu-24.04

    steps:
    - name: Install and Run Maelstrom Worker
      uses: maelstrom-software/maelstrom-worker-action@v1
```

# How to Use

Run at the beginning of a job that will use Maelstrom test runner, 
like `cargo-maelstrom`, `maelstrom-go-test`, or `maelstrom-pytest`. Maelstrom
workers should be run on separate jobs. See [this
example workflow](https://github.com/maelstrom-software/maelstrom-examples/blob/main/.github/workflows/ci-base.yml).

# What it Does

This action installs
[`maelstrom-broker`](https://maelstrom-software.com/doc/book/latest/broker.html),
and exposes some environment variables needed for later actions.

Unfortunately, because of limitations with GitHub actions, this action doesn't
start the broker. You need to do that yourself, as shown in the example.
Additionally, you should set a post handler to kill the broker at the end of
the job, also show in the example.

The test runners need to be run in the same job, after starting the broker. The
example above illustrates this.

# Learn More

You can find documentation for running Maelstrom in GitHub
[here](https://maelstrom-software.com/doc/book/latest/github.html).

# Licensing

This project is available under the terms of either the Apache 2.0 license or the MIT license.
