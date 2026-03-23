# Network File System (NFS)

A distributed Network File System in C with a central Naming Server, multiple Storage Servers, and a command-line Client.

This project provides path-based routing for file operations, storage-server registration, basic replication workflows, and concurrent client handling.

## Overview

The system is split into three main components:

- Naming Server: global namespace, routing, command coordination, health monitoring
- Storage Servers: actual file and directory data operations
- Client: command-line interface that talks to naming server and storage servers

High-level flow:

1. Storage servers start and register with the naming server.
2. Naming server indexes paths using a trie and tracks server metadata.
3. Client sends operations to naming server.
4. Naming server resolves destination storage server and forwards/coordinates operation.

## Key Features

- Trie-based path indexing for storage lookup
- LRU cache for faster repeated server resolution
- File operations: create, delete, read, write, info, copy
- Directory copy support via archive transfer
- Streaming support for audio files via client integration
- Multi-threaded server handling using pthreads
- Storage server health checking and recovery logging
- Backup synchronization hooks on storage side

## Repository Structure

- Clients: client program and request handling
- NamingServer: naming server core, trie, LRU, logging, command handling
- StorageServer: storage server command execution, file transfer, async write logic
- Utils: shared headers, constants, types, and error utilities
- Test: sample storage directories and test artifacts

## Build

This project is intended for Linux-like environments.

Build all binaries:

```bash
make all
```

Produced binaries:

- naming_server
- storage_server
- client

Clean build outputs:

```bash
make clean
```

## Run

Start components in this order from repository root:

1. Naming server

```bash
./naming_server
```

2. One or more storage servers (port + storage path)

```bash
./storage_server 8080 Test/SS1
./storage_server 7070 Test/SS2
```

3. Client

```bash
./client
```

You can also use `make.sh` as a convenience launcher in desktop Linux environments with terminal support.

## Client Commands

Supported command families in the current implementation:

- `READ <path>`
- `WRITE <path> <content> [SYNC]`
- `INFO <path>`
- `STREAM <path>`
- `CREATE <parent_path> <new_path> <0|1>`
    - `0` for file, `1` for directory
- `DELETE <parent_path> <target_path> <0|1>`
- `COPY <source_path> <destination_path> <0|1>`
    - `0` for file copy, `1` for directory copy
- `LIST`
- `exit`

Notes:

- Exact command parsing is token-based and space-sensitive.
- For `WRITE`, content is tokenized by spaces in the current parser.

## Configuration

Naming server address is configured in `Utils/constants.h`.

Default values include:

- Naming server IP (`NM_IP`)
- Naming server port (`NM_PORT`)

Update these values before running in a different network environment.

## Performance Characteristics

- O(1) expected cache lookup (hash-backed LRU)
- O(path length) trie insert and lookup
- Threaded request handling on naming and storage servers

## Assumptions

1. Duplicate path ownership across storage servers is not allowed.
2. Paths are expected in project-compatible format (for example, avoid `./`-prefixed paths).
3. Storage directories are accessible on the host where each storage server runs.

## Current Limitations

- Command parsing for some operations is strict and can be brittle.
- Write content tokenization is limited for multi-word payloads.
- Some backup and replication flows are implementation-heavy and environment-dependent.
- Linux shell tools (for example, `tar`, `rm`, `cp`) are used in parts of the implementation.

## Development Notes

- Code is compiled with debug and AddressSanitizer flags from the provided makefile.
- Logs are generated for naming server activity and server health status.

## Contributors

Contributions are welcome. If you plan to extend the system, useful next steps are:

- robust command parser and protocol framing
- stronger replication semantics
- better integration tests for multi-server workflows