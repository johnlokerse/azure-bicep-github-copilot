# Resource-Derived Types

Use `resourceInput<>` and `resourceOutput<>` to derive types directly from Azure resource schemas.

## Syntax

```bicep
resourceInput<'<resourceType>@<apiVersion>'>   // Writable properties
resourceOutput<'<resourceType>@<apiVersion>'>  // Readable properties (includes computed)
```

## When to Use

- Parameter must match resource property structure exactly
- Need compile-time validation against Azure schema
- Want to avoid maintaining custom types that mirror resource properties

## Examples

### Full Properties Block

```bicep
param storageAccountProps resourceInput<'Microsoft.Storage/storageAccounts@2023-01-01'>.properties = {
  accessTier: 'Hot'
  minimumTlsVersion: 'TLS1_2'
  allowBlobPublicAccess: false
  supportsHttpsTrafficOnly: true
}

resource storageAccount 'Microsoft.Storage/storageAccounts@2023-01-01' = {
  name: 'mystorageacct'
  location: resourceGroup().location
  sku: { name: 'Standard_LRS' }
  kind: 'StorageV2'
  properties: storageAccountProps
}
```

### Specific Property Types

```bicep
// Storage account kind
type accountKind = resourceInput<'Microsoft.Storage/storageAccounts@2024-01-01'>.kind
// Results in: 'BlobStorage' | 'BlockBlobStorage' | 'FileStorage' | 'Storage' | 'StorageV2'

// Key Vault SKU
type keyVaultSku = resourceInput<'Microsoft.KeyVault/vaults@2023-07-01'>.properties.sku

// Virtual network address space
param addressSpace resourceInput<'Microsoft.Network/virtualNetworks@2023-11-01'>.properties.addressSpace

// Key Vault access policies
type accessPoliciesType = resourceInput<'Microsoft.KeyVault/vaults@2023-07-01'>.properties.accessPolicies
```

### Output Types

```bicep
output endpoints resourceOutput<'Microsoft.Storage/storageAccounts@2024-01-01'>.properties.primaryEndpoints
```

## Finding Resource Types and Versions

Use MCP tools:
- `Bicep:list_az_resource_types_for_provider` - List all types for a provider
- `Bicep:get_az_resource_type_schema` - Get full schema for a resource type

## resourceInput vs resourceOutput

| Aspect | resourceInput<> | resourceOutput<> |
|--------|-----------------|------------------|
| Properties | Writable only | Readable (includes computed) |
| Use case | Parameters, variables for input | Output declarations |
| Example | accessTier, sku | provisioningState, endpoints |
