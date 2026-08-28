# ModelSetupHub

**Install, tune and measure local AI models — at any skill level.**

Running an AI model on your own machine is possible today, but setting it up is not pleasant. You have to know how much
VRAM your graphics card has, which model fits in it, which parameters to change for better speed, and what an opaque
`llama-server` error means when nothing works. Most of that is guesswork spread across forum posts.

ModelSetupHub is an open-source toolkit that makes the process controlled and measurable. It reads your hardware,
manages the Ollama runtime, installs and runs models, and — the part most tools leave out — runs the same prompts under
different parameter sets so you can see which configuration is actually faster on your machine.

فارسی: [README.fa.md](README.fa.md)

## Four repositories, one engine

Everything real happens in **Core**. The other three are different ways to reach it, aimed at different people, and
they are interchangeable on the same machine.

| Repository | What it is | Who it is for |
| --- | --- | --- |
| [**Core**](https://github.com/ModelSetupHub/Core) | The Python library that does the work | Scripting it yourself |
| [**MCP**](https://github.com/ModelSetupHub/MCP) | An MCP server that hands the same tools to an AI agent | Describing the goal in plain language |
| [**gui**](https://github.com/ModelSetupHub/gui) | A local web dashboard over the same functions | Full manual control |
| [**Website**](https://github.com/ModelSetupHub/Website) | The static product site | Anyone deciding whether to use it |

### Core — the library

The Python package everything else is a shell around. Five modules:

- **`core.System`** — hardware discovery: CPU details, RAM modules, GPU architectures and VRAM (NVIDIA, AMD, Intel),
  disks, and a capability scanner.
- **`core.ollama`** — the Ollama daemon lifecycle, local model management (create, run, load, stop, remove), and
  parameter benchmarks with token-throughput metrics.
- **`core.python`** — virtual environment lifecycle, package management through pip, script execution, and Windows
  Python installation discovery.
- **`core.download_manager`** — a sequential multi-file downloader with pause/resume, retries, speed and ETA, and
  cancellation.
- **`core.logging`** — structured JSON logging with component-level tagging and action tracking.

```python
from core.ollama import experiment

result = experiment.run_test(
    model="llama3",
    prompts=["Hello, world!", "Summarize the solar system."],
    config={"temperature": 0.7, "num_ctx": 4096},
    name="baseline_test",
)
print("Avg output tokens/sec:", result["summary"]["average_output_tokens_per_second"])
```

### MCP — the agent interface

An MCP (Model Context Protocol) server that exposes the toolkit to an AI agent, so you can say *install a coding model
that suits my system* and let the agent choose the steps. It publishes 54 tools — 48 thin pass-throughs to core across
system, runtime, models, benchmarking, Python environments, downloads and logs, plus 6 that drive an in-chat progress
panel.

The progress panel is the one piece of original logic: an MCP Apps UI resource that reports long operations while they
run, with Cancel (which ends the operation *and* undoes what it created) and Stop for downloads (which suspends the
transfer and resumes with an HTTP range request). Some tools are deliberately withheld — `write_log`, for instance, so a
client cannot inject records into the execution history.

### gui — the web dashboard

A local Flask interface for people who already know what they want. Nothing is decided for you, which makes it faster
and more precise, but it assumes you know what a context window is. Four tabs:

- **System** — a full hardware scan; start here, because the VRAM figure decides which models you can load.
- **Ollama** — installation state and server process control.
- **Models** — installed and in-memory lists with per-row info, load, stop and remove, plus forms to run a prompt,
  register a local GGUF file, and create a configured copy of a model.
- **Benchmark** — the headline feature. Pick a model and prompts, define several parameter sets by hand from 23 Ollama
  options or by importing JSON, and run them all. Results lead with the verdict: which configuration generated fastest
  and by how much over the runner-up.

### Website — the product site

A static site (HTML and CSS, no build step) presenting the project in English and Persian.

## Which one should you start with?

- You want a graphical interface and you know what you are doing → **gui**.
- You would rather describe the goal and let an agent drive → **MCP**.
- You are writing your own automation → **Core** on its own.

`Core` is a git submodule of both `MCP` and `gui`, so either front end has to fetch it before it will run.

```bash
# the dashboard
git clone --recurse-submodules https://github.com/ModelSetupHub/gui.git
cd gui
python -m venv .venv && .venv\Scripts\activate
pip install -e ./core
pip install -r requirements.txt
python app.py            # then open http://127.0.0.1:5000/
```

```bash
# the MCP server
git clone --recurse-submodules https://github.com/ModelSetupHub/MCP.git
cd MCP
pip install -r requirements.txt
python main.py           # register it in your client under mcpServers
```

The `pip install -e ./core` step is the one that is easy to miss. Without it every request fails with
`ModuleNotFoundError: No module named 'core'`, because the app imports the submodule by package name rather than by path.

## Requirements

- Python 3.10 or newer
- [Ollama](https://ollama.com/) installed, for everything except hardware scanning
- Git, to fetch the `Core` submodule

## Things worth knowing before you install

Transparency is more useful than polish here, so these are stated plainly rather than buried:

- **Windows only, for now.** The hardware scanner is built on PowerShell and WMI, so that part does not work elsewhere yet.
- **The dashboard has no authentication.** It is a local server whose endpoints start Ollama, load and remove models,
  and execute installers on the host. Keep it on localhost; it is not built to sit on a network interface or behind a
  public reverse proxy.
- **Ollama is required** for everything except the System tab.
- **Removing a model is not recoverable** from either interface.

## Status and licence

Early but working. All four repositories are public, free and open source. Issues and pull requests are welcome on the
repository the change belongs to — behaviour changes usually belong in **Core**, since the front ends only call into it.


