# chicago/module-template — a template for modules of the Chicago shell

A GitHub template repository for a Wippy module that adds windows, widgets or
games to the Chicago shell of the terminal desktop, in the look of the mid-nineties desktops
([chicago/shell](https://github.com/chicago-desktop/shell) on
[chicago/tui-desktop](https://github.com/chicago-desktop/tui-desktop)). Copy it,
run `make init`, and you have a working, tested module with one sample window
on the shell's SDK — ready for `make test` and `make publish`. It plays for
shell modules the role the Kickside module template plays for platform
modules: the identity scripts, the checks, the harness and the Makefile are
the same shape, adapted to a module whose surface is a window, not a page.

The sample is **Hello Window** (Start → Programs → Module Template): a menu
(File → Exit, Help → About), a line of text, a button that counts its clicks
and shows the count, a status bar, and a picture of its own in the menu and
the title bar. Small on purpose: every part of it is something a new window
needs, and the tests show how each part is checked.

## Three minutes to a module of your own

1. **Copy the repository** — "Use this template" on GitHub, or
   `git clone https://github.com/chicago-desktop/module-template my-module`.
2. **Name it** — once, from the pristine copy:

   ```bash
   make init ORG=acme MODULE_NAME=notes TITLE="Notes"
   ```

   `TITLE` names the menu folder and the harness, so it takes no slashes or
   colons; a window title with them ("Add/Remove Programs") is set in the
   entry's `meta.title` afterwards. The GitHub owner defaults to
   `chicago-desktop` for the Hub organization `chicago` (`GITHUB_OWNER=` for
   another).

   This renames `chicago/module-template`, the namespace
   `chicago.module_template` and the title everywhere (sources, tests,
   harness, Makefile), writes a README for the module, and records the
   identity in `.kickside-module.json`. Optional: `NAMESPACE=acme.work.notes`,
   `TAG=acme-notes`, `GITHUB_OWNER=acme-dev`. It refuses to rename an
   initialized checkout to another identity.
3. **Resolve the dependencies** — `make setup` writes `wippy.lock` for the
   module and for the harness (`chicago/shell` and `chicago/tui-desktop`,
   resolved from their GitHub repositories by tag — v0.2.0 is the first —
   and the runtime modules the harness boots).
4. **Run the tests** — `make test`, then look at `test/shots/hello.png`: the
   window as the shell's own renderer drew it, after two clicks.
5. **Write your window** — edit `src/view.lua` (the window as data),
   `src/window.lua` (the process) and the entry in `src/_index.yaml`; read
   [docs/sdk.md](docs/sdk.md) first. Draw your picture into
   `assets/images/{32,16}/<name>.png` and name it `<namespace>:images/<name>`.
   Keep the tests in step: `make lint`, `make test`.
6. **Publish** — `wippy auth login` once, then `make publish` (public by
   default; `make publish VIS=private` otherwise). A published version is
   immutable: the next `make publish` bumps it.

`make check` runs the repository's invariants; `make verify` is setup, check,
lint and test together, what CI runs.

## Anatomy of a module

- `wippy.yaml` — the module's identity for the Hub (organization, module,
  description, licence), what is excluded from the package (the harness), and
  `embed:`, the list of `fs.directory` entries that ship with it — the image
  pack is one.
- `src/_index.yaml` — the registry: the namespace, the dependencies on
  `chicago/shell` and `chicago/tui-desktop`, the image pack, the `view`
  library and the `window` process with its `meta.type: tui_desktop.window`
  entry (title, menu group, picture, size, pixel renderer).
- `src/view.lua` — the window as data: `init`, `tree` (the component tree of
  a model) and `update` (what an action does); pure, so the tests exercise it
  without a compositor.
- `src/window.lua` — the process: `app.main{init, view, update}` on the shell's
  SDK, delegating to `view`. Thin on purpose.
- `assets/images/32/hello.png`, `assets/images/16/hello.png` — the image pack,
  one picture at the two sizes the shell asks for; `tools/hello_icon.py` draws
  it (`make icons`).
- `test/wippy.yaml`, `test/.wippy.yaml` — the harness: a tiny application
  that boots the module (replaced with `..`) together with the shell, resolved
  from its GitHub repository by tag; the overrides give the base its shell environment and the shell its
  user group.
- `test/src/_index.yaml` — the harness's registry: the host resources the
  shell's boot needs (a database, a process host, a gateway on :19239, a
  permissive scope, the fonts, the shots folder) and one `meta.type: test`
  entry per test file.
- `test/src/view_test.lua` — the initial tree, the click count, Exit, About,
  the layout in cells and pixels without overlaps, and the shot.
- `test/src/window_test.lua` — the registry entry the Start menu reads, the
  picture found through the shell's `images.get`, the process running the
  view, Esc closing it.
- `tools/late-locals.py` — finds a `local` declared below the function that
  reads it; `wippy lint` does not.
- `scripts/` — `init-module.mjs` (the rename), `check-module.mjs` (the
  invariants), `test-initializer.mjs` (the rename's round trip, run in the
  pristine template only).
- `docs/sdk.md`, `skills/wippy-window-app/SKILL.md`,
  `.claude/skills/wippy-window-app/SKILL.md` — copies of the shell's SDK guide
  and of its skill for agents, as of `chicago/shell` 0.1.0; the shell's are
  canonical.
- `Makefile`, `make.ps1`, `make.bat` — the same targets on Linux, macOS and
  Windows.

## Requirements

- **A build of the runtime fork**
  [chicago-desktop/runtime](https://github.com/chicago-desktop/runtime), branch
  `wippy-projects`. The shell declares the `gfx` module (pixels in the
  terminal), which the release runtime does not have — and a release `wippy`
  does not load the shell at all; it says only
  `node with ID {gfx :gfx} not found`. Build with
  `CGO_CFLAGS="-I<dir with sqlite3.h>" make build-wippy-local` in the fork
  and point the Makefile at `dist/wippy-linux-amd64`: `make test WIPPY=…`, or
  edit the default at the top of the Makefile.
- **fonts-liberation** — the pixel theme reads Liberation Sans as bytes at
  run time, and the harness reads it for the shot
  (`/usr/share/fonts/truetype/liberation/`, the `system_fonts` entry).
- **node** (22 or newer) for the scripts, **python3** for the tools.

## Where the module shows up in an application

An application that depends on the module and runs the shell sees it without
any wiring of its own:

- **The Start menu** — the window's entry with `meta.type: tui_desktop.window`
  lands in the folder its `group` names; a module's windows go under
  `Programs/<Module>` ([docs/sdk.md, "Quick start"](docs/sdk.md#quick-start-entry--menu--window)).
  The entry adds the program to the menu; a desktop shortcut is the person's
  own data.
- **The tray** — a service of the module may put an item next to the clock
  with the compositor's `desktop.tray` command (the weather module does).
- **Desktop widgets** — a `process.lua` with `meta.type: chicago.widget` is a
  panel at the right edge of the desktop, drawn from the same kind of tree
  ([docs/sdk.md, "Desktop widgets"](docs/sdk.md#desktop-widgets)).
- **Pictures** — the image pack is found in the registry when a picture is
  asked for; a redrawn file shows without a restart.

## Traps

- **A `local` declared below the function that reads it is a nil global.**
  No error: the value is just `nil`, and the symptom looks like anything else
  ("attempt to call a non-function object", an empty line, a window that
  stopped answering). `tools/late-locals.py` catches it; `make lint` runs it
  first. Declare everything a function calls above that function.
- **An unquoted `: ` in a YAML comment or a `meta.comment` breaks the whole
  index** — the file fails to parse, and with it the module's boot. Quote a
  comment that contains a colon followed by a space.
- **A test file in the wrong form is green without running.** A file that
  puts `test.describe` inside a `run` function and returns it is counted,
  printed green in under a millisecond, and executes nothing. The form is
  `local run_cases = test.run_cases(define_tests)` …
  `return {run = function(options) return run_cases(options) end}`; break a
  new test on purpose once and see it go red.
- **`wippy publish` packs only `src/`.** An `fs.directory` outside it (the
  image pack) ships only when `wippy.yaml` lists it under `embed:`;
  `make check` verifies every pack is there.
- **`${env:…}` in a registry entry resolves against the environment
  registry, not the OS.** `exec` does not inherit the OS environment either;
  the harness hands the base `HOME` and `PATH` explicitly in `test/.wippy.yaml`.
- **A local build only.** A release `wippy` answers "clean" to `wippy lint` on
  code that uses `gfx` because it does not know the types; it verifies
  nothing there.

## Licence

MIT. The sample's picture is drawn by `tools/hello_icon.py`; there is no
third-party artwork in this repository.
