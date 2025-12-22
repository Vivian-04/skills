---
name: hf-cli
description: Execute Hugging Face Hub operations using the `hf` CLI. Use when the user needs to download models/datasets/spaces, upload files to Hub repositories, create repos, manage local cache, or run compute jobs on HF infrastructure. Covers authentication, file transfers, repository creation, cache operations, and cloud compute.
---

# Hugging Face CLI

The `hf` CLI provides direct terminal access to the Hugging Face Hub for downloading, uploading, and managing repositories, cache, and compute resources.

## Quick Command Reference

| Task | Command |
|------|---------|
| Login | `hf auth login` |
| Download model | `hf download <repo_id>` |
| Download to folder | `hf download <repo_id> --local-dir ./path` |
| Upload folder | `hf upload <repo_id> . .` |
| Create repo | `hf repo create <name>` |
| Create tag | `hf repo tag create <repo_id> <tag>` |
| Delete files | `hf repo-files delete <repo_id> <files>` |
| Scan cache | `hf cache scan` |
| Delete from cache | `hf cache delete` |
| Run GPU job | `hf jobs run --flavor a10g-small <image> <cmd>` |
| Environment info | `hf env` |

## Core Commands

### Authentication
```bash
hf auth login                    # Interactive login
hf auth login --token $HF_TOKEN  # Non-interactive
hf auth whoami                   # Check current user
hf auth list                     # List stored tokens
hf auth switch                   # Switch between tokens
hf auth logout                   # Log out
```

### Download
```bash
hf download <repo_id>                              # Full repo to cache
hf download <repo_id> file.safetensors             # Specific file
hf download <repo_id> --local-dir ./models         # To local directory
hf download <repo_id> --include "*.safetensors"    # Filter by pattern
hf download <repo_id> --repo-type dataset          # Dataset
hf download <repo_id> --revision v1.0              # Specific version
```

### Upload
```bash
hf upload <repo_id> . .                            # Current dir to root
hf upload <repo_id> ./models /weights              # Folder to path
hf upload <repo_id> model.safetensors              # Single file
hf upload <repo_id> . . --repo-type dataset        # Dataset
hf upload <repo_id> . . --create-pr                # Create PR
hf upload <repo_id> . . --commit-message="msg"     # Custom message
```

### Repository Management
```bash
hf repo create <name>                              # Create model repo
hf repo create <name> --repo-type dataset          # Create dataset
hf repo create <name> --private                    # Private repo
hf repo create <name> --repo-type space --space_sdk gradio  # Gradio space
hf repo tag create <repo_id> v1.0                  # Create tag
hf repo tag list <repo_id>                         # List tags
hf repo tag delete <repo_id> v1.0                  # Delete tag
```

### Delete Files from Repo
```bash
hf repo-files delete <repo_id> folder/             # Delete folder
hf repo-files delete <repo_id> "*.txt"             # Delete with pattern
```

### Cache Management
```bash
hf cache scan                    # Scan cached repos (shows size, refs)
hf cache scan --verbose          # Detailed view
hf cache delete                  # Interactive TUI to delete revisions
hf cache delete --disable-tui    # Non-interactive mode
```

### Jobs (Cloud Compute)
```bash
hf jobs run python:3.12 python script.py           # Run on CPU
hf jobs run --flavor a10g-small <image> <cmd>      # Run on GPU
hf jobs run --secrets HF_TOKEN <image> <cmd>       # With HF token
hf jobs ps                                         # List jobs
hf jobs logs <job_id>                              # View logs
hf jobs cancel <job_id>                            # Cancel job
```

**GPU Flavors:** `cpu-basic`, `cpu-upgrade`, `cpu-xl`, `t4-small`, `t4-medium`, `l4x1`, `l4x4`, `l40sx1`, `l40sx4`, `l40sx8`, `a10g-small`, `a10g-large`, `a10g-largex2`, `a10g-largex4`, `a100-large`, `h100`, `h100x8`

## Common Patterns

### Download and Use Model Locally
```bash
# Download to local directory for deployment
hf download meta-llama/Llama-3.2-1B-Instruct --local-dir ./model

# Or use cache and get path
MODEL_PATH=$(hf download meta-llama/Llama-3.2-1B-Instruct --quiet)
```

### Publish Model/Dataset
```bash
hf repo create my-username/my-model --private
hf upload my-username/my-model ./output . --commit-message="Initial release"
hf repo tag create my-username/my-model v1.0
```

### Sync Space with Local
```bash
hf upload my-username/my-space . . --repo-type space \
  --exclude="logs/*" --delete="*" --commit-message="Sync"
```

### Check Cache Usage
```bash
hf cache scan                    # See all cached repos and sizes
hf cache delete                  # Interactive cleanup
```

## Key Options

- `--repo-type`: `model` (default), `dataset`, `space`
- `--revision`: Branch, tag, or commit hash
- `--token`: Override authentication
- `--quiet`: Output only essential info (paths/URLs)

## References

- **Complete command reference**: See [references/commands.md](references/commands.md)
- **Workflow examples**: See [references/examples.md](references/examples.md)
