# Installation

## How do I install stoobly-agent?

### Option 1: Install with pipx (recommended)

**Prerequisites:**
- Python 3.12, 3.13, or 3.14

**Install pipx:**

macOS:
```bash
brew install pipx
pipx ensurepath
source ~/.bashrc
```

Linux:
```bash
python3 -m pip install --user pipx
python3 -m pipx ensurepath
source ~/.bashrc
```

**Install stoobly-agent:**
```bash
pipx install stoobly-agent --python python3
```

**Verify:**
```bash
stoobly-agent --help
```

More details: [Installation with pipx](https://docs.stoobly.com/getting-started/install-and-run/installation-with-pipx)

---

### Option 2: Install with Docker

**Prerequisites:**
- Docker installed ([official instructions](https://docs.docker.com/engine/install))

**Pull the image:**
```bash
docker pull stoobly/agent
```

**Verify:**
```bash
docker run \
    -v ~/.stoobly:/home/stoobly/.stoobly \
    stoobly/agent \
    stoobly-agent --help
```

More details: [Installation with Docker](https://docs.stoobly.com/getting-started/install-and-run/installation-with-docker)

---

## How do I update stoobly-agent?

### With pipx:
```bash
pipx upgrade stoobly-agent
```

### With Docker:
```bash
docker pull stoobly/agent
```

---

## What Python versions are supported?

Python 3.12, 3.13, and 3.14 are officially supported.

If you need to install a specific Python version, use [pyenv](https://github.com/pyenv/pyenv):

```bash
pyenv install 3.13.0
pyenv local 3.13.0
export PATH="$HOME/.pyenv/bin:$PATH"
eval "$(pyenv init --path)"
eval "$(pyenv init -)"
```

---

## How do I verify stoobly-agent is installed?

### With pipx:
```bash
stoobly-agent --help
```

### With Docker:
```bash
docker run \
    -v ~/.stoobly:/home/stoobly/.stoobly \
    stoobly/agent \
    stoobly-agent --help
```

---

## Next Steps

After installation:
- **API Mocking:** [How to Record Requests](https://docs.stoobly.com/guides/how-to-record-requests/)
- **E2E Testing:** [How to Integrate E2E Testing](https://docs.stoobly.com/guides/how-to-integrate-e2e-testing/)

