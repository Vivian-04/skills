# HF CLI Command Reference

Complete reference for all `hf` CLI commands and options.

## Table of Contents
- [Authentication](#authentication)
- [Download](#download)
- [Upload](#upload)
- [Repository Management](#repository-management)
- [Repository Files](#repository-files)
- [Cache Management](#cache-management)
- [Jobs](#jobs)
- [Environment](#environment)

---

## Authentication

### hf auth login
Authenticate with Hugging Face Hub.

```bash
hf auth login                              # Interactive login
hf auth login --token $HF_TOKEN            # Non-interactive with token
hf auth login --token $HF_TOKEN --add-to-git-credential  # Also save as git credential
```

**Options:**
| Option | Description |
|--------|-------------|
| `--token` | Access token to use |
| `--add-to-git-credential` | Save token to git credential helper |

### hf auth whoami
Display current authenticated user and organizations.

```bash
hf auth whoami
# Output: username + orgs list
```

### hf auth list
List all stored access tokens.

```bash
hf auth list
```

### hf auth switch
Switch between stored access tokens.

```bash
hf auth switch                              # Interactive selection
hf auth switch --token-name my-token        # Switch to specific token
hf auth switch --add-to-git-credential      # Also update git credentials
```

### hf auth logout
Remove stored authentication tokens.

```bash
hf auth logout                              # Remove active token
hf auth logout --token-name my-token        # Remove specific token
```

**Note:** If logged in via `HF_TOKEN` environment variable, you must unset it in your shell configuration.

---

## Download

### hf download
Download files from the Hub. Uses cache system by default.

**Syntax:**
```bash
hf download <repo_id> [files...] [options]
```

**Options:**
| Option | Description |
|--------|-------------|
| `--repo-type` | `model` (default), `dataset`, or `space` |
| `--revision` | Specific commit, branch, or tag |
| `--include` | Glob patterns to include (e.g., `"*.safetensors"`) |
| `--exclude` | Glob patterns to exclude (e.g., `"*.fp16.*"`) |
| `--local-dir` | Download to specific directory instead of cache |
| `--cache-dir` | Custom cache directory |
| `--force-download` | Force re-download even if cached |
| `--max-workers` | Number of concurrent downloads |
| `--token` | Authentication token |
| `--quiet` | Suppress output except final path |

**Examples:**
```bash
# Single file
hf download gpt2 config.json

# Entire repository
hf download HuggingFaceH4/zephyr-7b-beta

# Multiple specific files
hf download gpt2 config.json model.safetensors

# Nested file path
hf download HiDream-ai/HiDream-I1-Full text_encoder/model.safetensors

# Filter with patterns
hf download stabilityai/stable-diffusion-xl-base-1.0 --include "*.safetensors" --exclude "*.fp16.*"

# Dataset
hf download HuggingFaceH4/ultrachat_200k --repo-type dataset

# Space
hf download HuggingFaceH4/zephyr-chat --repo-type space

# Specific revision
hf download bigcode/the-stack --repo-type dataset --revision v1.1

# To local directory
hf download adept/fuyu-8b model-00001-of-00002.safetensors --local-dir fuyu

# Quiet mode (outputs only path)
hf download gpt2 --quiet
```

**Timeout:** Set `HF_HUB_DOWNLOAD_TIMEOUT` environment variable (default: 10 seconds).

---

## Upload

### hf upload
Upload files or folders to the Hub.

**Syntax:**
```bash
hf upload <repo_id> [local_path] [path_in_repo] [options]
```

**Options:**
| Option | Description |
|--------|-------------|
| `--repo-type` | `model` (default), `dataset`, or `space` |
| `--revision` | Target branch/ref |
| `--include` | Glob patterns to include |
| `--exclude` | Glob patterns to exclude |
| `--delete` | Patterns of remote files to delete |
| `--commit-message` | Custom commit message |
| `--commit-description` | Extended commit description |
| `--create-pr` | Create a pull request |
| `--every` | Upload at regular intervals (minutes) |
| `--token` | Authentication token |
| `--quiet` | Suppress output except final URL |

**Examples:**
```bash
# Upload current directory to repo root
hf upload my-cool-model . .

# Upload specific folder
hf upload my-cool-model ./models .

# Upload to specific path in repo
hf upload my-cool-model ./path/to/curated/data /data/train

# Upload single file
hf upload Wauplin/my-cool-model ./models/model.safetensors

# Upload to subdirectory
hf upload Wauplin/my-cool-model ./models/model.safetensors /vae/model.safetensors

# Upload to organization
hf upload MyCoolOrganization/my-cool-model . .

# Upload dataset
hf upload Wauplin/my-cool-dataset ./data /train --repo-type=dataset

# Create PR instead of direct push
hf upload bigcode/the-stack . . --repo-type dataset --create-pr

# Sync with delete (remove remote files not in local)
hf upload Wauplin/space-example --repo-type=space --exclude="/logs/*" --delete="*"

# Upload with custom commit message
hf upload Wauplin/my-cool-model ./models . --commit-message="Epoch 34/50" --commit-description="Val accuracy: 68%"

# Continuous upload every 10 minutes
hf upload training-model logs/ --every=10
```

### hf upload-large-folder
Optimized upload for very large folders with many files. Uses multi-threading and handles interruptions gracefully.

```bash
hf upload-large-folder <repo_id> <local_folder> [path_in_repo] [options]
```

**Options:**
| Option | Description |
|--------|-------------|
| `--repo-type` | `model`, `dataset`, or `space` |
| `--revision` | Target branch |
| `--private` | Create as private repo if doesn't exist |
| `--include` / `--exclude` | Filter patterns |
| `--num-workers` | Number of upload threads |
| `--token` | Authentication token |

---

## Repository Management

### hf repo create
Create a new repository.

```bash
hf repo create my-cool-model                                    # Public model
hf repo create my-cool-dataset --repo-type dataset --private    # Private dataset
hf repo create my-gradio-space --repo-type space --space_sdk gradio  # Gradio space
```

**Options:**
| Option | Description |
|--------|-------------|
| `--repo-type` | `model`, `dataset`, or `space` |
| `--private` | Create as private repository |
| `--space_sdk` | For spaces: `gradio`, `streamlit`, `docker`, `static` |
| `--token` | Authentication token |

**Note:** Use `--space_sdk` (with underscore), not `--space-sdk`.

### hf repo tag
Manage repository tags.

#### Create a tag
```bash
hf repo tag create <repo_id> <tag> [options]
```

```bash
hf repo tag create Wauplin/my-cool-model v1.0                  # Tag main branch
hf repo tag create Wauplin/my-cool-model v1.0 --revision refs/pr/104  # Tag specific revision
hf repo tag create bigcode/the-stack v1.0 --repo-type dataset  # Tag dataset
```

#### List tags
```bash
hf repo tag list <repo_id> [options]
```

```bash
hf repo tag list Wauplin/my-cool-model
hf repo tag list Wauplin/gradio-space-ci --repo-type space
```

#### Delete a tag
```bash
hf repo tag delete <repo_id> <tag> [options]
```

```bash
hf repo tag delete Wauplin/my-cool-model v1.0
hf repo tag delete Wauplin/my-cool-model v1.0 -y  # Skip confirmation
```

**Common options for all tag commands:**
| Option | Description |
|--------|-------------|
| `--repo-type` | `model`, `dataset`, or `space` |
| `--token` | Authentication token |
| `-y` | Skip confirmation (for delete) |

---

## Repository Files

### hf repo-files delete
Delete files from a repository.

```bash
hf repo-files delete <repo_id> <path_in_repo>... [options]
```

**Examples:**
```bash
# Delete folder
hf repo-files delete Wauplin/my-cool-model folder/

# Delete multiple files
hf repo-files delete Wauplin/my-cool-model file.txt folder/pytorch_model.bin

# Use Unix-style wildcards
hf repo-files delete Wauplin/my-cool-model "*.txt" "folder/*.bin"

# With explicit token
hf repo-files delete Wauplin/my-cool-model file.txt --token=hf_****

# Dataset
hf repo-files delete Wauplin/my-dataset data/old.parquet --repo-type dataset
```

**Options:**
| Option | Description |
|--------|-------------|
| `--repo-type` | `model`, `dataset`, or `space` |
| `--revision` | Branch to delete from |
| `--commit-message` | Custom commit message |
| `--commit-description` | Extended commit description |
| `--create-pr` | Create a PR instead of direct delete |
| `--token` | Authentication token |

---

## Cache Management

### hf cache scan
Scan and display the contents of the local cache.

```bash
hf cache scan                    # Show all cached repos
hf cache scan --verbose          # Detailed output with file info
hf cache scan --dir ~/.cache/huggingface/hub  # Custom cache directory
```

**Options:**
| Option | Description |
|--------|-------------|
| `--dir` | Cache directory to scan |
| `-v, --verbose` | Show verbose output |

**Output includes:** Repository IDs, revisions, sizes, and references (branches/tags).

### hf cache delete
Interactively delete revisions from the cache.

```bash
hf cache delete                  # Interactive TUI mode
hf cache delete --disable-tui    # Non-interactive mode
hf cache delete --dir ./cache    # Custom cache directory
```

**Options:**
| Option | Description |
|--------|-------------|
| `--dir` | Cache directory |
| `--disable-tui` | Disable Terminal User Interface |

**Note:** The TUI allows selecting specific revisions to delete. Use `--disable-tui` in scripts.

---

## Jobs

Run compute jobs on Hugging Face infrastructure.

### hf jobs run
Execute a job.

```bash
hf jobs run <image> <command> [options]
```

**Examples:**
```bash
# Basic Python execution
hf jobs run python:3.12 python -c 'print("Hello from the cloud!")'

# With GPU
hf jobs run --flavor a10g-small pytorch/pytorch:2.6.0-cuda12.4-cudnn9-devel \
  python -c "import torch; print(torch.cuda.get_device_name())"

# In organization namespace
hf jobs run --namespace my-org-name python:3.12 python -c "print('Running in org')"

# From HF Space
hf jobs run hf.co/spaces/lhoestq/duckdb duckdb -c "select 'hello world'"

# With environment variables
hf jobs run -e FOO=foo -e BAR=bar python:3.12 python -c "import os; print(os.environ['FOO'])"

# With env file
hf jobs run --env-file .env python:3.12 python script.py

# With secrets (encrypted)
hf jobs run -s MY_SECRET=psswrd python:3.12 python -c "import os; print(os.environ['MY_SECRET'])"

# Pass HF_TOKEN implicitly
hf jobs run --secrets HF_TOKEN python:3.12 python -c "print('authenticated')"

# Detached mode (returns job ID immediately)
hf jobs run --detach python:3.12 python -c "print('background job')"
```

**Options:**
| Option | Description |
|--------|-------------|
| `--flavor` | Hardware configuration (see below) |
| `--namespace` | Organization namespace |
| `-e` | Environment variable (KEY=value) |
| `--env-file` | Load env vars from file |
| `-s, --secrets` | Secret (encrypted, KEY=value or KEY to read from env) |
| `--secrets-file` | Load secrets from file |
| `--detach` | Run in background, print job ID |
| `--timeout` | Job timeout in seconds |

**Available Flavors:**
| Category | Flavors |
|----------|---------|
| CPU | `cpu-basic`, `cpu-upgrade`, `cpu-xl` |
| T4 GPU | `t4-small`, `t4-medium` |
| L4 GPU | `l4x1`, `l4x4` |
| L40S GPU | `l40sx1`, `l40sx4`, `l40sx8` |
| A10G GPU | `a10g-small`, `a10g-large`, `a10g-largex2`, `a10g-largex4` |
| A100 GPU | `a100-large` |
| H100 GPU | `h100`, `h100x8` |

### hf jobs uv run
Run UV scripts (Python with inline dependencies).

```bash
hf jobs uv run <script.py> [options]
hf jobs uv run <url> [options]
hf jobs uv run --with <package> python -c "<code>"
```

**Examples:**
```bash
hf jobs uv run my_script.py                        # Run script
hf jobs uv run my_script.py --repo my-uv-scripts   # With persistent repo
hf jobs uv run ml_training.py --flavor a10g-small  # With GPU
hf jobs uv run --with transformers --with torch train.py  # Add dependencies
hf jobs uv run https://huggingface.co/datasets/user/scripts/resolve/main/example.py
```

### Job Management
```bash
hf jobs ps                    # List running jobs
hf jobs inspect <job_id>      # Check job status and details
hf jobs logs <job_id>         # View job logs
hf jobs cancel <job_id>       # Cancel a running job
```

### Scheduled Jobs
Schedule jobs to run on a recurring basis.

```bash
# Schedule syntax: @hourly, @daily, @weekly, @monthly, or CRON expression
hf jobs scheduled run @hourly python:3.12 python -c 'print("Every hour")'
hf jobs scheduled run "*/5 * * * *" python:3.12 python -c 'print("Every 5 min")'
hf jobs scheduled run @daily --flavor a10g-small <image> <cmd>
```

**Manage scheduled jobs:**
```bash
hf jobs scheduled ps              # List scheduled jobs
hf jobs scheduled inspect <id>    # View schedule details
hf jobs scheduled suspend <id>    # Pause a schedule
hf jobs scheduled resume <id>     # Resume a paused schedule
hf jobs scheduled delete <id>     # Delete a schedule
```

**Scheduled UV scripts:**
```bash
hf jobs scheduled uv run @hourly my_script.py
```

---

## Environment

### hf env
Print environment information (useful for bug reports).

```bash
hf env
```

**Output includes:**
- huggingface_hub version
- Platform and Python version
- Token status
- Cache paths
- Relevant environment variables

### hf version
Print CLI version.

```bash
hf version
```

---

## Environment Variables

| Variable | Description | Default |
|----------|-------------|---------|
| `HF_TOKEN` | Authentication token | - |
| `HF_HUB_CACHE` | Cache directory | `~/.cache/huggingface/hub` |
| `HF_HUB_DOWNLOAD_TIMEOUT` | Download timeout (seconds) | 10 |
| `HF_HUB_OFFLINE` | Offline mode | False |
| `HF_HUB_DISABLE_PROGRESS_BARS` | Disable progress bars | False |

---

## Global Options

All commands support:
- `--help` - Show command help and options
- `--token` - Override authentication token
- `--repo-type` - Specify repository type (`model`, `dataset`, `space`)
