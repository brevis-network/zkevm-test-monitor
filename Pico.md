# Pico RV64IM RISCOF Testing Guide

This guide covers how to run RISCOF RISC-V compliance tests for pico with RV64IM support.

## Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                        AWS / Local                          │
│                                                             │
│  ┌─────────────────┐    ┌─────────────────────────────┐   │
│  │   brevis-vm     │    │     zkevm-test-monitor      │   │
│  │                 │    │                             │   │
│  │ cargo build     │───>│ binaries/pico-binary         │   │
│  │                 │    │         │                     │   │
│  │                 │    │         ▼                     │   │
│  │                 │    │ riskof:latest (Docker)       │   │
│  │                 │    │   ├── riscv-arch-test       │   │
│  │                 │    │   ├── sail-riscv (ref)      │   │
│  │                 │    │   └── pico (DUT)            │   │
│  └─────────────────┘    └─────────────────────────────┘   │
└─────────────────────────────────────────────────────────────┘
```

## Step 1: Setup zkevm-test-monitor

### 1.1 Clone

```bash
git clone -b pico-rv64 https://github.com/brevis-network/zkevm-test-monitor.git
```

## Step 2: Build RISCOF Docker Image

### 2.1 Build the Docker image

```bash
cd zkevm-test-monitor/riscof
docker build -t riscof:latest .
```

### 2.2 Verify the image exists

```bash
docker images | grep riscof
```

## Step 3: Setup brevis-vm

### 3.1 Clone

```bash
git clone -b cpu-u64-emu-test https://github.com/brevis-network/brevis-vm.git
```

### 3.2 Build pico-cli

```bash
cd brevis-vm
cargo build --release -p pico-cli
```

The binary will be at: `target/release/cargo-pico`

### 3.3 Copy pico binary

```bash
mkdir -p zkevm-test-monitor/binaries
cp brevis-vm/target/release/cargo-pico zkevm-test-monitor/binaries/pico-binary
chmod +x binaries/pico-binary
```

## Step 4: Run Tests

### 4.1 Run all tests

```bash
cd zkevm-test-monitor
./src/test.sh --arch pico
```

### 4.2 View results

```bash
test-results/pico
```

## Understanding Test Results

### Test Output Format

```
ERROR | /riscof/riscv-arch-test/.../add-01.S : <commit-hash> : Failed
```

- `<commit-hash>` - Git commit of the test
- `Failed` - Signature mismatch between pico and Sail reference

### Expected Behavior

Currently, **all tests should fail** because:
1. The emulator stub accepts RV64IM but truncates addresses to u32
2. RV64 instructions are not fully implemented

This is expected for TDD (Test-Driven Development).

### Test Suites

| Suite | Description |
|-------|-------------|
| `rv64i_m/I` | Base integer instructions (RV64I) |
| `rv64i_m/M` | Multiply/Divide instructions (RV64M) |
| `rv64i_m/A` | Atomic instructions (RV64A) |
| `rv64i_m/F` | Single-precision float (RV64F) |
| `rv64i_m/D` | Double-precision float (RV64D) |

## References

- [RISCOF Documentation](https://riscof.readthedocs.io/)
- [riscv-arch-test](https://github.com/riscv-non-isa/riscv-arch-test)
- [zkevm-test-monitor](https://github.com/eth-act/zkevm-test-monitor)
