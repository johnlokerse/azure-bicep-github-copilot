# Bicep conventions

Apply every rule below to each `.bicep` and `.bicepparam` file you create or change.

**Stop and ask** means: halt that change, tell the user exactly what is missing or which rule blocks you, and wait for their answer.

## Project settings

| Setting | Value |
| --- | --- |
| Module source | `<AVM: br/public:avm/...>` or `<Internal: registry alias / template spec / path>` |
| Naming convention | `<pattern and abbreviations, or link to the naming standard>` |
| Role assignments | `<roleDefinitions() function>` or `<project abstraction: module/function that takes a role name>` |
| Private DNS zones | `<central: zone resource IDs passed as params>` or `<created in this deployment>` or `<managed by Azure Policy>` |

- A setting still showing a `<placeholder>` that your change depends on: stop and ask.

## Modules

- Take modules only from the module source in Project settings.
- No module for the resource in that source: stop and ask.
- Reference modules by exact version; reuse the version the repo already uses for that module.
- Upgrade module versions only when the user asks.
- Declare modules without a `name` property; Bicep generates the deployment name.
- Give each module a logical symbolic name derived from the module it calls, e.g. `module keyVault '.../key-vault/...'`.
- Express module dependencies implicitly through symbolic references and module outputs; leave `dependsOn` out. If no reference can express a dependency: stop and ask.

## Resources

- Resource types already in the repo: use the API version the repo uses for that type.
- Resource types new to the repo: stop and ask for the API version.
- Express dependencies implicitly through symbolic references; leave `dependsOn` out. If no reference can express a dependency: stop and ask.
- Reference resources that already exist with the `existing` keyword.
- Declare existing child resources nested inside their parent's `existing` declaration and reference them with `::`, e.g. `vnet::subnet.id`.
- Name resources per the naming convention in Project settings.

## Role assignments

- Assign roles the way Project settings names:
  - `roleDefinitions()` function: `roleDefinitionId: roleDefinitions('<role name>').id`.
  - Project abstraction: pass the role name to it; use its interface as-is.
- Keep role definition GUIDs and name-to-GUID mappings out of your code.
- Use the role's exact display name; a wrong name fails only at deployment. Not certain of the exact name: stop and ask.

## Parameters and types

- Use `.bicepparam` files only.
- Name environment parameter files `parameters.{environment}.bicepparam`, e.g. `parameters.d.bicepparam`, `parameters.t.bicepparam`.
- Declare each parameter value exactly once across all environments:
  - Shared values live in `parameters.base.bicepparam`, which starts with `using none`.
  - Each environment file starts with `using '<template the file targets>'`, e.g. `using './main.bicep'`, and `extends './parameters.base.bicepparam'`, and declares only the values that differ for that environment.
  - Override part of an inherited object or array with the spread operator on `base`, e.g. `...base.parAppConfiguration`.
  - One `extends` per file; for layered bases, every intermediate file uses `using none`.
  - Variables, types and imports declared in a base file are not inherited; redeclare them in the extending file when needed.
- Keep environment-specific values in `.bicepparam` files; template parameters for them have no default.
- Exception: default the location parameter to `resourceGroup().location`.
- Give every parameter and output a `@description`.
- Type objects and arrays that mirror a resource property with a resource-derived type, e.g. `resourceInput<'Microsoft.Storage/storageAccounts@<api-version>'>.sku`; use `resourceOutput<>` for outputs. Take the API version from the module that deploys the resource.
- Type every other parameter and output precisely: user-defined types for objects and arrays, plus `@allowed`, `@minLength`, `@maxLength`, `@minValue`, `@maxValue` where they constrain real input.
- Declare user-defined types in `types.bicep` next to `main.bicep`; create or update it, mark types `@export()` and `import` them where used.
- Declare shared user-defined functions and variables in `functions.bicep` next to `main.bicep`, exported and imported the same way.
- Write no comments (`//` or `/* */`); `@description` carries all explanation.

## Security

- Mark secret parameters `@secure()`.
- Outputs carry no secrets.
- Source secrets from Key Vault via `getSecret()` or Key Vault references.
- Set `publicNetworkAccess: 'Disabled'` and add private endpoints on every resource that supports them.
- A security rule that cannot be met for a resource: stop and ask.

## Boundaries

- Work locally only. Deployments, what-if runs and other Azure calls are outside your scope: never run, propose or ask for them.
- `bicep restore` is local tooling, not an Azure call; run it when modules need restoring.
- Change linter rule levels only in `bicepconfig.json`, and only after the user approves; keep code free of inline suppressions.
