# RFdiffusion2 MCP

**Protein design and structure prediction using RFdiffusion2 and Chai1 via Docker**

An MCP (Model Context Protocol) server for protein design with 6 core tools:
- Fast structure prediction from protein sequences (Chai1)
- Submit high-quality structure prediction jobs
- Enzyme active site scaffolding (RFdiffusion2)
- Small molecule binder design (RFdiffusion2)
- Batch structure prediction for multiple sequences
- Async job tracking and result retrieval

## Quick Start with Docker

### Approach 1: Pull Pre-built Image from GitHub

The fastest way to get started. A pre-built Docker image is automatically published to GitHub Container Registry on every release.

```bash
# Pull the latest image
docker pull ghcr.io/macromnex/rfdiffusion2_mcp:latest

# Register with Claude Code (runs as current user to avoid permission issues)
claude mcp add rfdiffusion2 -- docker run -i --rm --user `id -u`:`id -g` --gpus all --ipc=host -v `pwd`:`pwd` ghcr.io/macromnex/rfdiffusion2_mcp:latest
```

**Note:** Run from your project directory. `` `pwd` `` expands to the current working directory.

**Requirements:**
- Docker with GPU support (`nvidia-docker` or Docker with NVIDIA runtime)
- Claude Code installed

That's it! The RFdiffusion2 MCP server is now available in Claude Code.

---

### Approach 2: Build Docker Image Locally

Build the image yourself and install it into Claude Code. Useful for customization or offline environments.

```bash
# Clone the repository
git clone https://github.com/MacromNex/rfdiffusion2_mcp.git
cd rfdiffusion2_mcp

# Build the Docker image
docker build -t rfdiffusion2_mcp:latest .

# Register with Claude Code (runs as current user to avoid permission issues)
claude mcp add rfdiffusion2 -- docker run -i --rm --user `id -u`:`id -g` --gpus all --ipc=host -v `pwd`:`pwd` rfdiffusion2_mcp:latest
```

**Note:** Run from your project directory. `` `pwd` `` expands to the current working directory.

**Requirements:**
- Docker with GPU support
- Claude Code installed
- Git (to clone the repository)

**About the Docker Flags:**
- `-i` — Interactive mode for Claude Code
- `--rm` — Automatically remove container after exit
- `` --user `id -u`:`id -g` `` — Runs the container as your current user, so output files are owned by you (not root)
- `--gpus all` — Grants access to all available GPUs
- `--ipc=host` — Uses host IPC namespace for PyTorch shared memory
- `-v` — Mounts your project directory so the container can access your data

---

## Verify Installation

After adding the MCP server, you can verify it's working:

```bash
# List registered MCP servers
claude mcp list

# You should see 'rfdiffusion2' in the output
```

In Claude Code, you can now use all 6 RFdiffusion2 tools:
- `predict_structure_fast`
- `submit_structure_prediction`
- `submit_enzyme_scaffolding`
- `submit_binder_design`
- `get_job_status`
- `get_job_result`

---

## Next Steps

- **Detailed documentation**: See [detail.md](detail.md) for comprehensive guides on:
  - Available MCP tools and parameters
  - Local Python environment setup (alternative to Docker)
  - Example workflows and use cases
  - Configuration file options
  - Troubleshooting

---

## Usage Examples

Once registered, you can use the RFdiffusion2 tools directly in Claude Code. Here are some common workflows:

### Example 1: Quick Structure Prediction

```
I have a protein sequence "MKLLISGLVFGLVLALILSHQQAYEMAQ". Can you use predict_structure_fast to quickly predict its structure and save results to /path/to/results/?
```

### Example 2: Enzyme Active Site Scaffolding

```
I have an enzyme structure at /path/to/enzyme.pdb with NAD and OXM ligands. Can you submit an enzyme scaffolding job using submit_enzyme_scaffolding to design 5 protein scaffolds around the active site? Save results to /path/to/scaffolds/.
```

### Example 3: Small Molecule Binder Design

```
I have a protein-small molecule complex at /path/to/complex.pdb. Can you first list available ligands, then submit a binder design job for ligand PH2 targeting lengths 50-120 residues? Save 3 designs to /path/to/binders/.
```

---

## Troubleshooting

**Docker not found?**
```bash
docker --version  # Install Docker if missing
```

**GPU not accessible?**
- Ensure NVIDIA Docker runtime is installed
- Check with: `docker run --gpus all ubuntu nvidia-smi`

**Claude Code not found?**
```bash
# Install Claude Code
npm install -g @anthropic-ai/claude-code
```

**RFdiffusion2 tools not working?**
- Full RFdiffusion2 functionality requires apptainer and the rfdf_repo setup
- Use `check_dependencies` tool to see what is available
- Structure prediction (Chai1) works without RFdiffusion2

---

## License

BSD — Based on [RFdiffusion2](https://github.com/baker-laboratory/RFdiffusion2) from the Baker Laboratory.
