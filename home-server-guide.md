# Home Server Guide for Beginners

A plain-language operating guide for a self-hosted Ubuntu home server: what to install,
where the data lives, how to run and maintain it, how to expand it with new drives and
services, and how to get it back after something breaks.

The example server in this guide runs:

- **Jellyfin** for private home-media streaming.
- **Ollama** for local AI models, light coding help, and scripting help.
- **Tailscale** for private remote access, without exposing Jellyfin to the public internet.
- **Docker** to run Jellyfin and Ollama in self-contained containers.
- **NVIDIA GPU acceleration** for either local AI or Jellyfin video transcoding.

You do not need to understand every technical detail to follow the day-to-day steps. Use
commands as shown, substitute your own values where the guide says to, and read the
warnings before running anything that changes or deletes data.

**This guide is written around one example machine.** Every address, account name, drive
name, and mount point below is an example. Yours will differ, and the guide says so at
each point where it matters. The hardware figures the guide reasons about - a 12 GB
NVIDIA GeForce RTX 3080 Ti and 64 GB of system RAM - are the example machine's, and they
are what the model-sizing advice is calibrated to. If your GPU has more or less VRAM,
the *reasoning* still applies but the specific model recommendations will shift.

**Companion guide:** once this server is running, [Overnight LLM Jobs
Guide](overnight-llm-jobs-guide.md) covers running long local-LLM jobs unattended against
it. Read this guide first; the overnight guide assumes the layout and the `gpu-mode`
helper described here.

**What was checked, and what could not be.** Commands, flags, image tags, model tags, and
documented settings in this guide were checked against upstream documentation and, where
possible, against the tools themselves: Ollama's published CLI flags and its model
library, the Docker Compose file reference, the NVIDIA Container Toolkit install guide,
the Jellyfin networking documentation, and `tailscale` 1.102.3. What could **not** be
checked is any claim about how a particular piece of hardware performs, because that
depends on a machine nobody but you has. Statements about model speed and GPU/RAM fit are
expectations to test on your own server, not measurements. **Where this guide and your own
machine disagree, your machine is right.**

## Table of Contents

