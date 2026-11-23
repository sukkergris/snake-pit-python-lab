# CLAUDE.md

## Projekt

Et lab til at lære og eksperimentere, i en devcontainer (Debian trixie slim, se `.devcontainer/debian/`).

- `notebooks/`: Jupyter notebooks. Python 3.14 styres med `uv` (`pyproject.toml`, `uv.lock`, `.venv`). Brug `uv add`/`uv sync`, ikke `pip`.
- Dotfiles ligger i `~/dotfiles` og bliver installeret med `stow`.

## Sådan arbejder vi

- Svar på dansk. Kort og direkte.
- Forklar årsagen først og derefter løsningen. Find den rigtige årsag i logs eller filer i stedet for at gætte.
- Når et problem i miljøet er løst, skriv det under "Kendte problemer" nedenfor.
- Spørg, før du ændrer Dockerfile eller devcontainer-konfiguration, da det kræver rebuild.
- Commit kun, når jeg beder om det.

## Kendte problemer i devcontainer

### "Invalid Python interpreter" for `notebooks/.venv`

VS Code (ms-python.python) afviser `notebooks/.venv/bin/python` med "An Invalid Python interpreter is selected". I Python-loggen (`~/.vscode-server/data/logs/*/exthost*/ms-python.python/Python.log`) står der `Invalid version ... "micro":-1`.

**Årsag:** `uv` bygger din venv oven på en mappe med kun minor-version (`~/.local/share/uv/python/cpython-3.14-linux-x86_64-gnu`). Derfor står der kun `version_info = 3.14` i `.venv/pyvenv.cfg`. Python-extension kræver major.minor.micro.

**Løsning:** skriv den fulde version i `pyvenv.cfg`, og kør derefter **Developer: Reload Window**:

```sh
cd notebooks
v=$(.venv/bin/python -c 'import platform;print(platform.python_version())')
sed -i "s/^version_info = .*/version_info = $v/" .venv/pyvenv.cfg
```

Linjen bliver nulstillet, hver gang din venv bliver lavet på ny (`uv venv`, eller hvis du sletter `.venv` og kører `uv sync`). Kør da kommandoen igen. Løsningen er ikke nødvendig længere, når Python-extension kan håndtere versioner uden patch-nummer.

### Locale-advarsel fra perl (fx `stow`)

`perl: warning: Setting locale failed` er harmløs. Base image er `debian:trixie-*-slim` (se `.devcontainer/debian/Dockerfile.debian`), og den har ikke pakken `locales`. Derfor findes kun `C`, `C.utf8` og `POSIX`, mens `LANG=en_US.UTF-8` bliver sendt ind. Perl bruger så `C` i stedet.

**Løsninger (ingen er brugt endnu):**

- `export LANG=C.UTF-8` i shell eller Dockerfile.
- Eller installér locale i Dockerfile:

  ```dockerfile
  RUN apt-get update && apt-get install -y --no-install-recommends locales \
      && sed -i 's/^# *en_US.UTF-8/en_US.UTF-8/' /etc/locale.gen \
      && locale-gen \
      && rm -rf /var/lib/apt/lists/*
  ENV LANG=en_US.UTF-8 LC_ALL=en_US.UTF-8
  ```

## Editor

Ruff formaterer Python-filer og notebooks, når du gemmer (se `.vscode/settings.json`). Ruff-extension har sin egen Ruff indbygget.
