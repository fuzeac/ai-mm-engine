# Ai MM Engine

> **Rust + WASM automated market‑making core for FUZE.ac**  
> Real‑time liquidity, RL inventory skew agent, PM2 orchestration.

[![CI](https://github.com/fuze-ac/fuze-ai-mm-engine/actions/workflows/ci.yml/badge.svg)](./actions)
[![Rust](https://img.shields.io/badge/Rust-1.77%2B-orange)](https://www.rust-lang.org/)
[![License: MIT](https://img.shields.io/badge/License-MIT-lightgrey.svg)](#license)

---

## ✨  What’s inside?

| Layer / Dir                 | Highlights                                                        |
|-----------------------------|-------------------------------------------------------------------|
| `crates/core-mm`           | Low‑latency Rust hot‑loop (25‑50 ms p99) compiling to native **or** WASM |
| `crates/rl-agent`          | PPO + Bi‑LSTM inventory skew policy, Torch‑RS, on‑policy retrain   |
| `process/pm2`              | Per‑pair process manifests, health checks, log rotation scripts   |
| `listeners/`               | Unlock feed, Twitter alert, CEX webhook handlers (TypeScript)     |
| `proof/`                   | Merkle tree builder → root publisher (Arbitrum Nova)              |
| `examples/`                | Demo pair FUZE/USDT on KuCoin sandbox                             |

---

## 🏗️  Quick Start

### 1 Prereqs

```bash
rustup install stable
cargo install wasm-pack
pnpm i -g pm2
````

### 2 Build Core

```bash
cargo build --release -p core-mm
wasm-pack build crates/core-mm --target web
```

### 3 Spin up a Local Pair Worker

```bash
export EXCH_API_KEY="..."
export PAIR="FUZE/USDT"
pm2 start process/pm2/pair-worker.json --name fuze-mm-demo
```

### 4 Run RL Training (optional)

```bash
cargo run -p rl-agent --features=train \
      -- --pair FUZE/USDT --epochs 1000
```

---

## ⚙️ Configuration

`config/default.toml`

```toml
[exchange]
venue      = "kucoin"
rest_url   = "https://api.kucoin.com"
ws_url     = "wss://ws-api.kucoin.com/endpoint"

[pair]
symbol     = "FUZE/USDT"
min_spread = 0.01
max_depth  = 5000

[risk]
max_inv_pct   = 0.15
kill_switch_spread = 0.04
```

Hot‑reload supported: `pm2 reload ecosystem.config.js`.

---

## 🔒 Security

* All API keys AES‑256‑GCM encrypted at rest (`~/.fuze-mm/keys.db`).
* Quote freeze + `SIGSTOP` if WebSocket lag > 500 ms.
* `proof/` publisher pushes hourly Merkle roots for on‑chain verification.

---

## 🧪 Testing & Benchmarks

```bash
cargo test --all
cargo bench -p core-mm --features=sim
```

Expected p99 latency < 50 ms on Intel 12700H / Ubuntu.

---

## 🤝 Contributing

1. Fork → feature branch
2. `cargo fmt && cargo clippy --all -- -D warnings`
3. Add unit tests (criterion or vitest for TS listeners)
4. PR against `develop`

All contributors must sign the CLA.

---

## 📄 License

MIT © 2025 FUZE Foundation

