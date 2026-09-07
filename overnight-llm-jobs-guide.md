# Overnight LLM Jobs Guide

A repeatable way to run long local-LLM jobs unattended on a home server: a small job
runner script, shell helpers for starting and watching jobs, and a checklist for leaving
one running overnight.

**Read the [Home Server Guide](home-server-guide.md) first.** This is a companion to it,
not a standalone setup guide. It assumes the layout that guide describes - Ollama running
in a Docker container named `ollama`, models under `/srv/models/ollama`, and the `gpu-mode`
helper that switches the GPU between the AI and media services. If you run Ollama some
other way, the ideas transfer but the commands will need adjusting.

**Version note:** the `ollama` command-line flags used here (`run`, `list`, `pull`,
`--hidethinking`) were checked against Ollama's published CLI, and every model tag named
below exists in Ollama's public model library. The runner script itself was checked as
Bash. What could **not** be checked is how any of this performs, because that depends on
your hardware - every statement about speed or memory fit below is an expectation to test,
not a measurement. If your machine disagrees with this page, your machine is right.

## The example job

This guide uses a large reasoning model as its worked example:

```text
deepseek-r1:32b
```

It is written around a machine with a 12 GB NVIDIA GeForce RTX 3080 Ti and 64 GB of system
RAM. A 32B model does not fit in 12 GB of VRAM, so Ollama will split it between GPU and
system RAM. That is exactly what makes it a good *overnight* job and a poor interactive
one: it will finish, but not while you wait for it.

Substitute whatever model suits your own hardware. On a card with more VRAM you may not
need the overnight treatment at all for a model this size; on a smaller card, even a 14B
model may deserve it. The workflow does not care which model you use.

## What you get

- A script that starts a timestamped background job from a prompt file.
- Persistent prompt and result folders.
- Shell functions for job creation, status, logs, output, GPU/RAM monitoring, and stopping.
- A checklist for running an overnight job safely.
- Maintenance and recovery notes.

## Table of Contents

