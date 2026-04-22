# Dark Auction MP-SPDZ Project

## Overview

This project implements a **dark auction** using MP-SPDZ, a versatile framework for secure multi-party computation (MPC). In a dark/sealed-bid auction, bids are private and revealed only after all parties submit them.

## Project Structure

```
mpc-project/
├── Schedules/
│   └── dark-auction.sch      # MP-SPDZ schedule file (compiles Programs/dark-auction.mpc)
├── Scripts/
│   ├── run_auction.sh       # Main script: compile + run with multiple protocols
│   ├── run_mascot_dark_auction.sh  # Convenience wrapper for MASCOT protocol
│   ├── test_run.sh          # Quick test with per-party output logs
│   └── generate_inputs.py   # Generates/converts input files
├── Inputs/
│   ├── party0.txt          # Human-readable inputs for party 0
│   ├── party1.txt          # Human-readable inputs for party 1
│   ├── party2.txt          # Human-readable inputs for party 2
│   ├── Input-P0-0          # MP-SPDZ binary format inputs
│   ├── Input-P1-0
│   └── Input-P2-0
├── Config/
│   └── IPs                  # Party hostnames (party0, party1, party2)
├── docker-compose.yml       # 3 Docker containers (party0, party1, party2)
└── Dockerfile              # Builds MP-SPDZ with mascot, shamir, semi, rep-ring
```

## Running the Dark Auction

### Quick Start

```bash
# Start containers and run dark auction with MASCOT protocol
./scripts/run_auction.sh

# Or use the convenience wrapper
./scripts/run_mascot_dark_auction.sh
```

### Custom Protocols

```bash
# Run with specific protocols (mascot, shamir, semi, rep-ring)
./scripts/run_auction.sh dark-auction mascot shamir semi
```

### Test Run (detailed output)

```bash
# Shows each party's output separately
./scripts/test_run.sh
```

## Input Format

Each party submits orders for 3 assets (BTC, ETH, SOL):
- Each asset has 10 orders (configurable)
- Each order: `bid_price bid_qty ask_price ask_qty`
- Zero price means "no order on that side"

**Human-readable** (`Inputs/party{pid}.txt`):
```
# bid_price  bid_qty  ask_price  ask_qty
        85         1         101         2
        91         5         108         2
```

**MP-SPDZ format** (`Inputs/Input-P{pid}-0`):
```
85
1
101
2
91
5
108
2
...
```

## Generate New Inputs

```bash
# Generate random inputs
python3 scripts/generate_inputs.py --n-orders 10 --seed 42

# Convert edited readable files to MP-SPDZ format
python3 scripts/generate_inputs.py --convert-only

# Fixed-point prices
python3 scripts/generate_inputs.py --sfix --n-orders 10 --seed 42
```

## MP-SPDZ Protocols

| Protocol | Security | Type |
|----------|----------|------|
| mascot | Malicious, dishonest majority | Arithmetic (mod prime) |
| shamir | Malicious, honest majority | Arithmetic (mod prime) |
| semi | Semi-honest, dishonest majority | Arithmetic (mod prime) |
| replicated-ring | Semi-honest, honest majority | Arithmetic (mod prime) |

## Docker Containers

The project runs 3 parties as separate Docker containers:
- `party0`, `party1`, `party2`
- Containers auto-configure IP addresses in `Config/IPs`
- Shared volumes for Programs, Config, Inputs, Player-Data

## MP-SPDZ Key Concepts

- `sint` - Secret integer (encrypted/shared value)
- `sfix` - Secret fixed-point (for decimals)
- `sbits` - Secret bits
- `reveal(x)` - Open a secret to all parties
- `print_ln(x)` - Public output
- `print_ln_to(pid, x)` - Private output to specific party