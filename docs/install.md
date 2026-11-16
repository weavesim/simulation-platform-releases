# Install, run, update, back up and remove the platform (Linux)

The platform comes as two container images. You need Docker or Podman and nothing else.

| Image | What it is | Size |
|---|---|---|
| `platform` | Modelica: the page, the API, the MCP server for agents, the WebAssembly engine and the native engine for long runs and sweeps | 288 MB |
| `platform-tools` | `platform` plus the Rumoca code generator, the C, Arm and WebAssembly compilers, QEMU, matiec, OpenPLC and Beremiz, for generating and validating controller code | 2.9 GB |

Run one or the other, never both: they use the same port and the same projects folder, so a project moves between them with no conversion. Start with `platform`; use `platform-tools` when a job tells you it needs it (`needs-tools-image`).

Nothing is sent anywhere unless you set a model server for the assistant, or turn on the check for new releases (below). There is no telemetry. The check is off until you say yes.

## Install

**Status:** the first release, 0.3.0, is out. It is an early release: only Linux with Docker has been tried (see the end of this page). To install, follow the steps below.

The platform is installed by a small program, the **installer**, that runs in a container of its own. It downloads the release's files, checks each one against the release's signed manifest, builds the image **on your machine** (so the first install takes a few minutes and needs internet: the base image and the Ubuntu packages come from their own sources), and starts it. Nothing is built from files that do not match the signature.