1. [Before using overnight jobs](#before-using-overnight-jobs)
2. [Directory layout](#directory-layout)
3. [Overnight job runner script](#overnight-job-runner-script)
4. [Add shell aliases/functions](#add-shell-aliasesfunctions)
5. [Alias reference](#alias-reference)
6. [Step-by-step: run an overnight job](#step-by-step-run-an-overnight-job)
7. [Job output locations](#job-output-locations)
8. [Important limitations](#important-limitations)
9. [Troubleshooting](#troubleshooting)
10. [Backup guidance](#backup-guidance)
11. [What to record in your own notes](#what-to-record-in-your-own-notes)

---

## Before using overnight jobs

Before starting a large model job:

1. Reserve the RTX 3080 Ti for AI:

```bash
gpu-mode ai
```

2. Confirm Jellyfin is stopped if GPU transcoding might otherwise occur:

```bash
cd /srv/compose/jellyfin
docker compose stop
```

3. Confirm enough free storage exists for the model and output:

```bash
df -h / /srv
du -sh /srv/models/ollama
```

4. Confirm Ollama is running:

```bash
cd /srv/compose/ollama
docker compose ps
```

5. Pull the reasoning model once, if it is not already installed:

```bash
ollama pull deepseek-r1:32b
```

6. Confirm the model is available:

```bash
ollama list
```

A bare `ollama` here means the wrapper function defined in [The `ollama` and `ai-*`
commands](home-server-guide.md#the-ollama-and-ai--commands) in the Home Server Guide; Ollama
itself runs only in the container. Without that wrapper, use the container form directly -
`docker exec ollama ollama pull deepseek-r1:32b` and `docker exec ollama ollama list`, which
is the form the runner script below uses throughout.

## Directory layout

Store prompt files and results under your own home directory. This guide writes it as
`/home/you/`; substitute your own login name, or just use `~`:

```text
/home/you/llm-jobs/
├── prompts/                 Input task files
├── output/                  Final model output files
├── error/                   Standard-error files
├── logs/                    Runner metadata logs
└── running/                 PID and job metadata while jobs run
```

Create the layout once:

```bash
mkdir -p ~/llm-jobs/{prompts,output,error,logs,running}
```

## Overnight job runner script

```bash
mkdir -p ~/.local/bin
```

Create the file:

```text
~/.local/bin/llm-job
```

with this content:

```bash
#!/usr/bin/env bash
set -Eeuo pipefail

JOBS_DIR="$HOME/llm-jobs"
PROMPTS_DIR="$JOBS_DIR/prompts"
OUTPUT_DIR="$JOBS_DIR/output"
ERROR_DIR="$JOBS_DIR/error"
LOGS_DIR="$JOBS_DIR/logs"
RUNNING_DIR="$JOBS_DIR/running"
DEFAULT_MODEL="deepseek-r1:32b"

usage() {
  cat <<'USAGE'
Usage:
  llm-job run PROMPT_FILE [MODEL] [JOB_NAME]
  llm-job status [JOB_NAME]
  llm-job list
  llm-job tail JOB_NAME
  llm-job output JOB_NAME
  llm-job error JOB_NAME
  llm-job stop JOB_NAME
  llm-job cleanup
  llm-job help

Commands:
  run       Start a detached Ollama job from a prompt file.
  status    Show state and file locations for one job, or all jobs.
  list      List submitted jobs and their current state.
  tail      Follow a job's output file. Press Ctrl+C to stop viewing.
  output    Open a job's output in Neovim.
  error     Open a job's error file in Neovim.
  stop      Stop a running job by name.
  cleanup   Remove stale PID files for jobs that are no longer running.
  help      Show this help.

Examples:
  llm-job run ~/llm-jobs/prompts/overnight.txt
  llm-job run ~/llm-jobs/prompts/research.txt deepseek-r1:32b research-test
  llm-job status research-test-YYYY-MM-DD-HHMMSS
  llm-job tail research-test-YYYY-MM-DD-HHMMSS
  llm-job output research-test-YYYY-MM-DD-HHMMSS
  llm-job stop research-test-YYYY-MM-DD-HHMMSS
USAGE
}

init_dirs() {
  mkdir -p "$PROMPTS_DIR" "$OUTPUT_DIR" "$ERROR_DIR" "$LOGS_DIR" "$RUNNING_DIR"
}

safe_name() {
  local value="$1"
  value="${value// /-}"
  value="${value//[^a-zA-Z0-9._-]/-}"
  printf '%s' "$value"
}

meta_file() {
  printf '%s/%s.meta' "$LOGS_DIR" "$1"
}

pid_file() {
  printf '%s/%s.pid' "$RUNNING_DIR" "$1"
}

# Field 22 of /proc/<pid>/stat is the process start time, measured from boot. It is
# unique per PID incarnation, so it tells a reused PID apart from the original job.
# The comm field can contain spaces and parentheses, so drop everything up to ") "
# before counting fields.
proc_starttime() {
  local pid="$1" stat rest
  stat="$(cat "/proc/$pid/stat" 2>/dev/null)" || return 1
  rest="${stat##*) }"
  awk '{print $20}' <<<"$rest"
}

# True only when this PID is still the process the job started. Fails closed: if
# anything is missing or unreadable, the answer is "not ours", because the cost of a
# wrong "yes" is signalling an unrelated process.
pid_owned_by_job() {
  local pid="$1" want_start="$2" now_start
  [[ -n "$pid" && -n "$want_start" ]] || return 1
  kill -0 "$pid" 2>/dev/null || return 1
  now_start="$(proc_starttime "$pid")" || return 1
  [[ "$now_start" == "$want_start" ]]
}

read_meta() {
  local job_name="$1"
  local file
  file="$(meta_file "$job_name")"

  if [[ ! -f "$file" ]]; then
    printf 'Error: job "%s" was not found.\n' "$job_name" >&2
    exit 1
  fi

  # shellcheck disable=SC1090
  source "$file"
}

job_state() {
  local job_name="$1"
  local pid_path
  pid_path="$(pid_file "$job_name")"

  if [[ ! -f "$pid_path" ]]; then
    printf 'finished or unknown'
    return
  fi

  local pid
  pid="$(cat "$pid_path")"

  if pid_owned_by_job "$pid" "${STARTTIME:-}"; then
    printf 'running (PID %s)' "$pid"
  else
    printf 'finished'
  fi
}

run_job() {
  init_dirs

  local prompt_file="${1:-}"
  local model="${2:-$DEFAULT_MODEL}"
  local supplied_name="${3:-}"

  if [[ -z "$prompt_file" ]]; then
    printf 'Error: a prompt file is required.\n\n' >&2
    usage >&2
    exit 2
  fi

  if [[ ! -f "$prompt_file" ]]; then
    printf 'Error: prompt file does not exist: %s\n' "$prompt_file" >&2
    exit 1
  fi

  local base_name job_name stamp
  if [[ -n "$supplied_name" ]]; then
    base_name="$(safe_name "$supplied_name")"
  else
    base_name="${prompt_file##*/}"
    base_name="$(safe_name "${base_name%.*}")"
  fi
  stamp="$(date +%F-%H%M%S)"
  job_name="${base_name}-${stamp}"

  local output_file error_file log_file pid_path
  output_file="$OUTPUT_DIR/$job_name.txt"
  error_file="$ERROR_DIR/$job_name.err"
  log_file="$LOGS_DIR/$job_name.meta"
  pid_path="$(pid_file "$job_name")"

  if ! docker inspect ollama >/dev/null 2>&1; then
    printf 'Error: the Ollama container does not exist. Start it first.\n' >&2
    exit 1
  fi

  if [[ "$(docker inspect --format '{{.State.Running}}' ollama)" != "true" ]]; then
    printf 'Error: the Ollama container is not running. Start it with gpu-mode ai or docker compose start.\n' >&2
    exit 1
  fi

  # Ollama's default tag is "latest", but `ollama list` always prints the full
  # name:tag. Add the implicit tag before matching, so `devstral` finds
  # `devstral:latest` instead of reporting it as not installed.
  local model_tagged="$model"
  [[ "$model_tagged" == *:* ]] || model_tagged="$model_tagged:latest"

  if ! docker exec ollama ollama list | awk 'NR>1 {print $1}' | grep -Fxq "$model_tagged"; then
    printf 'Error: model "%s" is not installed.\n' "$model" >&2
    printf 'Install it with: docker exec ollama ollama pull %s\n' "$model" >&2
    exit 1
  fi

  cat > "$log_file" <<EOF
JOB_NAME=$(printf '%q' "$job_name")
MODEL=$(printf '%q' "$model")
PROMPT_FILE=$(printf '%q' "$(readlink -f "$prompt_file")")
OUTPUT_FILE=$(printf '%q' "$output_file")
ERROR_FILE=$(printf '%q' "$error_file")
STARTED_AT=$(printf '%q' "$(date -Is)")
EOF

  printf 'Starting job: %s\n' "$job_name"
  printf 'Model:        %s\n' "$model"
  printf 'Prompt:       %s\n' "$(readlink -f "$prompt_file")"
  printf 'Output:       %s\n' "$output_file"
  printf 'Errors:       %s\n' "$error_file"

  nohup docker exec -i ollama ollama run "$model" --hidethinking \
    < "$prompt_file" \
    > "$output_file" \
    2> "$error_file" \
    &

  local pid=$!
  printf '%s\n' "$pid" > "$pid_path"

  printf 'PID=%q\n' "$pid" >> "$log_file"
  printf 'STARTTIME=%q\n' "$(proc_starttime "$pid")" >> "$log_file"
  printf 'Submitted job. It will continue after this terminal disconnects.\n'
  printf 'Check it with: llm-job status %s\n' "$job_name"
  printf 'Follow output:  llm-job tail %s\n' "$job_name"
}

show_one_status() {
  local job_name="$1"
  read_meta "$job_name"

  printf 'Job:          %s\n' "$JOB_NAME"
  printf 'State:        %s\n' "$(job_state "$JOB_NAME")"
  printf 'Model:        %s\n' "$MODEL"
  printf 'Started:      %s\n' "$STARTED_AT"
  printf 'Prompt:       %s\n' "$PROMPT_FILE"
  printf 'Output:       %s\n' "$OUTPUT_FILE"
  printf 'Errors:       %s\n' "$ERROR_FILE"

  if [[ -f "$OUTPUT_FILE" ]]; then
    printf 'Output size:  '
    du -h "$OUTPUT_FILE" | awk '{print $1}'
  fi

  if [[ -f "$ERROR_FILE" && -s "$ERROR_FILE" ]]; then
    printf 'Error size:   '
    du -h "$ERROR_FILE" | awk '{print $1}'
  fi
}

list_jobs() {
  init_dirs

  shopt -s nullglob
  local file job_name
  local files=("$LOGS_DIR"/*.meta)

  if (( ${#files[@]} == 0 )); then
    printf 'No submitted jobs found in %s\n' "$LOGS_DIR"
    return
  fi

  printf '%-36s %-22s %-18s %s\n' 'JOB' 'STATE' 'MODEL' 'STARTED'
  for file in "${files[@]}"; do
    job_name="$(basename "$file" .meta)"
    read_meta "$job_name"
    printf '%-36s %-22s %-18s %s\n' \
      "$JOB_NAME" "$(job_state "$JOB_NAME")" "$MODEL" "$STARTED_AT"
  done
}

tail_job() {
  local job_name="$1"
  read_meta "$job_name"
  touch "$OUTPUT_FILE"
  tail -n 40 -f "$OUTPUT_FILE"
}

open_output() {
  local job_name="$1"
  read_meta "$job_name"
  nvim "$OUTPUT_FILE"
}

open_error() {
  local job_name="$1"
  read_meta "$job_name"
  touch "$ERROR_FILE"
  nvim "$ERROR_FILE"
}

stop_job() {
  local job_name="$1"
  read_meta "$job_name"

  local pid_path pid
  pid_path="$(pid_file "$JOB_NAME")"

  if [[ ! -f "$pid_path" ]]; then
    printf 'Job "%s" does not have a running PID file.\n' "$JOB_NAME" >&2
    exit 1
  fi

  pid="$(cat "$pid_path")"

  if ! pid_owned_by_job "$pid" "${STARTTIME:-}"; then
    printf 'Job "%s" is already finished, or its record is stale (for example after a\n' "$JOB_NAME"
    printf 'reboot). PID %s is not this job any more, so nothing was signalled.\n' "$pid"
    rm -f "$pid_path"
    return
  fi

  printf 'Stopping job "%s" (PID %s)...\n' "$JOB_NAME" "$pid"
  kill "$pid"
  sleep 2

  if kill -0 "$pid" 2>/dev/null; then
    printf 'Job did not stop gracefully; sending a forceful stop.\n'
    kill -9 "$pid"
  fi

  rm -f "$pid_path"
  printf 'Stopped the local client. The model may still be generating inside the\n'
  printf 'container - check with: docker exec ollama ollama ps\n'
  printf 'Output written so far, if any, is in: %s\n' "$OUTPUT_FILE"
}

cleanup() {
  init_dirs

  shopt -s nullglob
  local file job_name pid
  local files=("$RUNNING_DIR"/*.pid)

  for file in "${files[@]}"; do
    job_name="$(basename "$file" .pid)"
    pid="$(cat "$file" 2>/dev/null || true)"
    STARTTIME=""
    # shellcheck disable=SC1090
    [[ -f "$(meta_file "$job_name")" ]] && source "$(meta_file "$job_name")"
    if ! pid_owned_by_job "$pid" "${STARTTIME:-}"; then
      rm -f "$file"
      printf 'Removed stale PID record: %s\n' "$file"
    fi
  done
}

command="${1:-help}"
shift || true

case "$command" in
  run)
    run_job "$@"
    ;;
  status)
    if [[ $# -eq 0 ]]; then
      list_jobs
    else
      show_one_status "$1"
    fi
    ;;
  list)
    list_jobs
    ;;
  tail)
    if [[ $# -ne 1 ]]; then
      printf 'Error: tail requires one job name.\n' >&2
      exit 2
    fi
    tail_job "$1"
    ;;
  output)
    if [[ $# -ne 1 ]]; then
      printf 'Error: output requires one job name.\n' >&2
      exit 2
    fi
    open_output "$1"
    ;;
  error)
    if [[ $# -ne 1 ]]; then
      printf 'Error: error requires one job name.\n' >&2
      exit 2
    fi
    open_error "$1"
    ;;
  stop)
    if [[ $# -ne 1 ]]; then
      printf 'Error: stop requires one job name.\n' >&2
      exit 2
    fi
    stop_job "$1"
    ;;
  cleanup)
    cleanup
    ;;
  help|-h|--help)
    usage
    ;;
  *)
    printf 'Unknown command: %s\n\n' "$command" >&2
    usage >&2
    exit 2
    ;;
esac
```

Two details in that script are easy to get wrong and worth pointing out, because both
fail quietly rather than loudly:

- **The order of redirections matters.** Bash applies them left to right, and the last one
  wins. Adding a `< /dev/null` after `< "$prompt_file"` - a habit people carry over from
  detaching other background jobs - silently replaces the prompt with nothing, and the job
  runs to completion against an empty input. Redirect stdin from the prompt file and
  nothing else.
- **Ollama's implicit tag is `latest`.** `ollama pull devstral` stores the model as
  `devstral:latest`, and that is what `ollama list` prints. Comparing a bare `devstral`
  against that output finds no match, so the script fills the implicit tag in before
  checking.

Make it executable:

```bash
chmod 0755 ~/.local/bin/llm-job
```

Confirm it is available:

```bash
command -v llm-job
llm-job help
```

Expected location:

```text
/home/you/.local/bin/llm-job
```

## Add shell aliases/functions

Add these functions to the end of `~/.bashrc` with Neovim:

```bash
nvim ~/.bashrc
```

Add:

```bash
# Large reasoning model: DeepSeek-R1 32B.
# `ollama` is the wrapper function from the Home Server Guide; without it, use
# `docker exec -it ollama ollama run ...` here instead.
ai-reason() {
  ollama run deepseek-r1:32b --hidethinking "$@"
}

# Overnight/background LLM job helpers.
ai-job() {
  llm-job run "$@"
}

ai-jobs() {
  llm-job list
}

ai-job-status() {
  llm-job status "$@"
}

ai-job-tail() {
  llm-job tail "$@"
}

ai-job-output() {
  llm-job output "$@"
}

ai-job-error() {
  llm-job error "$@"
}

ai-job-stop() {
  llm-job stop "$@"
}

ai-job-cleanup() {
  llm-job cleanup
}

# Snapshot and live monitoring for long jobs.
ai-monitor() {
  gpu-mode status
}

ai-watch() {
  gpu-mode watch
}
```

Save and exit Neovim:

```text
Esc
:wq
Enter
```

Reload Bash and verify:

```bash
source ~/.bashrc
hash -r

type ai-reason ai-job ai-jobs ai-job-status ai-job-tail ai-job-output ai-job-error ai-job-stop ai-job-cleanup ai-monitor ai-watch
```

Each should report `function`.

## Alias reference

| Command | Background action | Use case |
|---|---|---|
| `ai-reason` | Runs `deepseek-r1:32b` with hidden thinking output. | Interactive larger reasoning tasks. |
| `ai-job PROMPT [MODEL] [NAME]` | Runs `llm-job run`. | Start a detached, timestamped job from a text prompt file. |
| `ai-jobs` | Runs `llm-job list`. | List submitted job names, model names, start times, and states. |
| `ai-job-status JOB` | Runs `llm-job status JOB`. | Show one job's state, PID, prompt, output, and error locations. |
| `ai-job-tail JOB` | Runs `llm-job tail JOB`. | Watch a running job's final-answer output. Press `Ctrl+C` to stop watching. |
| `ai-job-output JOB` | Runs `llm-job output JOB`. | Open completed or partial output in Neovim. |
| `ai-job-error JOB` | Runs `llm-job error JOB`. | Open standard errors in Neovim. |
| `ai-job-stop JOB` | Runs `llm-job stop JOB`. | Stop a job's local client; output written so far remains saved. See [`stop` stops the client, not necessarily the model](#stop-stops-the-client-not-necessarily-the-model). |
| `ai-job-cleanup` | Runs `llm-job cleanup`. | Remove stale job PID records after a job ends or the server reboots. |
| `ai-monitor` | Runs `gpu-mode status`. | One-time GPU/RAM/service/disk snapshot. |
| `ai-watch` | Runs `gpu-mode watch`. | Live GPU/RAM/service/disk monitoring during a long job. |

## Step-by-step: run an overnight job

### 1. Reserve the GPU for AI

```bash
gpu-mode ai
```

This stops Jellyfin and starts Ollama. It avoids GPU contention with Jellyfin transcoding.

### 2. Create a prompt file

Create a clearly named prompt file:

```bash
nvim ~/llm-jobs/prompts/overnight-reasoning.txt
```

Write the full task in the file. Include context, required output format, constraints, and what the model should not do.

Example prompt:

```text
Analyze the following project plan as a careful technical reviewer.

Goals:
- Identify hidden assumptions.
- Identify technical, security, operational, and cost risks.
- Identify missing decisions.
- Propose a prioritized remediation plan.
- Separate high-confidence conclusions from uncertain assumptions.

Write the final result in these sections:
1. Executive summary
2. Risks and severity
3. Missing information
4. Recommended next steps
5. Questions to answer before implementation

Project plan:
PASTE THE PLAN HERE
```

Save and exit:

```text
Esc
:wq
Enter
```

### 3. Start the job

Use the default large reasoning model:

```bash
ai-job ~/llm-jobs/prompts/overnight-reasoning.txt
```

Or choose a clear job label:

```bash
ai-job ~/llm-jobs/prompts/overnight-reasoning.txt deepseek-r1:32b project-review
```

The command prints the exact generated job name. Copy it. It will resemble:

```text
project-review-2026-09-06-031500
```

The process continues even when the terminal/SSH connection closes.

### 4. Check status before going to sleep

```bash
ai-jobs
```

Then check the exact job:

```bash
ai-job-status project-review-YYYY-MM-DD-HHMMSS
```

Replace the example name with the name printed by the job runner.

### 5. Monitor while it runs

Watch the output arrive:

```bash
ai-job-tail project-review-YYYY-MM-DD-HHMMSS
```

Press `Ctrl+C` to stop following output; this does not stop the job.

Watch hardware usage:

```bash
ai-watch
```

Press `Ctrl+C` to stop the live monitor.

For a one-time resource snapshot:

```bash
ai-monitor
```

### 6. Read results the next day

List jobs:

```bash
ai-jobs
```

Open output:

```bash
ai-job-output project-review-YYYY-MM-DD-HHMMSS
```

Open errors if the result is missing or incomplete:

```bash
ai-job-error project-review-YYYY-MM-DD-HHMMSS
```

### 7. Stop an unwanted job

```bash
ai-job-stop project-review-YYYY-MM-DD-HHMMSS
```

The script keeps whatever final-answer output and error log were written, and it does not
delete your prompt file. It stops the local client rather than the model itself - read
[`stop` stops the client, not necessarily the
model](#stop-stops-the-client-not-necessarily-the-model) before starting a replacement job.

### 8. Remove stale job records

After jobs finish, or after a server reboot:

```bash
ai-job-cleanup
```

This removes only old PID tracking files whose processes are no longer running. It does not remove prompts or outputs.

## Job output locations

For a job named:

```text
project-review-2026-09-06-031500
```

files are stored here:

```text
~/llm-jobs/prompts/                       Original input task file
~/llm-jobs/output/project-review-2026-09-06-031500.txt  Final answer output
~/llm-jobs/error/project-review-2026-09-06-031500.err   Error output
~/llm-jobs/logs/project-review-2026-09-06-031500.meta   Job details
~/llm-jobs/running/project-review-2026-09-06-031500.pid PID while running
```

The final-answer file contains only normal model output because the script passes `--hidethinking`.

## Important limitations

### One large job at a time

Run one large reasoning job at a time. Multiple large jobs can compete for the same 12 GB GPU and 64 GB system RAM, slowing all jobs or causing out-of-memory errors.

### Do not transcode media concurrently

For the best chance of a successful overnight job:

```bash
gpu-mode ai
```

Do not start a Jellyfin GPU transcode until the LLM job finishes. If media streaming is needed overnight, use Direct Play where possible or run the LLM job another night.

### Model thinking is hidden, not necessarily disabled

Ollama documents `--hidethinking` as "Hide thinking output (if provided)". It suppresses
the reasoning text from what gets written to your output file; it does not stop the model
reasoning, and the model still spends the time. Expect a long quiet stretch before any
final answer appears - that is normal for a reasoning model and not a sign the job has
hung. The separate `--think` flag is what actually controls thinking mode on models that
support it.

### `stop` stops the client, not necessarily the model

`llm-job stop` kills the host-side `docker exec` client. Docker does not signal a process
inside a container when its exec client goes away, so the model can keep generating and keep
holding VRAM after the command reports success. `llm-job list` then reports the job as
`finished or unknown`, while the work behind it may still be running.

That matters because starting a replacement job right after a "stop" can put two large models
on one GPU at once - exactly the situation [One large job at a
time](#one-large-job-at-a-time) warns about.

Check what is actually loaded, and unload it if needed:

```bash
docker exec ollama ollama ps
docker exec ollama ollama stop deepseek-r1:32b
```

`ollama ps` lists running models. `ollama stop MODEL` takes exactly one model name and is
documented as stopping a running model: it sets that model's keep-alive to zero and unloads
it, and reports `couldn't find model "<name>" to stop` if the model is not loaded. Whether it
also aborts a generation that is already in flight has **not** been verified here against a
running server, so do not assume it does. The check that matters is `docker exec ollama
ollama ps` showing nothing before you start a replacement job.

### Background job versus reboot

`nohup` keeps a job running after an SSH disconnect. It does not survive a server reboot, Docker restart, Ollama container restart, or power loss. If the server reboots, the job ends and partial output may remain.

A job's PID file can outlive the boot that created it, and after a reboot that number may
belong to an entirely unrelated process. This is why the runner compares the process start
time it recorded at launch against the process holding the PID now, rather than trusting the
PID alone: without that check, `stop` would signal whatever inherited the number.

### Large outputs

For very large research jobs, output files can grow. Check:

```bash
du -sh ~/llm-jobs
df -h / /srv
```

The prompt/output folders live in your home directory, which on this layout is on the OS drive rather than the larger `/srv` service drive. Keep output files reasonably sized. If long-term archival use becomes frequent, move job data to a dedicated directory on `/srv` and add it to the backup plan.

## Troubleshooting

### The job says Ollama is not running

```bash
gpu-mode ai
cd /srv/compose/ollama
docker compose ps
```

Then retry the job.

### The job says the model is not installed

```bash
docker exec ollama ollama pull deepseek-r1:32b
```

Then retry.

### The job appears slow

Check model/GPU/RAM status:

```bash
ai-monitor
```

Make sure Jellyfin is stopped:

```bash
gpu-mode ai
```

A 32B reasoning model is expected to be slower than 4B–14B models on a 12 GB GPU because it may use host RAM.

### The job ends unexpectedly

Check its status and error file:

```bash
ai-jobs
ai-job-status JOB-NAME
ai-job-error JOB-NAME
```

Check Ollama logs:

```bash
docker logs --tail=200 ollama
```

Check memory and disk:

```bash
free -h
df -h / /srv
docker system df
```

### The job runner command is not found

```bash
source ~/.bashrc
hash -r
command -v llm-job
ls -l ~/.local/bin/llm-job
```

If `~/.local/bin` did not exist when you logged in, it may not be on your `PATH` yet - log
out and back in, or run this for the current session:

```bash
export PATH="$HOME/.local/bin:$PATH"
```

If the file exists but is not executable:

```bash
chmod 0755 ~/.local/bin/llm-job
```

## Backup guidance

The existing routine backup plan excludes local Ollama model downloads because models are reproducible. Prompt files and output reports may become personally valuable, however.

Choose one of these policies:

| Policy | What to do |
|---|---|
| Temporary experiment results | Leave `~/llm-jobs` out of routine backups and delete outputs that are no longer useful. |
| Important reports/plans | Add `/home/you/llm-jobs/prompts` and `/home/you/llm-jobs/output` to the backup script. |
| Mixed use | Copy only selected completed outputs to a documented project folder that is included in backups. |

Do not back up `~/llm-jobs/running` because PID files are temporary and not useful after a reboot. Error/log files can be backed up only if they are useful for audit/troubleshooting.

Back up the runner script itself, and the prompts and outputs too if you decide they are worth keeping, by editing the backup script:

```bash
nvim ~/.local/bin/backup-jellyfin-ollama
```

Add these paths to the backup `tar` command:

```bash
  /home/you/.local/bin/llm-job \
  /home/you/llm-jobs/prompts \
  /home/you/llm-jobs/output \
```

Then test a backup and inspect it before relying on the new scope.

## What to record in your own notes

Whatever you use to document your own server - the [Home Server
Guide](home-server-guide.md) layout, a wiki page, a text file next to your backups - a
section on overnight jobs is worth keeping there too. Six months later, the thing you will
have forgotten is not how `llm-job` works but where its files went and why the job died.

Record:

- The directory layout you chose for prompts, output, errors, logs, and PID files.
- The `llm-job` script path and its command reference.
- The `ai-reason`, `ai-job`, `ai-jobs`, `ai-job-status`, `ai-job-tail`, `ai-job-output`,
  `ai-job-error`, `ai-job-stop`, `ai-job-cleanup`, `ai-monitor`, and `ai-watch` helpers.
- The standard sequence: `gpu-mode ai`, write the prompt, start `ai-job`, verify status,
  monitor if you want to, read the output the next day.
- The warning that jobs do not survive a reboot or power loss.
- The rule of one large job at a time, and no Jellyfin GPU transcoding alongside it.
- Which backup policy you picked for prompts and reports.
