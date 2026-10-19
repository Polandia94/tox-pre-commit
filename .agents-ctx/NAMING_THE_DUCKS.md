# Naming the Ducks

Canonical names for this project, and how to refer to it in
prose. Other files in `.agents-ctx/` (and any generated content)
should say "the project" rather than hardcoding one of these --
reach for a specific name only when the distinction actually
matters (e.g. citing a package, an import, or a repository URL).

## Distribution / PyPI name: `tox-pre-commit`

The installable, published name -- recorded in
`pyproject.toml`'s `[project].name` per PEP 621. This is also
the preferred way to refer to the project by name in prose.

## Python import name: `tox_plugins.pre_commit`

The plugin lives in the `tox_plugins` PEP 420 implicit namespace
package, shared with other `tox_plugins.*` distributions. Nothing
outside tox imports it directly: tox finds it through the
`tox-pre-commit` entry point in the `tox` group, declared in
`pyproject.toml`'s `[project.entry-points.tox]`.

## GitHub repository slug: `tox-dev/tox-pre-commit`

Use it verbatim when it's actually the repository being
referenced (URLs, `gh` invocations, PR/issue citations) -- not as
a stand-in for the project's name in prose.

## Don't say bare `pre-commit`

`pre-commit` on its own is ambiguous here. It's the upstream tool
the plugin wraps, the name of the tox env the plugin registers,
and the name of the env this repository's own `tox.ini` defines
for linting itself. Say which one is meant -- e.g. "the
`pre-commit` tox env", "the `pre-commit` tool" -- and say
`tox-pre-commit` for this project.
