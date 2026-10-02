# Oasis Statistics -- IQR and z-score outlier detection on Oasis

This repository holds two independent extensions of [Oasis](https://github.com/celeris-labs/oasis)
(Parquet decoding on an FPGA SmartNIC), each adding a statistical outlier-detection operator
in hardware and exposing it as a DuckDB table function:

| Folder | Operator | Guide | Bitstream build |
|---|---|---|---|
| [`iqr/`](iqr) | IQR outlier detection (1.5 × IQR rule, 4096-bin on-chip histogram, `INT64`) | [`iqr/IQR_GUIDE.md`](iqr/IQR_GUIDE.md) | `--no-rdma --decoders 4` |
| [`zscore/`](zscore) | Global z-score outlier detection (two-phase, `INT32`) | [`zscore/ZSCORE_GUIDE.md`](zscore/ZSCORE_GUIDE.md) | `--no-rdma --decoders 3` |

Each folder is a complete Oasis tree (hardware, software, DuckDB extension, scripts) and is built,
programmed and run on its own. Its guide is the full runbook: requirements, build, FPGA
programming, datasets, the real-dataset benchmark, and troubleshooting.

## Quick start

Clone **without** `--recurse-submodules` and initialise only the submodules of the folder you use
(the `celeris` submodules are not needed by either build):

```bash
git config --global url."https://github.com/".insteadOf "git@github.com:"
git clone https://github.com/meminyaksi/oasis-statistics.git && cd oasis-statistics

# IQR
git submodule update --init iqr/extension/duckdb iqr/extension/extension-ci-tools
git submodule update --init --recursive iqr/parcore

# z-score
git submodule update --init zscore/extension/duckdb zscore/extension/extension-ci-tools
git submodule update --init --recursive zscore/parcore
```

Then `cd iqr` or `cd zscore` and follow that folder's guide from step 3. All commands in a guide
run inside its folder.

## Why two folders

The two operators were developed independently on different Oasis versions and are not yet one
design:

- **Separate bitstreams.** The hardware tops differ (`hardware/hdl/vfpga_top.svh`): IQR uses one
  dedicated stream with 4 decoders, z-score runs per decoder lane with 3 decoders. A single FPGA
  image currently runs one operator, so the FPGA is reprogrammed when switching.
- **Different ParCore pins.** `iqr/parcore` and `zscore/parcore` point to different ParCore
  commits, so each folder builds against its own submodules (see `git submodule status`).
- **RDMA.** Both bitstreams are built with `--no-rdma`.

Merging both operators into one hardware design is future work.
