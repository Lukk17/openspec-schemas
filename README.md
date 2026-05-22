# openspec-schemas

Community [OpenSpec](https://github.com/Fission-AI/OpenSpec) schemas for spec-driven AI workflows.

## Schemas

| Schema | Purpose |
| --- | --- |
| [e2e-runbooks](./e2e-runbooks/) | Capability-level e2e test suite: spec + tasks template + run records, behaviour-only assertions, per-run token + duration accounting. |

## Install (any schema)

OpenSpec has no schema-install CLI yet, so install is a manual copy. From your consumer project's root:

```bash
git clone --depth 1 https://github.com/Lukk17/openspec-schemas /tmp/lukk17-schemas
```

```bash
cp -r /tmp/lukk17-schemas/<schema-name> openspec/schemas/
```

```bash
rm -rf /tmp/lukk17-schemas
```

```powershell
git clone --depth 1 https://github.com/Lukk17/openspec-schemas $env:TEMP\lukk17-schemas
```

```powershell
Copy-Item -Recurse $env:TEMP\lukk17-schemas\<schema-name> openspec\schemas\
```

```powershell
Remove-Item -Recurse -Force $env:TEMP\lukk17-schemas
```

Then either pass `--schema <schema-name>` to `/opsx:new`, or set `default_schema: <schema-name>` in
`openspec/config.yaml`.

## Contributing

Each schema is its own folder at the repo root with the standard OpenSpec bundle layout (`schema.yaml`, `README.md`,
optional `INTEGRATION.md`, `templates/`). Open a PR against `master`. Validate locally with
`openspec schema validate <schema-name>` before submitting.

## License

MIT, see [LICENSE](./LICENSE).
