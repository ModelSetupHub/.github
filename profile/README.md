# ModelSetupHub

**Install, tune and measure local AI models — at any skill level.**

English · [فارسی](README.fa.md)

Running an AI model on your own machine is possible today, but setting it up is not pleasant. You have to know how much
VRAM your graphics card has, which model fits in it, which parameters to change for better speed, and what an opaque
`llama-server` error means when nothing works. Most of that is guesswork spread across forum posts.

ModelSetupHub is an open-source toolkit that makes the process controlled and measurable. It reads your hardware, manages
the Ollama runtime, installs and runs models, and — the part most tools leave out — runs the same prompts under different
parameter sets so you can see which configuration is actually faster on your machine. Python-based models are a path of
their own: fetch a YOLO checkpoint, prepare an environment, run it from a script.

## Three ways in, one engine

Everything real happens in **Core**. The other two are different ways to reach it, aimed at different people, and they
are interchangeable on the same machine.

| Repository | What it is | Who it is for |
| --- | --- | --- |
| [**Core**](https://github.com/ModelSetupHub/Core) | The Python library that does the work | Scripting it yourself |
| [**WebApp**](https://github.com/ModelSetupHub/WebApp) | A local web dashboard over the same functions | Full manual control |
| [**MCPServer**](https://github.com/ModelSetupHub/MCPServer) | An MCP server that hands the same tools to an AI agent | Describing the goal in plain language |
| [**website**](https://github.com/ModelSetupHub/website) | The static product site | Anyone deciding whether to use it |

### Core — the library

The Python package everything else is a shell around, in five areas:

- **Hardware discovery** — processor details, memory modules, NVIDIA GPUs with their VRAM and CUDA version, and every
  drive with its free space.
- **Ollama** — the service lifecycle including installing Ollama itself, the local model lifecycle (register, run,
  preload, unload, configure, remove), and parameter benchmarks reporting token throughput.
- **Python environments** — virtual environment lifecycle, packages through pip, script execution, and finding the Python
  versions already installed. This is the path for Python-based models such as YOLO.
- **Downloads** — a queue that fetches large files one at a time, with pause, resume, retry, transfer speed and
  cancellation, restricted to a fixed list of trusted hosts.
- **Logging** — one structured execution log every operation writes to, readable and filterable afterwards.

It is a library and nothing else: no command-line tool, no server, no interface. You import it and call functions.

### WebApp — the web dashboard

A local dashboard for people who already know what they want. Nothing is decided for you, which makes it faster and more
precise, but it assumes you know what a context window is. Available in English and Persian, with full right-to-left
layout. It covers the parts of the toolkit that suit a graphical interface — the Python environment and download tools are
not among them. Four tabs:

- **System** — a full hardware scan; start here, because the VRAM figure decides which models you can load.
- **Ollama** — installation state, control of the local server process, and installing Ollama itself.
- **Models** — installed and in-memory lists with per-row info, load, stop and remove, plus forms to run a prompt,
  register a local GGUF file, and create a configured copy of a model.
- **Benchmark** — the headline feature. Pick a model and prompts, define several parameter sets by hand from 23
  generation options or by importing a JSON file, and run them all. Results lead with the verdict: which configuration
  generated fastest and by how much over the runner-up.

### MCPServer — the agent interface

An MCP server that exposes the toolkit to an AI agent, so you can say *install a coding model that suits my system* and
let the agent choose the steps. It publishes 59 tools across system, runtime, models, benchmarking, Python environments,
downloads and logs.

Long operations report themselves in the conversation: downloads and benchmarks draw a live progress bar next to the
assistant's message, with **Cancel**, which ends the operation *and* undoes what it created, and **Stop** for downloads,
which suspends the transfer and later continues from where it stopped. Some tools are deliberately withheld — writing to
the execution log, for instance, so a client cannot inject records into the history.

### website — the product site

A static site presenting the project in English and Persian.

## Which one should you start with?

- You want a graphical interface and you know what you are doing → **WebApp**.
- You would rather describe the goal and let an agent drive → **MCPServer**.
- You are writing your own automation → **Core** on its own.

`Core` is a git submodule of both front ends, so either one has to fetch it before it will run.

```bash
# the dashboard
git clone --recurse-submodules https://github.com/ModelSetupHub/WebApp.git
cd WebApp
python -m venv .venv && .venv\Scripts\activate
pip install -e ./core
pip install -r requirements.txt
python app.py            # then open http://127.0.0.1:5000/
```

```bash
# the MCP server
git clone --recurse-submodules https://github.com/ModelSetupHub/MCPServer.git
cd MCPServer
python utils/install_mshcore.py
pip install -r requirements.txt
python main.py           # then register it in your client under mcpServers
```

Installing the submodule as a package is the step that is easy to miss. Without it every call fails with
`ModuleNotFoundError: No module named 'MSHCore'`, because both front ends import it by package name rather than by file
path.

## Requirements

- Python 3.10 or newer
- Git, to fetch the `Core` submodule

Each repository pins its dependencies in `requirements.txt`, and `Core` declares them in `pyproject.toml` too, so
installing it pulls them in.

## Things worth knowing before you install

Transparency is more useful than polish here, so these are stated plainly rather than buried:

- **Windows only, for now.** Hardware detection is built on PowerShell and WMI, and GPU detection on `nvidia-smi`, so
  that part is thinner elsewhere and AMD or Intel GPUs are not reported yet.
- **The dashboard has no authentication.** It is a local server whose endpoints start Ollama, load and remove models, and
  execute installers on the host. Keep it on localhost; it is not built to sit on a network interface or behind a public
  reverse proxy.
- **The dashboard covers a subset of the toolkit.** The Python environment tools and the download manager are available
  through Core and the MCP server, not the dashboard.
- **Removing a model is not recoverable** from either interface, and cancelling a download removes the files that
  download produced.

## Status and licence

Early but working, and under active development. All four repositories are public, free and open source. Issues and pull
requests are welcome on the repository the change belongs to — behaviour changes usually belong in **Core**, since the
front ends only call into it.
