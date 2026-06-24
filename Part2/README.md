# Exercise 2: Property-Driven Training for Drug Dosing

This exercise trains and formally verifies a neural network controller for an antibiotic dispenser. Choose one of the three setup options below, then open `vancomycert_example.ipynb`. Runtime depends and varies between 40 - 90 minutes dependent on your training hyperparameters!

---

## Option A: Google Colab (no install required)

Click the badge at the top of `vancomycert_example.ipynb` to open it directly in Colab. 

Change Runtime > Change Runtime type > 2025.07 (for Python3.11)

The first cell installs all dependencies automatically. If marabou fails for some reason it will build from source compatible with Python3.13 however, takes a long time (30 minutes)

**Requirements:** a Google account on Colab CPU.

---

## Option B: Linux lab machines

Two sub-options depending on your preference.

### B1: Install uv and Python 3.11 (self-contained)

```bash
curl -LsSf https://astral.sh/uv/install.sh | sh
source $HOME/.local/bin/env
hash -r
uv python install 3.11
uv venv --python 3.11 .venv
source .venv/bin/activate
uv pip install -e .
```

### B2: Use the lab's native Python 3.13 (Marabou pre-built)

Marabou has been compiled from source on the lab machines and is available in the native Python 3.13 environment. However, has had some issues with the vehicle backend compataibility. Recommended to proceed with B1 if on lab machine, or option C.

```bash
python3.13 -m venv .venv
source .venv/bin/activate
pip install -e .
```

## Option C: Own machine

**Requirements:** Python ≥ 3.11.

```bash
python3.11 -m venv .venv
source .venv/bin/activate
pip install -e .
```

---

## Running in a code editor (VS Code / JetBrains)

After installing, register the venv as a Jupyter kernel so the editor can find it:

```bash
source .venv/bin/activate
python -m ipykernel install --user --name vancomycert --display-name "python (vancomycert)"
```

Then open `vancomycert_example.ipynb`, click the kernel picker in the top-right, and select **python (vancomycert)**.
