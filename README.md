# Declank (`declank`)

A small Node wrapper around the
[`remove-ai-watermarks`](https://github.com/wiltodelta/remove-ai-watermarks)
CLI, plus a web UI that shells out through the local `declank` command.

The Node project does not remove watermarks itself. It stages uploads in a temp
directory, invokes the external CLI with whitelisted arguments, and streams the
result back for preview and download.

## Prerequisites

Install the Python CLI first:

```bash
pipx install git+https://github.com/wiltodelta/remove-ai-watermarks.git
# or
uv tool install git+https://github.com/wiltodelta/remove-ai-watermarks.git
```

The wrapper expects the executable to be named `remove-ai-watermarks`. If it is
installed somewhere else, set:

```bash
export REMOVE_AI_WATERMARKS_CLI=/path/to/remove-ai-watermarks
```

Visible watermark removal and metadata stripping run on CPU. Invisible watermark
removal depends on the upstream CLI's optional GPU dependencies.

## Install

```bash
npm install
```

## CLI

```bash
node bin/imgx.js <input> [options]
# or, after `npm link`: declank <input> [options]
```

`clearframe` and `imgx` are still published as legacy aliases.

Options:

| Flag | Description |
|------|-------------|
| `-o, --output <path>` | Output path (default: `<input>.clean.<ext>`) |
| `-m, --mode <mode>` | `visible` \| `metadata` \| `invisible` \| `all` \| `erase` \| `identify` |
| `--mark <mark>` | `auto` \| `gemini` \| `doubao` \| `jimeng` \| `samsung` |
| `--region <x,y,w,h>` | Region for `erase` mode; repeatable |
| `--backend <backend>` | `cv2` \| `lama` for `erase` mode |
| `--strength <n>` | Invisible watermark denoising strength |
| `--steps <n>` | Invisible watermark denoising steps |
| `--pipeline <name>` | `default` \| `controlnet` |
| `--device <name>` | `auto` \| `cpu` \| `mps` \| `cuda` \| `xpu` |
| `--max-resolution <px>` | Cap long side before diffusion |
| `--json` | JSON output for `identify` mode |

Examples:

```bash
declank image.png -o clean.png
declank image.png --mode metadata -o clean.png
declank image.png --mode all -o clean.png --max-resolution 2048
declank image.png --mode erase --region 1640,1930,400,100 -o clean.png
declank image.png --mode identify --json
```

Pass extra upstream CLI flags after `--`:

```bash
declank image.png --mode invisible -o clean.png -- --auto --restore-faces
```

## Web UI

```bash
npm start             # http://localhost:3333
PORT=8080 npm start   # custom port
```

The web UI supports PNG, JPEG, and WebP uploads up to 25 MB. It exposes these
modes:

| Mode | Upstream command |
|------|------------------|
| Visible marks | `remove-ai-watermarks visible` |
| Metadata only | `remove-ai-watermarks metadata --remove` |
| All signals | `remove-ai-watermarks all` |
| Invisible marks | `remove-ai-watermarks invisible` |

Set `DECLANK_TIMEOUT_MS` if invisible or all-signal processing needs more
than the default 10 minute server timeout. `CLEARFRAME_TIMEOUT_MS` and
`IMGX_TIMEOUT_MS` are still supported.

## Architecture

```text
public/index.html   upload UI
server.js           Express; POST /api/transform -> execFile(node bin/imgx.js)
bin/imgx.js         local wrapper around remove-ai-watermarks
src/transform.js    shared mode validation and CLI argument builder
```

Use this only on files you have the rights and permissions to process, and make
sure downstream use complies with applicable laws and platform terms.
