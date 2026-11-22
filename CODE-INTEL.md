# Code intelligence for the agent: Rarefaction, Serena, graphify

Left alone, a coding agent understands a codebase by grepping and reading whole files. That
is slow, fills the context window, and misses callers that a text search does not find. Three
MCP servers give it what an IDE gives you. Each is good at a different job:

| tool | its job | ask it |
|---|---|---|
| **Rarefaction** | one symbol: its callers (tests marked), callees, the tests that reach it, the files a change can affect | "what calls this and what would break?" |
| **Serena** | navigating and editing symbols: find symbol, references, diagnostics, rename, replace a body | "find it, check it, change it safely" |
| **graphify** | the whole-repo map: clusters, central "god" nodes, how two things connect, docs linked to code | "how is this repo organized? how does A reach B?" |

All three use language servers or tree-sitter. They are only as good as the project's index,
so **check them on your project before trusting them** (see "Verify" below).

## Rarefaction

LSP call-graph server: one `orient` call returns a symbol's signature, doc, body, callers up
to two levels, callees, implementations, the tests that reach it with a "run only these"
command, and the live files. `scope` adds the files that historically change together with it
(from git history). Profiles: Swift, Rust, C/C++, TypeScript/JavaScript, Python, WebAssembly
text. Source and docs: <https://github.com/ip2k/rarefaction> (GPL-3.0-or-later; by
the author of this guide).

Install (needs Node 22+; TypeScript and Python language servers are bundled. Install
`rust-analyzer`, `clangd`, `sourcekit-lsp` yourself if you need them):

```bash
claude plugin install rarefaction --marketplace ip2k/rarefaction
```

(That is the form in the project's README. If your Claude Code version rejects it, run
`/plugin` in a session, add the marketplace `ip2k/rarefaction`, and install from there.)

or as a bare MCP server:

```bash
claude mcp add --scope user --transport stdio rarefaction -- npx -y rarefaction-lsp-mcp
```

If `rust-analyzer` or other servers live outside the client's PATH, add
`--env PATH="$HOME/.local/bin:$HOME/.cargo/bin:$PATH"` before the `--`.

Notes from using it:
- Its three tools (`status`, `orient`, `scope`) are read-only. Allow them in your
  permissions so you are not prompted every call.
- Xcode projects need `xcode-build-server` and a fresh build for the cross-file index.
  C/C++ needs `compile_commands.json`.
- In TypeScript monorepos with a `tsconfig.json` per package, callers in *other* packages
  are not listed (the TS language server answers one project at a time). It warns you.
  Search across packages separately.
- `scope` is advisory. On real commits it found about half of the files that actually
  changed. Treat it as "read these first", not "only these matter".
- The first call per session pays the language server's cold start (seconds to tens of
  seconds on big projects). After that, calls take well under a second.

## Serena

Symbol-level navigation and editing over LSP, many languages.
Docs: <https://oraios.github.io/serena/>. Needs `uv` (`brew install uv` or see uv's docs).

```bash
claude mcp add --scope user serena -- uvx --from git+https://github.com/oraios/serena \
  serena start-mcp-server --context=claude-code --project-from-cwd
```

The `claude-code` context switches off Serena tools that duplicate Claude Code's own file
and shell tools. At the start of a session the agent should call Serena's
`initial_instructions`, and check with `get_current_config` that the **active project is
the directory you are actually working in** (worktrees especially). A project can carry
`.serena/project.yml` listing its languages.

## graphify

Builds a knowledge graph of a repo (code through tree-sitter; optionally docs, PDFs and images
through an LLM pass). PyPI package `graphifyy` (note the double y), Apache-2.0.

```bash
uv tool install graphifyy   # then, in a repo:
graphify update .           # code-only graph, local, seconds
graphify explain <name>     # what is this, what touches it
graphify path <A> <B>       # how A connects to B
graphify query "<question>"
```

Notes:
- Edges marked `INFERRED` come from name matching. Treat them as leads, not proof.
- The full `/graphify .` pipeline sends docs through an LLM and costs tokens. Know the cost
  before running it. `graphify update` is free and local.
- Its installers offer to add hooks to your projects and a block to your global
  `CLAUDE.md`. We declined both and keep `graphify-out/` in the global git ignore. Decide
  for yourself, but do it on purpose.

## How they fit together (put this in your CLAUDE.md)

> Start broad questions ("where/how/what connects") with graphify if the repo has a graph.
> As soon as a question names a symbol, `orient` it with Rarefaction before reading the file,
> and `scope` it before editing. Use Serena for symbol navigation, diagnostics and
> structured edits. An empty answer is not proof of no callers: dynamic dispatch,
> reflection (`getattr`, `importlib`), generated bindings and FFI are invisible to all
> three. Grep before a risky edit.

## Verify on your project (5 minutes, once per project)

1. Rarefaction `status`: `root` is the project you are in and the profile says `ok: true`.
2. Pick a function you know has callers and a test. `orient` it and compare with what you
   know. Did it find the test?
3. Serena `find_referencing_symbols` on the same function. Do the two agree?
4. Edit the file, save, and query again. Does the answer reflect the edit (index freshness)?
5. Write the result in the project's `CLAUDE.md`: what works, what is missed, and any
   config needed (`.rarefaction.json`, `.serena/project.yml`, build-server config).

When a tool returns nothing, the usual cause is the wrong root, an unbuilt index or a
missing language server. It does not mean "no callers". Run `status` and fix the setup.