1. Install Docker (https://docs.docker.com/get-docker/) or Podman.
2. Download the launcher, one small file, and make it executable. Copy these three lines into a terminal:

   ```
   mkdir -p ~/bin
   curl -fsSL https://raw.githubusercontent.com/weavesim/simulation-platform-releases/main/launcher/platform -o ~/bin/platform
   chmod +x ~/bin/platform
   ```

   Without `curl`, open [`launcher/platform`](https://github.com/weavesim/simulation-platform-releases/blob/main/launcher/platform) on GitHub, press the **Download raw file** button (the arrow above the code), and save it as `~/bin/platform`. If `platform` is not found afterwards, add the folder to your PATH: `export PATH="$HOME/bin:$PATH"` (put the line in `~/.bashrc` or `~/.zshrc` to keep it). Windows has no installer commands yet.
3. Install the platform. This downloads the release, checks it, and builds the image on your machine; the first time it takes a few minutes and needs internet:

   ```
   platform update
   ```

## Start

```
platform up
```

`platform` always needs a command: on its own it prints the list of commands and options and starts nothing, so nothing happens by default (no container, no folder made).

It prints the address, `http://127.0.0.1:9091`, and where your projects are (`~/platform-projects`; choose another folder with `--volume DIR`, another port with `--port N`). Open the address in a browser. The first time, a card says where the folder is, whether the assistant has a model, and offers Test connection; Dismiss hides it.

The container runs as you, with a read-only root and no added privileges, and the port answers on this machine only. `platform --open up` publishes it on every network interface: other machines can load the page but the server refuses their API requests, and there is no encryption; for a shared server put your own TLS proxy in front.

`platform --view-port N up` also serves share links (read-only dashboards, see "Share a dashboard" in the user guide inside the page) on port N of this machine, and nothing else on that port. `--view-host 0.0.0.0` publishes it to other machines, with no encryption: put a TLS proxy in front of it before you use it outside a trusted network. `PLATFORM_VIEW_PORT` and `PLATFORM_VIEW_HOST` are the same settings in the environment.


## Your projects, and after a restart

Your projects are plain folders in the projects folder (`~/platform-projects`), not in the container. The container can be stopped, removed, replaced by a new release or restarted by the machine, and the folder is untouched. Make a project from the page (the project menu: new, from a template, or Open ZIP), have an agent make one through the MCP server, or put a folder with a `platform.toml` into the projects folder yourself with any editor; the page lists it. The examples in the page are read only: copy one to make your own. Every change made through the page or the API is saved at once as one step of the project's history, and Undo takes it back. If you change a file in the folder while the page is open, the page does not notice by itself: reload the project to see it. If the page then saves its older text, the save is refused (nothing of your edit is overwritten) and the page asks you to reload.

`platform up` starts the container with a restart policy (`unless-stopped`): after a crash, or when the machine restarts, Docker starts it again by itself, with the same folder, until you run `platform stop`. This needs Docker itself to start with the machine: on Linux `sudo systemctl enable docker` (the default on most systems); with Docker Desktop, turn on "Start Docker Desktop when you sign in". With Podman the restart policy is not enough: run the container from a systemd user unit (`podman generate systemd --new --name platform`), which is not yet tested here. With Docker Compose the same policy is in `compose.yaml`.

## Work on a server

The platform's server answers on the machine it runs on and refuses the write API from any other, on purpose. To work on a platform that runs on another machine (a server, a lab PC) as if it were local, open an SSH tunnel to it:

```
platform connect myserver        # myserver: a name ssh knows (~/.ssh/config), or user@machine
platform disconnect myserver     # close it; with no name, every tunnel
```

`connect` prints an address such as `http://127.0.0.1:9091`: open it and the page, the projects and the assistant are those of the server. The projects are on the server, not on this machine. What it needs:

- **On the server:** the platform running (`platform up`), listening on its own `127.0.0.1:9091`. If it uses another port, `platform connect myserver --remote-port N`.
- **On this machine:** `ssh` and the `platform` file, nothing else (no Docker, no local platform). If a platform runs here on port 9091 too, `connect` uses the next free port and prints it; `--port N` names one.
- **Sign-in:** the one ssh already does, with your own keys, ssh-agent and `~/.ssh/config` (ProxyJump and the rest), exactly as for `ssh myserver`. A private key works, and where the server allows it a password does too: ssh asks on the terminal, for the password, the passphrase of a key, or whether to trust a server it has not met. The answers go to ssh itself, never through the launcher, which keeps and logs nothing; the launcher never trusts a new host key by itself. There is no new password, token or certificate, and the traffic is encrypted by ssh.
- **Without questions:** `platform connect myserver --no-ask` (or `PLATFORM_CONNECT_ASK=0`) runs ssh in batch mode: it asks nothing and fails instead, which is what a script needs. Then the key must work on its own: a passphrase key loaded into ssh-agent (`ssh-add`), and a new server accepted once with `ssh myserver`.

The tunnel listens on `127.0.0.1` of this machine only. It stays open until `platform disconnect`, or ends by itself after about 90 seconds without a network. `platform.cmd` (Windows) has no `connect` yet: use `ssh -N -L 9091:127.0.0.1:9091 myserver`, or run `platform` in WSL.

This is not the `--remote` door of the live data (the guide's chapter "Live data"): that one gives other machines the live data routes only, with tokens, and no project work.

## The assistant

The assistant needs a model server that speaks the OpenAI request (Ollama, vLLM, llama.cpp, LM Studio, or the OpenAI API). Open the assistant, then Settings, and give the server's address and the model's name; the key, if any, is kept on the server and is never shown again. Without a model everything else works. To use your own Claude Code instead, `platform --claude up` mounts your `claude` program (read-only) and your sign-in folder `~/.claude` (read-write) into the container; that lets anything in the container read your sign-in, so it is off unless you ask for it. See `platform/docs/assistant-setup.md`.

## Agent clients (MCP)

`platform mcp-config` prints the configuration for an MCP client such as Claude Code. It starts `docker run -i --rm ... platform mcp`, which runs the same tools against your projects folder with no network.

## Update

```
platform check             is there a newer release, and what would it ask?
platform update            install the newest release
platform rollback          go back to the release before
```

`platform update` runs the installer, which, in this order: reads the signed manifest and refuses it if its signature is not one of the keys built into the program; downloads the files and refuses any that does not match the manifest; **builds the new image beside the running one** (nothing changes until it is built); copies `.platform` (settings, sessions, secrets) to `.platform/backups/`; stops the running container and starts a new one with **the same settings** (the read-only root, the port, the volume, your `--claude` or `--view-port`); waits until it is healthy; and **puts the old one back** (and `.platform`) if it is not. The old release stays as the image `platform:previous`, which is what `platform rollback` starts.

**Your projects are never changed by an update.** A release that changes how a project is stored says so first, and your projects then open read only until you migrate them, one at a time: [migrate.md](migrate.md).

A release that changes an interface, needs a migration or brings new tools is not installed until you say yes (`--yes`); a release that is too old to update to directly is refused (update in steps, as its notes say). Other options: `--patch-only`, `--to X.Y.Z`, `--no-snapshot`, `--check` (the same as `platform check`), and `--tools` for the platform-tools image.

### Hearing about a new release

The check is **off until you turn it on**. The first-run card asks once. When it is on, once a day the server reads two small files from the release site (a manifest and its signature): no cookie, no identifier, nothing about you or your projects. A notice then shows on the page, with what the release asks of you and a link to its notes. `PLATFORM_UPDATE_CHECK=0` (or `--no-update-check` on the server) turns it off for good. `platform check` asks once, when you run it, whatever the setting.

### Updating by itself

```
platform update --install-timer     every day, by itself
platform update --remove-timer
```

This installs a systemd user timer (on macOS a launchd agent) that runs `platform --auto update` every day. **It installs only a patch release of the one you have, or a security release, and only one that changes no interface, needs no migration and brings no new tools.** Everything else is shown by the page and by `platform check` and waits for you. It **never interrupts your work**: before it switches it asks the running platform whether it is busy (a job running, a tool call, a change being written), waits up to six hours (`--max-wait`), and otherwise leaves the switch to the next night. The log is `.platform/update-timer.log` in your projects folder. A systemd user timer runs while you are logged in; on a server nobody logs into, run once `loginctl enable-linger <your user>`.

### Behind a firewall, or from a mirror

A site that cannot reach the release site can mirror it. Copy the manifest files (`stable/latest.json`, `stable/latest.json.sig`, and `versions/` if you use `--to`) to a web server of your own and set `PLATFORM_MANIFEST_URL=https://mirror.example/releases`; copy the release files to one folder and set `PLATFORM_FILES_URL=https://mirror.example/releases/files` (each file is then fetched from there, by its name). The manifest is **signed** and holds each file's SHA-256, so a mirror cannot change a release: it can only serve it, and a changed manifest or file is refused. The base image and the packages are fetched by the engine from Docker Hub and Ubuntu's mirrors; to use your own, configure the engine (a registry mirror, an apt proxy) as you would for any build. `PLATFORM_INSTALLER_IMAGE` names another installer image, `PLATFORM_ENGINE_SOCKET` the engine's socket (Podman: `systemctl --user start podman.socket`, and the launcher finds `$XDG_RUNTIME_DIR/podman/podman.sock`).

### Checking a release by hand

Each release has `SHA256SUMS` and a signature of it, and the manifest has a signature too (`latest.json.sig`). The public keys are in the program and in the release repository (`cosign.pub`). With `cosign` installed:

```
cosign verify-blob --key cosign.pub --signature latest.json.sig --insecure-ignore-tlog=true latest.json
cosign verify-blob --key cosign.pub --signature SHA256SUMS.sig --insecure-ignore-tlog=true SHA256SUMS
sha256sum -c SHA256SUMS
```

The images you build are not signed: what is signed are the files they are built from, and the installer checks those before it builds. The scan and the SBOM published with a release describe the image that the release's maintainers built from the same files at release time; yours may have newer Ubuntu packages, because it is built on your machine against the current ones (`platform update` after a new toolchain release, or when you want fresh packages).

### If an update fails

| What you see | What it means | What to do |
|---|---|---|
| `does not match the signed manifest` | a downloaded file is not the one that was signed | nothing was changed; try again, or check your mirror |
| `does not match any key of this build` | the manifest is not signed by a key this program trusts | nothing was changed; do not use that release site |
| `this update needs your yes` | the release changes an interface, needs a migration or brings new tools | read its notes, then `platform update --yes` |
| `the build failed, so nothing was changed` | a step of the build failed (a mirror down, no disk) | the running platform is untouched; try again |
| `the old container was put back and is running again` | the new one was not healthy | nothing is lost; report the message with `platform logs` |
| the page says a project opens read only | the release changed the project format | [migrate.md](migrate.md) |

## Back up and restore

Your projects, the assistant's settings, its sessions and the secrets file are plain files in the projects folder. Copy the folder to back it up, and copy it back to restore. The secrets file holds the model server's key (mode 600), so a backup holds the key too.

## Stop and remove

`platform stop` removes the container and leaves your projects folder. To remove the platform completely: `platform stop`, then `docker rmi platform platform-tools`, then delete the folder if you no longer want your projects.

## If something goes wrong

| What you see | What to do |
|---|---|
| `platform: Docker or Podman is needed` | Install one of them. |
| The port is in use | `platform --port 9092 up`. |
| It did not become healthy | `platform logs`. |
| A job says `needs-tools-image` | Run `platform stop`, then `platform --tools up`. |
| The folder is not yours or not writable | The launcher runs the container as your user: make the folder yours (`chown -R $(id -u) DIR`). |
| The assistant says it has no model | Settings, then the model's address and name; Test connection. |

## What is tested

Linux (x86-64) with Docker is tested. Podman, macOS and Windows (`platform.cmd`) are written but not yet tested; arm64 images are not built yet.