1. [Design and paths](#design-and-paths)
2. [Building the server](#building-the-server)
3. [Everyday use](#everyday-use)
4. [All aliases and commands](#all-aliases-and-commands)
5. [Ollama model management](#ollama-model-management)
6. [Add a new Docker container](#add-a-new-docker-container)
7. [Add a new drive](#add-a-new-drive)
8. [Add a new partition](#add-a-new-partition)
9. [Maintenance schedule](#maintenance-schedule)
10. [Backup plan](#backup-plan)
11. [Recover an individual file](#recover-an-individual-file)
12. [Recover Docker containers](#recover-docker-containers)
13. [Recover the entire OS drive](#recover-the-entire-os-drive)
14. [Future projects](#future-projects)
15. [Troubleshooting](#troubleshooting)
16. [Safety rules](#safety-rules)
17. [Quick reference](#quick-reference)

---

## Design and paths

### The shape of the setup

Fill in the right-hand column for your own machine. The left column is the part worth
copying; the right column is only the example this guide is written around.

| Item | Example setup |
|---|---|
| Operating system | Ubuntu Server LTS |
| Primary administrator account | your own login; written as `you` and `/home/you/` throughout |
| Docker data location | `/srv/docker` |
| Docker Compose files | `/srv/compose` |
| Application data | `/srv/appdata` |
| Ollama model files | `/srv/models/ollama` |
| Media library | `/srv/media` |
| Media server | Jellyfin in Docker |
| Local AI service | Ollama in Docker |
| GPU | NVIDIA GeForce RTX 3080 Ti, 12 GB VRAM |
| System memory | 64 GB RAM |
| Jellyfin LAN address | `http://192.168.1.50:8096` - **your server's own LAN address** |
| Jellyfin Tailscale address | `http://100.x.y.z:8096` - **your server's own tailnet address** |
| Jellyfin public internet access | Not configured; no router port-forward should exist |
| Backup target expected by scripts | `/mnt/backup` |

Two of those rows are addresses you have to look up rather than copy:

```bash
hostname -I          # this server's LAN address(es)
tailscale ip -4      # this server's tailnet address, which always starts 100.
```

Tailscale hands every device an address inside `100.64.0.0/10`, so a tailnet address
always begins with `100.` - but the rest is yours alone. Wherever this guide prints
`100.x.y.z`, substitute what `tailscale ip -4` reports on your server. Wherever it prints
`192.168.1.50`, substitute your own LAN address; home routers commonly hand out
`192.168.0.x`, `192.168.1.x`, or `10.0.0.x`, so yours may look quite different.

### Storage layout

```text
Ubuntu OS drive
├── /                         Ubuntu system files
├── /etc                      System settings
├── /home/you                 Administrator account files and shortcuts
└── /var/log                  System logs

Service-data NVMe mounted at /srv
├── /srv/docker               Docker images, containers, volumes, build cache, Docker logs
├── /srv/compose              Docker Compose configuration files
├── /srv/appdata              Persistent application data
│   └── /srv/appdata/jellyfin Jellyfin database, settings, cache, and transcodes
├── /srv/models/ollama        Downloaded local AI models
└── /srv/media                Movies, television, and music library
```

Putting Docker's data root on `/srv` keeps its images, layers, and build cache from
filling the OS drive. This is the single most useful structural decision in the whole
layout, and it is worth making before you pull your first image rather than after.

### Important service files

```text
/srv/compose/jellyfin/compose.yaml   Jellyfin Docker Compose project
/srv/compose/ollama/compose.yaml     Ollama Docker Compose project
/srv/appdata/jellyfin/config/        Jellyfin database, users, settings, library configuration
/srv/appdata/jellyfin/cache/         Jellyfin artwork and cache
/srv/appdata/jellyfin/transcodes/    Temporary Jellyfin transcode files
/srv/models/ollama/                  Downloaded Ollama model data
/srv/media/                          Movies, TV, and music
/srv/docker/                         Docker Engine images, layers, volumes, and build cache
/etc/docker/daemon.json              Docker data-root and NVIDIA runtime configuration
/home/you/.bashrc                    Ollama aliases and personal PATH configuration
/home/you/.local/bin/gpu-mode        GPU workload switching script
/home/you/.local/bin/backup-jellyfin-ollama  Backup script, if installed
```

The `gpu-mode` and `backup-jellyfin-ollama` scripts are this setup's own conveniences, not
software you can install from a package manager. `gpu-mode` is a small wrapper that starts
one Compose project and stops the other; `backup-jellyfin-ollama` is a `tar` of the paths
listed under [Backup plan](#backup-plan). Write your own versions, or replace them with
plain `docker compose` and `tar` commands - every section below tells you what the script
would have done, so nothing here depends on having them.

## Building the server

This section describes the state the rest of the guide assumes. Work through it once.

### Server, storage, and monitoring

Set the machine up as an always-on Ubuntu Server system with working remote command-line
access, then deliberately pull the power and confirm it comes back up on its own. An
always-on server that does not survive a power cut is a server you will have to visit.

Install the health tools:

```bash
sudo apt update
sudo apt install lm-sensors btop ncdu smartmontools
```

| Tool | What it tells you |
|---|---|
| `sensors` | CPU, RAM, NVMe, cooler, and other temperature readings. |
| `btop` | Interactive view of CPU, memory, disk, and running processes. |
| `ncdu` | Which folders are using disk space. |
| `smartctl` | SSD/NVMe health, from the `smartmontools` package. |
| `smartd` | Persistent background SMART monitoring; enable it with `sudo systemctl enable --now smartd`. |
| `nvidia-smi` | GPU status, temperature, running processes, and VRAM use; installed with the NVIDIA driver. |

Run `sensors` once while the machine is idle and write the numbers down somewhere outside
the server. That idle baseline is what makes a future reading meaningful - "58°C" means
nothing on its own, but "58°C when it idles at 27°C" means something. Do the same with
`smartctl -a` for each drive, so you have a starting point for its wear counters.

Create the two directories the rest of this guide writes into. `/srv/compose` is yours all
the way down; `/srv/appdata` is yours only at the top level, so you can add service folders.
What a container writes inside its own folder belongs to that container - reach in with
`sudo`:

```bash
sudo mkdir -p /srv/compose /srv/appdata
sudo chown "$USER:$USER" /srv/compose /srv/appdata
```

### Docker and GPU support

Install Docker Engine, the Compose plugin, Buildx, and containerd from Docker's own
repository rather than the Ubuntu archive, so you get current versions. Follow the
official instructions at <https://docs.docker.com/engine/install/ubuntu/>; they change
often enough that reproducing them here would go stale.

Point Docker's data root at the service drive by creating `/etc/docker/daemon.json`:

```json
{
  "data-root": "/srv/docker"
}
```

Do this before pulling images. Moving the data root later means stopping Docker and
copying everything across.

Add your account to the `docker` group so Docker commands work without `sudo`:

```bash
sudo usermod -aG docker "$USER"
```

Log out and back in for that to take effect. **Membership in the `docker` group is
effectively root access to the machine** - anyone in it can start a container that mounts
the whole filesystem. Do not add accounts you do not fully trust.

Then install the NVIDIA driver and the NVIDIA Container Toolkit so Docker can pass the GPU
into selected containers. The toolkit's own install guide is at
<https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/latest/install-guide.html>;
after installing it, configure the Docker runtime and restart Docker:

```bash
sudo nvidia-ctk runtime configure --runtime=docker
sudo systemctl restart docker
```

Check that a container can reach the GPU:

```bash
sudo docker run --rm --runtime=nvidia --gpus all ubuntu nvidia-smi
```

That is NVIDIA's own documented verification command. It should print the same GPU table
`nvidia-smi` prints on the host. If it errors instead, stop and fix that before going
further - every GPU section below depends on it.

### Ollama

Run Ollama in Docker as a container named `ollama`, with:

```text
/srv/compose/ollama/compose.yaml   Its Compose file
/srv/models/ollama                 Where models persist
```

Bind Ollama's API to `127.0.0.1:11434` so it is reachable from the server only. Ollama has
no authentication of its own, so a LAN-wide or tailnet-wide binding is an open,
unauthenticated service that will happily run any prompt anyone sends it. If you later
want remote access, put something in front of it that authenticates, rather than widening
this binding.

### The `ollama` and `ai-*` commands

Ollama here runs only as a Docker container. There is no `ollama` binary on the host, so a
bare `ollama ...` command fails with `ollama: command not found` until you define a wrapper
for it. The underlying form that always works is the container form:

```bash
docker exec ollama ollama list
docker exec -i ollama ollama run MODEL --hidethinking
```

In the same way, `ai`, `ai-fast`, `ai-code`, `ai-pro`, `ai-code-pro`, `ai-models` and
`ai-status` are personal shell helpers of the same kind as `gpu-mode`. They are not commands
that exist once you have followed this guide; the guide shows you what they are for, and you
write them.

One wrapper function makes every `ollama ...` command in the rest of this guide work as
written. Add it to `~/.bashrc`:

```bash
ollama() {
  if [ -t 0 ]; then
    docker exec -it ollama ollama "$@"
  else
    docker exec -i ollama ollama "$@"
  fi
}
```

`-t` allocates a pseudo-TTY, which an interactive chat needs; the non-terminal branch is what
lets a prompt be piped in instead.

The `ai-*` helpers are then one-liners over that wrapper. The `ai-strong` function in
[Install `qwen3:14b`](#install-qwen314b) is the shape to copy - substitute the model and the
name you want.

### Jellyfin

Run Jellyfin in Docker as a container named `jellyfin`, with:

```text
/srv/compose/jellyfin/compose.yaml   Its Compose file
/srv/appdata/jellyfin/config         Jellyfin database, users, settings, library configuration
/srv/appdata/jellyfin/cache          Rebuildable artwork/image/cache data
/srv/appdata/jellyfin/transcodes     Temporary transcoding files
```

Mount the media read-only, so a misbehaving container cannot write to the library:

```text
/srv/media/movies  → /media/movies
/srv/media/tv      → /media/tv
/srv/media/music   → /media/music
```

Configure Jellyfin for NVIDIA NVENC hardware transcoding, and create a second, non-admin
Jellyfin account for ordinary viewing. Watching media through the admin account means every
session is one mis-click away from a settings change.

### Tailscale

Tailscale provides private remote access without a public router port-forward. Install it
from <https://tailscale.com/download>, run `sudo tailscale up`, and note the address it
assigns:

```bash
tailscale ip -4
tailscale status
```

Jellyfin then answers on two addresses - substitute your own for both:

```text
LAN:        http://192.168.1.50:8096
Tailscale:  http://100.x.y.z:8096
```

Jellyfin's default ports are 8096/TCP for HTTP and 8920/TCP for HTTPS, and its client
auto-discovery runs on 7359/UDP. Per Jellyfin's networking documentation, auto-discovery
does not work outside your local subnet, so a client connecting over Tailscale will not
find the server by itself - add the server address manually on those clients.

Enabling remote access for an account is a **per-user** setting in Jellyfin: *Dashboard →
Users → (the user) → Allow remote connections to this server*. It is easy to look for this
under Dashboard → Networking, where the port and binding settings live, and not find it.

Leave automatic router port mapping off, and do not add a port-forward for 8096. Tailscale
is what makes remote access work here; a port-forward would put your media server on the
public internet alongside it.

## Everyday use

### Use local AI

Reserve the GPU for AI when you expect substantial LLM use:

```bash
gpu-mode ai
```

Then choose a model:

```bash
ai-fast
ai
ai-code
ai-pro
ai-code-pro
```

Example:

```bash
ai-code "Write a Bash script that checks disk usage and warns when a drive is nearly full."
```

Exit an interactive chat with:

```text
/bye
```

### Use Jellyfin

Reserve the GPU for Jellyfin when a transcode may be needed:

```bash
gpu-mode media
```

Open Jellyfin, substituting the two addresses you looked up in
[The shape of the setup](#the-shape-of-the-setup):

```text
Home LAN:  http://192.168.1.50:8096
Tailscale: http://100.x.y.z:8096
```

### Check server health

```bash
gpu-mode status
```

For live GPU/service monitoring:

```bash
gpu-mode watch
```

Press `Ctrl+C` to stop the live display.

## All aliases and commands

### GPU workload commands

| Command | What it does | When to use it |
|---|---|---|
| `gpu-mode status` | Shows Ollama/Jellyfin state, loaded AI models, GPU, RAM, and disk status. | Before workloads or while troubleshooting. |
| `gpu-mode watch` | Refreshes the status display every second. | Watch GPU usage during AI or Jellyfin transcodes. |
| `gpu-mode ai` | Stops Jellyfin and starts Ollama. | Before substantial local AI use. |
| `gpu-mode media` | Stops Ollama and starts Jellyfin. | Before media playback likely to transcode. |
| `gpu-mode both` | Starts both services. | Fine when one service is idle; avoid active AI while transcoding. |
| `gpu-mode stop` | Stops both services. | Maintenance or troubleshooting. |
| `gpu-mode help` | Shows a help screen. | Reminder of available options. |

### Ollama aliases

None of these exist until you write them, and neither does `ollama` itself - see
[The `ollama` and `ai-*` commands](#the-ollama-and-ai--commands) for why, and for the one
wrapper function that makes the rest of this table work.

| Command | Model/action | Best use |
|---|---|---|
| `ai-fast` | `qwen3.5:4b` | Fast simple questions, summaries, quick explanations, and small scripts. |
| `ai` | `qwen3:8b` | General technical questions, planning, and everyday local assistant use. |
| `ai-code` | `qwen2.5-coder:7b` | Default Bash, Python, Docker Compose, YAML, JSON, debugging, and automation help. |
| `ai-code-pro` | `qwen2.5-coder:14b` | More difficult coding, code review, debugging, and refactoring; may be slower. |
| `ai-pro` | `gemma3:12b` | Stronger analysis, careful writing, and image-aware requests. |
| `ai-models` | Lists downloaded Ollama models. | Review installed model inventory. |
| `ai-status` | Shows loaded models, GPU state, and RAM use. | Check model placement and resource use. |
| `ollama` | Your own wrapper around `docker exec ... ollama` - see [The `ollama` and `ai-*` commands](#the-ollama-and-ai--commands). | Use `ollama list`, `ollama ps`, `ollama pull MODEL`, and `ollama rm MODEL`. |

The model launch commands use `--hidethinking`. Ollama documents that flag as "Hide thinking
output (if provided)" - it suppresses the reasoning text a thinking model emits, so what
reaches your terminal is just the answer. It does not turn thinking off, and the model still
spends time on it. The separate `--think` flag is what actually controls thinking mode on
models that support it.

### Service commands

Jellyfin:

```bash
cd /srv/compose/jellyfin
docker compose ps
docker compose logs --tail=100
docker compose logs -f
docker compose restart
docker compose stop
docker compose start
docker compose up -d
```

Ollama:

```bash
cd /srv/compose/ollama
docker compose ps
docker compose logs --tail=100
docker compose logs -f
docker compose restart
docker compose stop
docker compose start
docker compose up -d
```

## Ollama model management

Every `ollama ...` and `ai-*` command in this chapter assumes the wrapper and helpers from
[The `ollama` and `ai-*` commands](#the-ollama-and-ai--commands). Without them, use the
container form directly - `docker exec ollama ollama pull MODEL`, and so on.

### Current recommended models

| Model | Alias | Role | Hardware expectation |
|---|---|---|---|
| `qwen3.5:4b` | `ai-fast` | Fast simple chat, summaries, explanations, and small scripts. | Should fit comfortably in 12 GB of VRAM. |
| `qwen3:8b` | `ai` | Default general local assistant. | Should fit in 12 GB of VRAM. |
| `qwen2.5-coder:7b` | `ai-code` | Default coding and scripting model. | Should fit comfortably in 12 GB of VRAM. |
| `gemma3:12b` | `ai-pro` | Stronger general and image-aware assistant. | Uses much of a 12 GB VRAM budget; check residency and speed. |
| `qwen2.5-coder:14b` | `ai-code-pro` | Stronger coding model. | Likely to spill into host RAM on 12 GB; test speed before relying on it. |

Every model tag above exists in Ollama's public library and can be pulled as written. The
"hardware expectation" column is reasoning about a 12 GB card, not a measurement - quantization
level, context length, and Ollama version all move the real numbers. Treat the column as a
prediction to check with `ollama ps` on your own machine, not a promise.

List installed models:

```bash
ai-models
```

Show active model placement:

```bash
ollama ps
```

Show model, GPU, and RAM status:

```bash
ai-status
```

Remove only one model:

```bash
ollama rm MODEL-NAME
```

Example:

```bash
ollama rm qwen2.5-coder:14b
```

Do not delete `/srv/models/ollama` unless the goal is to remove every local model.

### Install the recommended models

Before downloading models, reserve the GPU for AI and check space:

```bash
gpu-mode ai
df -h /srv
du -sh /srv/models/ollama
```

Pull one model at a time:

```bash
ollama pull qwen3.5:4b
ollama pull qwen3:8b
ollama pull qwen2.5-coder:7b
ollama pull gemma3:12b
ollama pull qwen2.5-coder:14b
```

After each download:

```bash
ai-models
du -sh /srv/models/ollama
```

Test a model interactively:

```bash
ollama run qwen2.5-coder:7b --hidethinking
```

In another terminal, check resource use while it is responding:

```bash
ai-status
```

### Larger models worth considering

The example machine has a 12 GB RTX 3080 Ti and 64 GB of system RAM. That makes larger models possible, but a larger model may not fit entirely in GPU memory. Scale the thresholds below to your own card: the dividing line is your VRAM, not the specific numbers here. When a model partly runs from system RAM, it can still work but is often noticeably slower.

The following are reasonable optional experiments on a 12 GB card:

| Model | Intended use | Expected fit/performance | Recommendation |
|---|---|---|---|
| `qwen3:14b` | Stronger general text model. | May need GPU/RAM split on 12 GB VRAM. | Test only if the 8B model is not sufficient. |
| `qwen2.5-coder:14b` | Stronger code and scripting model. | May use both GPU VRAM and 64 GB RAM. | Recommended larger-model experiment; already has alias `ai-code-pro`. |
| `devstral` | Agentic software-engineering/coding model. | Large for 12 GB VRAM; likely slow due to CPU/RAM offload. | Optional experiment, not a daily default. |
| `qwen3-coder:30b` | Advanced coding/agentic model. | Too large for comfortable fully GPU-resident use; substantial offload expected. | Experiment only; likely poor interactive experience. |
| `qwen3:30b` | Larger general text/reasoning model. | Too large for 12 GB VRAM; substantial offload expected. | Experiment only; not recommended for routine use. |
| `qwen2.5-coder:32b` | Large coding model. | Too large for 12 GB VRAM; substantial offload expected. | Not recommended as a normal interactive model. |

Avoid pulling many large models at once. They consume `/srv` storage and can make the service disk fill unexpectedly.

### Before installing a larger model

1. Check free service-drive capacity.
2. Stop Jellyfin or use `gpu-mode ai` to reduce GPU contention.
3. Pull only one experimental model at a time.
4. Test speed and GPU/RAM usage before retaining it.
5. Remove it if it does not provide enough value.

Commands:

```bash
gpu-mode ai
df -h /srv
docker system df
du -sh /srv/models/ollama
```

### Install `qwen3:14b`

This is the most reasonable next general-purpose larger-model test:

```bash
gpu-mode ai
ollama pull qwen3:14b
```

Test it:

```bash
ollama run qwen3:14b --hidethinking
```

Use a realistic task, for example:

```text
Review this Docker Compose design for security, persistence, networking, and maintainability. Explain any risks and propose a corrected version.
```

While it responds, run in a second terminal:

```bash
ai-status
```

Keep it if response speed is acceptable and quality is meaningfully better than `qwen3:8b`. Remove it if it is too slow:

```bash
ollama rm qwen3:14b
```

If you keep it and want an alias, add this function to `~/.bashrc` using Neovim:

```bash
nvim ~/.bashrc
```

Add:

```bash
ai-strong() {
  ollama run qwen3:14b --hidethinking "$@"
}
```

Save and reload:

```bash
source ~/.bashrc
hash -r
type ai-strong
```

Use it:

```bash
ai-strong "Explain the tradeoffs between Docker bind mounts and named volumes for a home server."
```

### Install `devstral`

`devstral` is a coding/agentic model that may be useful for exploring programming tasks. It is large relative to a 12 GB card's VRAM, so expect a GPU/RAM split and slower answers.

Install:

```bash
gpu-mode ai
ollama pull devstral
```

Test:

```bash
ollama run devstral --hidethinking
```

Example prompt:

```text
Create a step-by-step plan to review a small Python project. Identify likely bugs, missing validation, logging improvements, tests to add, and safe refactoring steps. Do not make changes; explain the plan first.
```

In another terminal:

```bash
ai-status
```

If response time is too slow, remove it:

```bash
ollama rm devstral
```

If it is useful enough to keep, add an alias:

```bash
nvim ~/.bashrc
```

Add:

```bash
ai-agent() {
  ollama run devstral --hidethinking "$@"
}
```

Reload:

```bash
source ~/.bashrc
hash -r
```

### Install `qwen3-coder:30b`

This is a large coding model. On a 12 GB card it is an experiment, not a default recommendation. Expect it to exceed GPU VRAM and use host RAM, which may make generation slow.

Install only after confirming sufficient storage and accepting the likely performance tradeoff:

```bash
gpu-mode ai
df -h /srv
ollama pull qwen3-coder:30b
```

Test:

```bash
ollama run qwen3-coder:30b --hidethinking
```

Monitor during use:

```bash
ai-status
```

Remove if the result is not worth the speed/storage cost:

```bash
ollama rm qwen3-coder:30b
```

Optional alias if retained:

```bash
nvim ~/.bashrc
```

Add:

```bash
ai-code-lab() {
  ollama run qwen3-coder:30b --hidethinking "$@"
}
```

Reload:

```bash
source ~/.bashrc
hash -r
```

### Install `qwen3:30b`

This is a large general model and is also an experiment on a 12 GB GPU.

Install:

```bash
gpu-mode ai
df -h /srv
ollama pull qwen3:30b
```

Test:

```bash
ollama run qwen3:30b --hidethinking
```

Monitor:

```bash
ai-status
```

Remove if not worthwhile:

```bash
ollama rm qwen3:30b
```

Optional alias if retained:

```bash
nvim ~/.bashrc
```

Add:

```bash
ai-lab() {
  ollama run qwen3:30b --hidethinking "$@"
}
```

Reload:

```bash
source ~/.bashrc
hash -r
```

### Install `qwen2.5-coder:32b`

This model is likely to be too slow for routine interactive use on a 12 GB GPU even with 64 GB RAM. Install only as a deliberate experiment:

```bash
gpu-mode ai
df -h /srv
ollama pull qwen2.5-coder:32b
```

Test and monitor:

```bash
ollama run qwen2.5-coder:32b --hidethinking
ai-status
```

Remove it if it is not useful:

```bash
ollama rm qwen2.5-coder:32b
```

### Model test checklist

When deciding whether to keep a large model, check:

```bash
ollama ps
nvidia-smi
free -h
df -h /srv
```

Keep a model only if:

- It produces better answers than your smaller models for your actual work.
- It responds quickly enough to be useful.
- The server has comfortable free RAM and `/srv` disk space.
- It does not interfere with your normal Jellyfin use.

Remove models that are redundant, slow, or rarely used.

## Add a new Docker container

### Basic rule

Every new service should have:

```text
/srv/compose/SERVICE-NAME/compose.yaml       Service instructions
/srv/appdata/SERVICE-NAME/                   Important persistent data, if required
```

Examples:

```text
/srv/compose/homepage/compose.yaml
/srv/appdata/homepage/

/srv/compose/uptime-kuma/compose.yaml
/srv/appdata/uptime-kuma/

/srv/compose/vaultwarden/compose.yaml
/srv/appdata/vaultwarden/
```

Do not store important configuration or databases only inside a disposable container. Use `/srv/appdata/SERVICE-NAME` bind mounts whenever practical.

### Before adding a service

1. Decide the service's purpose.
2. Decide whether access should be server-only, LAN-only, LAN-and-Tailscale, or public.
3. Decide what data must survive upgrades and container replacement.
4. Check free space.
5. Read the project's official documentation.
6. Make a backup before significant changes.

```bash
df -h /srv
docker system df
backup-jellyfin-ollama
```

### Step-by-step service creation

Replace `example-service` with the real lowercase service name.

Create folders:

```bash
mkdir -p /srv/compose/example-service
mkdir -p /srv/appdata/example-service
```

Create the file with Neovim:

```bash
nvim /srv/compose/example-service/compose.yaml
```

Safe starter pattern:

```yaml
services:
  example-service:
    image: example/image:stable
    container_name: example-service
    restart: unless-stopped
    ports:
      - "127.0.0.1:PORT:CONTAINER_PORT"
    volumes:
      - /srv/appdata/example-service:/config
```

Replace the image name, ports, and `/config` destination with values from the service's official documentation.

Save Neovim changes:

```text
Esc
:wq
Enter
```

Choose network binding:

| Need | Compose example | Meaning |
|---|---|---|
| Server-only | `"127.0.0.1:3000:3000"` | Safest default; accessible only from this server. |
| LAN plus Tailscale | `"3000:3000"` | Listens on all server interfaces. Use login/authentication and Tailscale policy. |
| LAN-only | `"192.168.1.50:3000:3000"` | Available on home LAN but not Tailscale. Use your server's own LAN address, not this example. |

Validate and start:

```bash
cd /srv/compose/example-service
docker compose config
docker compose pull
docker compose up -d
docker compose ps
docker compose logs --tail=100
```

If the service contains important persistent data, add its `/srv/appdata/example-service` folder to the backup script and update this guide.

### GPU-enabled new containers

Only services that genuinely need the NVIDIA GPU should include GPU configuration:

```yaml
gpus: all
environment:
  NVIDIA_VISIBLE_DEVICES: all
  NVIDIA_DRIVER_CAPABILITIES: compute,video,utility
```

After starting a GPU-enabled container:

```bash
docker inspect CONTAINER-NAME --format '{{json .HostConfig.DeviceRequests}}'
```

A working GPU request should not report `null`.

### Safe service removal

Stop only:

```bash
cd /srv/compose/example-service
docker compose stop
```

Start again:

```bash
docker compose start
```

Remove container but keep bind-mounted data:

```bash
docker compose down
```

Do not use `docker compose down -v` unless you understand exactly which Docker volumes will be removed. Do not delete `/srv/appdata/example-service` until a verified backup exists and the data is no longer wanted.

## Add a new drive

### Safety warning

Adding or formatting a drive can permanently erase data. Never run formatting commands until the exact drive is confirmed by device name, size, model, serial number, and purpose.

### Decide its job

| Drive purpose | Suggested mount point |
|---|---|
| More media | `/srv/media2` or `/srv/media/new-library` |
| Backup drive | `/mnt/backup` |
| Extra app data | `/srv/extra` or an app-specific path |
| Downloads/staging | `/srv/downloads` |
| General files | `/srv/data` or `/mnt/data` |

### Identify a newly connected drive

```bash
lsblk -o NAME,SIZE,TYPE,FSTYPE,LABEL,UUID,MOUNTPOINTS,MODEL,SERIAL
sudo fdisk -l
```

Common drive names:

```text
/dev/nvme0n1
/dev/nvme1n1
/dev/sda
/dev/sdb
```

Partitions commonly look like:

```text
/dev/nvme1n1p1
/dev/sdb1
```

Confirm a candidate drive:

```bash
lsblk -f /dev/sdb
```

Replace `/dev/sdb` only with the real new drive.

### Mount an existing data drive without formatting

Find the partition UUID:

```bash
sudo blkid /dev/sdb1
```

Create mount point:

```bash
sudo mkdir -p /srv/media2
```

Test mount:

```bash
sudo mount /dev/sdb1 /srv/media2
findmnt /srv/media2
ls -la /srv/media2 | head
```

Unmount if it is not correct:

```bash
sudo umount /srv/media2
```

### Prepare a new empty drive

This example uses `/dev/sdb` and erases it. Replace only after confirming the real device.

```bash
sudo parted /dev/sdb --script mklabel gpt
sudo parted /dev/sdb --script mkpart primary ext4 0% 100%
sudo mkfs.ext4 -L media2 /dev/sdb1

sudo mkdir -p /srv/media2
sudo mount /dev/sdb1 /srv/media2
sudo chown "$USER:$USER" /srv/media2

findmnt /srv/media2
df -hT /srv/media2
```

### Make drive mounting permanent

Get UUID:

```bash
sudo blkid /dev/sdb1
```

Back up and edit `/etc/fstab`:

```bash
sudo cp /etc/fstab /etc/fstab.before-new-drive-$(date +%F-%H%M%S)
sudo nvim /etc/fstab
```

Add a line like this, replacing the UUID:

```fstab
UUID=YOUR-REAL-UUID /srv/media2 ext4 defaults,nofail 0 2
```

Validate before reboot:

```bash
sudo findmnt --verify
sudo mount -a
findmnt /srv/media2
```

Test after a convenient reboot:

```bash
sudo reboot
```

### Add new media drive to Jellyfin

If a new drive is mounted at `/srv/media2` and contains movies/TV, add read-only mounts to Jellyfin Compose:

```bash
nvim /srv/compose/jellyfin/compose.yaml
```

Under `volumes:`, add:

```yaml
      - /srv/media2/movies:/media2/movies:ro
      - /srv/media2/tv:/media2/tv:ro
```

Apply:

```bash
cd /srv/compose/jellyfin
docker compose config
docker compose up -d
```

Then in Jellyfin use container paths:

```text
/media2/movies
/media2/tv
```

Do not enter host paths such as `/srv/media2/movies` in the Jellyfin web interface.

## Add a new partition

### What a partition is

A drive can be divided into separate sections called partitions. Each partition can have its own filesystem and mount point.

```text
One drive: /dev/sdb
├── /dev/sdb1 mounted at /srv/media2
└── /dev/sdb2 mounted at /mnt/backup
```

For a simple home server, one filesystem per drive is usually easier. Create multiple partitions only for a clear reason.

### Create a partition in unallocated space

Do not shrink existing OS, `/srv`, or media partitions without a verified backup and a detailed storage-layout review.

Inspect layout and free space:

```bash
lsblk -o NAME,SIZE,TYPE,FSTYPE,LABEL,UUID,MOUNTPOINTS,MODEL,SERIAL
sudo parted -l
sudo parted /dev/sdb print free
```

Create a partition only in shown free space:

```bash
sudo parted /dev/sdb
```

At the prompt:

```text
print free
mkpart primary ext4 START END
quit
```

Use actual free-space boundary values from your own output. Do not copy example sizes from another system.

Confirm new partition:

```bash
lsblk -f /dev/sdb
```

Format it only after confirming its name:

```bash
sudo mkfs.ext4 -L new-partition /dev/sdb2
```

Mount and set ownership:

```bash
sudo mkdir -p /srv/data
sudo mount /dev/sdb2 /srv/data
sudo chown "$USER:$USER" /srv/data
```

Make it permanent by adding its UUID to `/etc/fstab`, then validate:

```bash
sudo blkid /dev/sdb2
sudo cp /etc/fstab /etc/fstab.before-new-partition-$(date +%F-%H%M%S)
sudo nvim /etc/fstab
sudo findmnt --verify
sudo mount -a
findmnt /srv/data
```

## Maintenance schedule

### Daily or when needed

```bash
gpu-mode status
```

During active AI or transcoding:

```bash
gpu-mode watch
```

### Weekly

```bash
df -hT / /srv /srv/media
du -sh /srv/models/ollama
du -sh /srv/appdata/jellyfin/config
du -sh /srv/appdata/jellyfin/cache
du -sh /srv/appdata/jellyfin/transcodes
docker system df
sensors
```

Check drive health after identifying correct device names:

```bash
lsblk -o NAME,SIZE,FSTYPE,MOUNTPOINTS,MODEL,SERIAL
sudo smartctl -a /dev/nvme0n1
sudo smartctl -a /dev/nvme1n1
sudo smartctl -a /dev/nvme2n1
```

### Monthly

Update Ubuntu when services are not needed:

```bash
sudo apt update
apt list --upgradable
sudo apt upgrade
```

Reboot if kernel, NVIDIA driver, Docker, or other low-level components changed:

```bash
sudo reboot
```

After reconnecting:

```bash
nvidia-smi
systemctl is-active docker.service
systemctl is-active containerd.service
systemctl is-active tailscaled.service
gpu-mode status
```

Update Jellyfin:

```bash
cd /srv/compose/jellyfin
docker compose pull
docker compose up -d
docker compose ps
docker compose logs --tail=100 jellyfin
```

Update Ollama:

```bash
cd /srv/compose/ollama
docker compose pull
docker compose up -d
docker compose ps
docker compose logs --tail=100 ollama
```

Review Docker space before safe cleanup:

```bash
docker system df
docker image prune
```

### Quarterly

Test a planned reboot:

```bash
sudo reboot
```

After reconnecting:

```bash
systemctl is-active docker.service
systemctl is-active containerd.service
systemctl is-active tailscaled.service
docker ps
gpu-mode status
```

Test Direct Play and one forced Jellyfin transcode. During transcoding:

```bash
nvidia-smi -l 1
```

Review Tailscale Machines/access policies and remove old/untrusted devices. Confirm no router port-forward exists for port 8096.

## Backup plan

### Back up

| Item | Why |
|---|---|
| `/srv/compose/` | Compose definitions for current and future Docker services. |
| `/srv/appdata/jellyfin/config/` | Jellyfin database, users, settings, watch state, and library setup. |
| Important new `/srv/appdata/SERVICE-NAME/` folders | Future-service configuration, databases, and user data. |
| `/etc/docker/daemon.json` | Docker data root and NVIDIA runtime configuration. |
| `/etc/fstab` | Automatic drive/partition mount configuration. |
| `/home/you/.bashrc` | AI aliases, hidden-thinking behavior, and PATH setup. |
| `/home/you/.local/bin/gpu-mode` | GPU switching script. |
| `/home/you/.local/bin/backup-jellyfin-ollama` | Backup script, if installed. |
| This guide/current documentation | Makes recovery easier. |

### Do not routinely back up

| Item | Why |
|---|---|
| `/srv/media/` | Large media library; intentionally outside current backup scope. |
| `/srv/models/ollama/` | Models can be downloaded again; intentionally outside current scope. |
| `/srv/docker/` | Docker runtime data/images/layers can be recreated from Compose. |
| `/srv/appdata/jellyfin/cache/` | Rebuildable cache/artwork. |
| `/srv/appdata/jellyfin/transcodes/` | Temporary files. |

Use a separate backup disk mounted at `/mnt/backup`, another server, or another trusted destination. A backup on the same physical disk is not a real backup against disk failure.

### Run backups

```bash
backup-jellyfin-ollama
ls -lh /mnt/backup/jellyfin-ollama/
```

Suggested schedule after manual testing. Schedule it from **root's** crontab, not your own:

```bash
sudo crontab -e
```

```cron
30 2 * * * /home/you/.local/bin/backup-jellyfin-ollama >> /var/log/backup-jellyfin-ollama.log 2>&1
```

Substitute your own home directory for `/home/you`. Spell the path out rather than using
`$HOME`: this entry belongs to root, so `$HOME` here is `/root`, not the account that holds
the script. The same applies *inside* `backup-jellyfin-ollama` - under this schedule `~` and
`$HOME` are `/root` there too, so write `/home/you/.bashrc`,
`/home/you/.local/bin/gpu-mode` and `/home/you/.local/bin/backup-jellyfin-ollama` in the
`sudo tar` command rather than `~/.bashrc`, `~/.local/bin/gpu-mode` and
`~/.local/bin/backup-jellyfin-ollama`, or the nightly archive will quietly capture root's
dotfiles instead of yours.

Scheduling this from your own `crontab -e` does not work, and it fails silently. The script
archives root-owned paths - `/etc/fstab`, `/etc/docker/daemon.json`,
`/srv/appdata/jellyfin/config` - so it contains `sudo`. At 02:30 there is no terminal to
read a password from, and sudo(8) documents that case: *"a terminal is required to read the
password - sudo needs to read the password but there is no mechanism available for it to do
so."* No archive is created.

`crontab -e` under `sudo` edits root's crontab, because crontab(1) without `-u` "examines
'your' crontab, i.e., the crontab of the person executing the command". A `sudo` inside the
script is harmless when the job is already privileged. Adding a NOPASSWD `sudoers` entry for
the script is the other conventional route, but a malformed `sudoers` file can lock you out
of `sudo` entirely, so this guide does not print one.

Whichever route you pick, do not assume it worked - see [Future project: verified automated
backups](#future-project-verified-automated-backups) for the check that proves it.

### Add future services to backup

Edit backup script:

```bash
nvim ~/.local/bin/backup-jellyfin-ollama
```

Add a new important persistent path to the `sudo tar` command, for example:

```bash
  /srv/appdata/example-service \
```

Test after editing:

```bash
backup-jellyfin-ollama
```

## Recover an individual file

### 1. Confirm backup disk

```bash
mountpoint /mnt/backup
```

Expected:

```text
/mnt/backup is a mountpoint
```

### 2. List available archives

```bash
ls -lh /mnt/backup/jellyfin-ollama/
```

### 3. Inspect archive

```bash
sudo tar -tzf /mnt/backup/jellyfin-ollama/ARCHIVE-NAME.tar.gz | less
```

### 4. Extract to a safe temporary folder

```bash
mkdir -p ~/restore-check
cd ~/restore-check

sudo tar -xzf /mnt/backup/jellyfin-ollama/ARCHIVE-NAME.tar.gz \
  srv/compose/jellyfin/compose.yaml
```

Review first:

```bash
nvim ~/restore-check/srv/compose/jellyfin/compose.yaml
```

### 5. Preserve current version and restore

```bash
cp /srv/compose/jellyfin/compose.yaml \
  /srv/compose/jellyfin/compose.yaml.before-restore-$(date +%F-%H%M%S)

cp ~/restore-check/srv/compose/jellyfin/compose.yaml \
  /srv/compose/jellyfin/compose.yaml
```

### 6. Validate/apply

```bash
cd /srv/compose/jellyfin
docker compose config
docker compose up -d
```

For `.bashrc`, run `source ~/.bashrc`. For `/etc/fstab`, validate using `sudo findmnt --verify` before rebooting.

## Recover Docker containers

### Jellyfin

If container is missing but config remains:

```bash
cd /srv/compose/jellyfin
docker compose up -d
docker compose ps
docker compose logs --tail=100 jellyfin
```

If Jellyfin config is lost, stop Jellyfin, rename the damaged directory, then restore only configuration from backup:

```bash
cd /srv/compose/jellyfin
docker compose stop

sudo mv /srv/appdata/jellyfin/config \
  /srv/appdata/jellyfin/config.damaged-$(date +%F-%H%M%S)
sudo mkdir -p /srv/appdata/jellyfin/config

cd /
sudo tar -xzf /mnt/backup/jellyfin-ollama/ARCHIVE-NAME.tar.gz \
  srv/appdata/jellyfin/config
sudo chown -R "$USER:$USER" /srv/appdata/jellyfin/config

cd /srv/compose/jellyfin
docker compose up -d
```

### Ollama

```bash
cd /srv/compose/ollama
docker compose up -d
docker compose ps
docker compose logs --tail=100 ollama
ai-models
```

If models are missing, re-download only wanted models with `docker exec ollama ollama pull MODEL`.

### Future service

```bash
cd /srv/compose/SERVICE-NAME
docker compose config
docker compose up -d
docker compose ps
docker compose logs --tail=100
```

Restore `/srv/appdata/SERVICE-NAME` from backup before starting if it contains important data that was lost.

## Recover the entire OS drive

Use this only if the Ubuntu OS drive fails, cannot boot, or is replaced. Media and local models are not part of routine backup scope.

### Recovery summary

1. Install Ubuntu Server on replacement OS drive.
2. Do not format separate `/srv` or `/srv/media` data disks.
3. Identify disks/UUIDs with `lsblk -f` and `sudo blkid`.
4. Restore and validate `/etc/fstab`.
5. Install Docker and restore `/etc/docker/daemon.json`.
6. Install NVIDIA driver and NVIDIA Container Toolkit.
7. Install/authenticate Tailscale.
8. Restore Compose files, Jellyfin config, `.bashrc`, `gpu-mode`, and `backup-jellyfin-ollama`.
9. Start Ollama/Jellyfin.
10. Test LAN, Tailscale, GPU, storage, and application health.
11. Re-download Ollama models if absent.
12. Create a new backup.

### Detailed recovery

Install Ubuntu Server, create your administrator account, enable SSH, and update:

```bash
sudo apt update
sudo apt upgrade
```

Identify drives:

```bash
lsblk -f
sudo blkid
```

Create mount points:

```bash
sudo mkdir -p /srv /srv/media /mnt/backup
```

Extract prior `fstab` for inspection:

```bash
mkdir -p ~/restore-check
cd ~/restore-check
sudo tar -xzf /mnt/backup/jellyfin-ollama/ARCHIVE-NAME.tar.gz etc/fstab
nvim ~/restore-check/etc/fstab
```

Compare its UUIDs with current `sudo blkid`. If correct:

```bash
sudo cp /etc/fstab /etc/fstab.before-restore-$(date +%F-%H%M%S)
sudo cp ~/restore-check/etc/fstab /etc/fstab
sudo findmnt --verify
sudo mount -a
findmnt /srv
findmnt /srv/media
```

Install Docker, create `/etc/docker`, and restore `daemon.json`:

```bash
sudo mkdir -p /etc/docker /srv/docker /srv/compose /srv/appdata /srv/models
sudo chown "$USER:$USER" /srv/compose /srv/appdata
cd ~/restore-check
sudo tar -xzf /mnt/backup/jellyfin-ollama/ARCHIVE-NAME.tar.gz etc/docker/daemon.json
sudo cp ~/restore-check/etc/docker/daemon.json /etc/docker/daemon.json
sudo systemctl enable containerd.service docker.service
sudo systemctl restart docker
sudo usermod -aG docker "$USER"
```

Installing Docker already started the daemon, so it is running with the default data root
at this point. The restart is what makes it re-read the `daemon.json` you just restored;
without it the check below still reports `/var/lib/docker`.

Log out/back in, then check:

```bash
docker info --format 'Docker Root Dir: {{.DockerRootDir}}'
```

Expected:

```text
Docker Root Dir: /srv/docker
```

Install the NVIDIA driver. `ubuntu-drivers devices` lists the drivers Ubuntu considers
suitable for the card it detects and marks one `recommended`:

```bash
ubuntu-drivers devices
sudo ubuntu-drivers install
```

Install the same driver branch the working setup used if you know it - the version is
worth recording alongside your backups, since `nvidia-smi` cannot tell you it after the
drive is gone. Reboot if asked, then verify:

```bash
nvidia-smi
```

Install/configure NVIDIA Container Toolkit, then restart Docker:

```bash
sudo nvidia-ctk runtime configure --runtime=docker
sudo systemctl restart docker
```

Test GPU Docker access with NVIDIA's documented sample workload:

```bash
sudo docker run --rm --runtime=nvidia --gpus all ubuntu nvidia-smi
```

Install/authenticate Tailscale normally. Do not copy Tailscale machine-state files to clone the prior device.

Restore service configuration and aliases:

The paths *inside* the archive carry whatever account name the old machine used, which this
guide cannot know and which need not match the account you just created. List them and read
the name off the `home/` entries first:

```bash
sudo tar -tzf /mnt/backup/jellyfin-ollama/ARCHIVE-NAME.tar.gz | grep '^home/'
```

Use that name wherever `OLDUSER` appears below. The `srv` paths restore straight to their
own locations. The three home-directory files go through `~/restore-check` first, the same
way `/etc/fstab` and `daemon.json` did above, because their path inside the archive is the
*old* account's while `$HOME` is already correct for the account you are logged in as -
skip any of these helpers you never wrote:

```bash
OLDUSER=the-name-you-just-read

cd /
sudo tar -xzf /mnt/backup/jellyfin-ollama/ARCHIVE-NAME.tar.gz \
  srv/compose \
  srv/appdata/jellyfin/config

mkdir -p ~/restore-check
cd ~/restore-check
sudo tar -xzf /mnt/backup/jellyfin-ollama/ARCHIVE-NAME.tar.gz \
  "home/$OLDUSER/.bashrc" \
  "home/$OLDUSER/.local/bin/gpu-mode" \
  "home/$OLDUSER/.local/bin/backup-jellyfin-ollama"

mkdir -p "$HOME/.local/bin"
sudo cp ~/restore-check/home/"$OLDUSER"/.bashrc "$HOME/.bashrc"
sudo cp ~/restore-check/home/"$OLDUSER"/.local/bin/gpu-mode "$HOME/.local/bin/gpu-mode"
sudo cp ~/restore-check/home/"$OLDUSER"/.local/bin/backup-jellyfin-ollama "$HOME/.local/bin/backup-jellyfin-ollama"

sudo chown -R "$USER:$USER" /srv/compose
sudo chown -R "$USER:$USER" /srv/appdata/jellyfin/config
sudo chown "$USER:$USER" "$HOME/.bashrc"
sudo chown -R "$USER:$USER" "$HOME/.local"
sudo chmod 0755 "$HOME/.local/bin/gpu-mode"
sudo chmod 0755 "$HOME/.local/bin/backup-jellyfin-ollama"
```

Reload commands:

```bash
source "$HOME/.bashrc"
hash -r
```

Create cache/transcode/model directories if needed:

```bash
sudo mkdir -p /srv/appdata/jellyfin/cache /srv/appdata/jellyfin/transcodes /srv/models/ollama
sudo chown -R "$USER:$USER" /srv/appdata/jellyfin /srv/models
```

Start Ollama:

```bash
cd /srv/compose/ollama
docker compose config
docker compose up -d
```

Start Jellyfin:

```bash
cd /srv/compose/jellyfin
docker compose config
docker compose up -d
```

Test, re-download models if needed, and make a fresh backup.

## Future projects

These projects are optional. Add them only when there is a real use case. Before installing any project: make a backup, check free storage, read the project's official documentation, and decide whether it should be server-only, LAN-only, or available through Tailscale.

### Priority and value

| Project | What it provides | Suggested priority | Notes |
|---|---|---|---|
| Verified automated backups | Makes configuration recovery real rather than theoretical. | Highest | Set up the external backup disk, test a backup, test one-file restore, then schedule it. |
| Forced Jellyfin GPU transcode test | Confirms end-to-end NVIDIA transcoding. | Highest | Direct Play is not a transcode test. Use reduced playback quality and `nvidia-smi -l 1`. |
| MagicDNS and Tailnet policy review | Easier private addresses and tighter access control. | High | Prefer a MagicDNS name over remembering a 100.x IP. Restrict tailnet access to trusted devices. |
| Automated disk/SMART alerts | Warns early about failed disks or low space. | Medium | Decide whether alerts should go to email, a mobile notification, or a messaging service. |
| Open WebUI or another local Ollama web UI | Browser-based chat UI for local models. | Medium | Initially bind locally or protect with authentication before LAN/Tailscale access. |
| Jellyseerr | Media discovery and request workflow for Jellyfin. | Medium | Adds convenience; use a separate Compose/appdata directory and authenticated access. |
| Uptime Kuma | Simple uptime checks for Jellyfin, Ollama, Tailscale, and other services. | Low-medium | Useful after several services exist. Keep it internal or protect it carefully. |
| Docker Compose dashboard | Visual service control/logs. | Low | Convenient, but CLI/Compose remains the source of truth. Avoid exposing management dashboards broadly. |
| Tailscale HTTPS / Tailscale Serve | Friendly private HTTPS URL of the form `https://<machine>.<your-tailnet>.ts.net`. | Low-medium | Do after basic access is stable; document resulting URL and test Jellyfin clients. |
| Additional Docker services | New server capability. | As needed | Use the standard `/srv/compose` + `/srv/appdata` pattern. |

### Future project: verified automated backups

This is the most important future project because the backup plan must be tested, not merely documented.

1. Ensure a separate backup disk mounts at `/mnt/backup`.
2. Run:

```bash
mountpoint /mnt/backup
backup-jellyfin-ollama
ls -lh /mnt/backup/jellyfin-ollama/
```

3. Extract one harmless file into `~/restore-check` using the individual-file recovery instructions.
4. Only after manual backups work, schedule the nightly cron job.
5. The morning after the first scheduled run was due, confirm an archive was *really* written:

```bash
ls -lh /mnt/backup/jellyfin-ollama/
sudo tar -tzf /mnt/backup/jellyfin-ollama/ARCHIVE-NAME.tar.gz \
  | cut -d/ -f1-3 \
  | sort -u
```

   Compare the newest file's timestamp against the time the job should have run, and confirm
   the listing shows the paths you meant to archive - a scheduled backup can fail silently,
   and one written under the wrong account is the right size with the wrong contents. Do it
   again whenever you change the schedule, the script, or the account it runs as.

6. Once per quarter, test restoring a copied file or a disposable Compose file.

### Future project: automated disk and SMART alerts

Before configuring alerts, decide where alerts should be sent:

- Email address.
- Mobile push service.
- Private messaging service.
- Uptime Kuma notifications.

Do not expose a notification dashboard to the internet. Once a destination is selected, configure SMART alerting and disk-capacity checks one at a time, then test each alert deliberately.

### Future project: Open WebUI for Ollama

Open WebUI can provide a browser-based interface for local Ollama models. It is optional because terminal aliases already work.

Recommended design:

```text
/srv/compose/open-webui/compose.yaml
/srv/appdata/open-webui/
```

Safety recommendations:

- Start with a localhost-only port binding.
- Create an admin account in the web UI.
- Do not expose an unauthenticated LLM UI to the LAN or internet.
- If remote access is wanted, use Tailscale policy and authentication.
- Keep Ollama's API at `127.0.0.1:11434`; allow Open WebUI to reach it through Docker networking or a deliberate local configuration.

### Future project: Jellyseerr

Jellyseerr adds a media discovery/request workflow alongside Jellyfin.

Recommended design:

```text
/srv/compose/jellyseerr/compose.yaml
/srv/appdata/jellyseerr/
```

Safety recommendations:

- Start LAN/Tailscale-only, not public internet.
- Require login.
- Back up `/srv/appdata/jellyseerr` if it contains user requests/configuration you care about.
- Keep API keys and passwords out of shared documentation and public repositories.

### Future project: Uptime Kuma

Uptime Kuma can monitor whether Jellyfin, Ollama, and other services are available.

Recommended design:

```text
/srv/compose/uptime-kuma/compose.yaml
/srv/appdata/uptime-kuma/
```

Suggested first monitors:

- Jellyfin local HTTP endpoint: `http://jellyfin:8096` if monitored through Docker networking, or your server's own LAN address on port 8096 if that suits better.
- Ollama API: `http://ollama:11434/api/tags` through Docker networking, or local host endpoint when configured safely.
- Tailscale service/endpoint.
- Backup-target capacity if notifications are later added.

Protect the Uptime Kuma dashboard with a strong password and keep it private.

### Future project: Docker management dashboard

A management dashboard can make containers easier to see and restart, but it is not required. The Compose YAML files and CLI remain the primary source of truth.

If you add one:

- Keep it server-only or Tailscale-only.
- Require strong authentication.
- Do not expose Docker socket access to an untrusted container or user.
- Keep Compose files and `/srv/appdata` backups current.

### Future project: Tailscale HTTPS / Serve

Tailscale HTTPS and Serve can give private, browser-trusted HTTPS access without opening router ports.

Before enabling it:

1. Confirm MagicDNS works.
2. Record the exact current Jellyfin URL and server name.
3. Back up the Compose and Jellyfin configuration.
4. Test from a single Tailscale client first.
5. Confirm Jellyfin apps still connect correctly after any URL changes.

Do not treat Tailscale HTTPS as a reason to open a public router port.

## Troubleshooting

### Jellyfin will not load

```bash
cd /srv/compose/jellyfin
docker compose ps
docker compose logs --tail=200 jellyfin
curl -I http://127.0.0.1:8096
```

If it is stopped:

```bash
docker compose up -d
```

For remote access, check:

```bash
tailscale status
```

**Allow remote connections to this server** is a per-user setting, not a server-wide one:
check it under *Dashboard → Users → (the user) → Allow remote connections to this server*.
Dashboard → Networking holds the port and binding settings, which is where people look
for this first and do not find it.

### Jellyfin will not transcode

```bash
gpu-mode media
nvidia-smi
docker inspect jellyfin --format '{{json .HostConfig.DeviceRequests}}'
docker logs --tail=200 jellyfin
```

The GPU request should not be `null`. In Jellyfin, ensure NVIDIA NVENC is selected. Force a transcode because Direct Play does not use GPU transcoding.

### Ollama is slow

```bash
gpu-mode ai
ai-status
ollama ps
nvidia-smi
free -h
docker logs --tail=200 ollama
```

Use smaller models for faster interactive responses. Larger models may partly use RAM and run more slowly.

### `/srv` is nearly full

```bash
df -h /srv
docker system df
du -sh /srv/models/ollama
du -sh /srv/appdata/jellyfin/*
```

Safe first actions:

```bash
ollama rm MODEL-NAME
docker image prune
```

Do not run `docker system prune -a --volumes` casually. Do not delete `/srv/docker`, `/srv/appdata/jellyfin/config`, or `/srv/models/ollama` without understanding exactly what will be lost.

### New drive did not mount after reboot

```bash
sudo findmnt --verify
findmnt /YOUR/MOUNT/POINT
sudo journalctl -b | grep -iE 'mount|fstab|UUID'
sudo blkid
sudo nvim /etc/fstab
```

If the new `/etc/fstab` line is wrong, place `#` at the beginning of that line, validate again, and only then reboot.

### `ai` or `gpu-mode` command missing

```bash
source ~/.bashrc
hash -r
command -v gpu-mode
type ai
ls -l ~/.local/bin/gpu-mode
```

If `~/.local/bin` did not exist when you logged in, it may not be on your `PATH` yet - log
out and back in, or run this for the current session:

```bash
export PATH="$HOME/.local/bin:$PATH"
```

Restore missing scripts or shell configuration from backup if needed.

## Safety rules

1. Do not run a delete/format command you do not understand.
2. Never run `rm -rf` against `/srv`, `/srv/docker`, `/srv/appdata`, `/srv/models`, or `/srv/media`.
3. Create a backup before changing Docker, disks, partitions, `/etc/fstab`, drivers, or networking.
4. Change one thing at a time and test immediately.
5. Verify disk size, model, serial, and device name twice before formatting.
6. Do not expose Jellyfin TCP 8096 with public router port forwarding.
7. Keep Tailscale limited to trusted accounts/devices.
8. Use the Jellyfin non-admin account for watching media; reserve admin credentials for administration.
9. Use `gpu-mode ai` for significant LLM work and `gpu-mode media` for GPU transcoding.
10. Treat AI-generated commands as drafts and read them before running them.
11. Keep this guide and current backup information outside the OS drive too.

## Quick reference

```bash
# Server state
gpu-mode status

# AI
gpu-mode ai
ai
ai-code
ai-pro
ai-code-pro
ai-models
ai-status

# Media
gpu-mode media

# Live GPU view
gpu-mode watch

# Update Jellyfin
cd /srv/compose/jellyfin && docker compose pull && docker compose up -d

# Update Ollama
cd /srv/compose/ollama && docker compose pull && docker compose up -d

# Add service
mkdir -p /srv/compose/SERVICE-NAME /srv/appdata/SERVICE-NAME
nvim /srv/compose/SERVICE-NAME/compose.yaml
cd /srv/compose/SERVICE-NAME && docker compose config && docker compose up -d

# Identify drives
lsblk -o NAME,SIZE,TYPE,FSTYPE,LABEL,UUID,MOUNTPOINTS,MODEL,SERIAL

# Validate mount configuration
sudo findmnt --verify
sudo mount -a

# Health
sensors
nvidia-smi
df -hT / /srv /srv/media
docker system df

# Backup
backup-jellyfin-ollama

# Tailscale
tailscale status
```
