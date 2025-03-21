# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

## [1.1.0] - 2025-03-21
### Changes
- The action now runs the broker after installing it. As a result, the action
  must now be run on its own job.
- The action now uses a specific verson of `maelstrom-broker` (v0.14.0) so that
  it can cache the result.
- Updated to use the environment variables introduced in v0.14.0.
- Updated README.md to reflect new usage.

## [1.0.0] - 2025-03-07
### Added
- The initial version of this action.

[unreleased]: https://github.com/maelstrom-software/maelstrom-broker-action/compare/v1.1.0...HEAD
[1.1.0]: https://github.com/maelstrom-software/maelstrom-broker-action/compare/v1.0.0...v1.1.0
[1.0.0]: https://github.com/maelstrom-software/maelstrom-broker-action/releases/tag/v1.0.0
