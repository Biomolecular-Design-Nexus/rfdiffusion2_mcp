# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

RFdiffusion2 MCP is a FastMCP server exposing protein design and structure prediction tools via the Model Context Protocol. It wraps **Chai-1** (structure prediction) and **RFdiffusion2** (enzyme scaffolding, binder design) with async job management.

## Common Commands

```bash
# Setup (creates conda env at ./env, clones RFdiffusion2 repo, installs deps)
bash quick_setup.sh

# Run MCP server
./env/bin/python src/server.py

# Run tests
./env/bin/python tests/test_mcp_server.py
./env/bin/python tests/run_integration_tests.py

# Run scripts directly
./env/bin/python scripts/chai1_structure_prediction.py --sequence "MKLL..." --recycles 3 --output results/
./env/bin/python scripts/enzyme_active_site_scaffolding.py --input examples/data/enzyme.pdb --ligands "NAD,OXM" --output results/
./env/bin/python scripts/small_molecule_binder.py --input examples/data/complex.pdb --ligand PH2 --output results/

# Docker
docker build -t rfdiffusion2-mcp .
docker run --gpus all -p 8000:8000 rfdiffusion2-mcp

# Register with Claude Code
claude mcp add rfdiffusion2 -- $(pwd)/env/bin/python $(pwd)/src/server.py
```

## Architecture

```
Claude Code → MCP Server (src/server.py) → Tool dispatch
                                              ↓
                                    job_manager.submit_job()
                                              ↓
                                    Background thread runs:
                                      mamba run -p ./env python scripts/<tool>.py --args
                                              ↓
                                    jobs/<job_id>/  (metadata.json, job.log, output/)
```

**MCP Server** (`src/server.py`): 12 tools — 1 synchronous (`predict_structure_fast`), 4 async submit tools, 5 job management tools, 2 utility tools. All tools return `{"status": "success"|"error", ...}` dicts.

**Job Manager** (`src/jobs/manager.py`): Spawns background threads that shell out via `mamba run -p ./env python script.py`. Each job gets a UUID-based directory under `jobs/` with metadata.json, job.log, and output files. Status lifecycle: pending → running → completed/failed/cancelled.

**Scripts** (`scripts/`): Three standalone Python scripts that can run independently or be invoked by the job manager:
- `chai1_structure_prediction.py` — uses `chai_lab` Python API directly
- `enzyme_active_site_scaffolding.py` — runs RFdiffusion2 via `apptainer exec`
- `small_molecule_binder.py` — runs RFdiffusion2 via `apptainer exec`

**Shared library** (`scripts/lib/`): `utils.py` for path setup, config merging, FASTA parsing, param validation. `io.py` for file I/O helpers.

**Dependency validation** (`src/utils.py`): Checks three optional deps — `chai_lab` (Python import), `apptainer` (PATH), `rfdf_repo` (repo dir + SIF container). Tools gracefully degrade with installation suggestions when deps are missing.

## Key Patterns

- **Config precedence**: kwargs > user_config > default_config (via `merge_configs()` in `scripts/lib/utils.py`)
- **All paths relative to script location**: `MCP_ROOT = Path(__file__).parent.parent`, `SCRIPTS_DIR = MCP_ROOT / "scripts"`
- **Docker compatibility**: A `mamba` wrapper script in the Dockerfile translates `mamba run -p /app/env python ...` to direct `python ...` execution
- **Model weights location**: `repo/RFdiffusion2/rf_diffusion/model_weights/` (RFD_173.pt, RFD_140.pt) and `third_party_model_weights/ligand_mpnn/`

## Dependencies

- **Core**: `fastmcp==2.14.1`, `loguru==0.7.3`, `numpy`, `pandas`, `tqdm`
- **Structure prediction**: `chai-lab` (includes PyTorch)
- **Protein design**: `apptainer`/`singularity` + RFdiffusion2 repo with SIF containers and model weights
- **RFdiffusion2 Python deps**: See `repo/RFdiffusion2/envs/requirements_cuda121.txt` (PyG, DGL, hydra-core, biopython, rdkit, e3nn, scipy, etc.)
