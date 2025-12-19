# Common Strong Type Patterns

Ready-to-use type definitions for common Azure resources.

## Tags Parameter

```bicep
// Flexible: any string keys
type tagsType = {
  *: string
}
param tags tagsType?

// Strict: known tags only
@sealed()
type strictTagsType = {
  environment: 'dev' | 'staging' | 'prod'
  costCenter: string
  owner: string
  application: string?
}
param tags strictTagsType
```

## Key Vault Access Policies

```bicep
type keyPermissionsType = ('get' | 'list' | 'create' | 'update' | 'delete' | 'backup' | 'restore' | 'recover' | 'purge')[]
type secretPermissionsType = ('get' | 'list' | 'set' | 'delete' | 'backup' | 'restore' | 'recover' | 'purge')[]

type accessPolicyType = {
  tenantId: string
  objectId: string
  permissions: {
    keys: keyPermissionsType?
    secrets: secretPermissionsType?
  }
}

param accessPolicies accessPolicyType[]
```

## Virtual Network Subnets

```bicep
type subnetType = {
  name: string
  addressPrefix: string
  privateEndpointNetworkPolicies: ('Disabled' | 'Enabled')?
  privateLinkServiceNetworkPolicies: ('Disabled' | 'Enabled')?
  serviceEndpoints: {
    service: string
    locations: string[]?
  }[]?
  delegations: {
    name: string
    serviceName: string
  }[]?
}

param subnets subnetType[]
```

## Diagnostic Settings

```bicep
type diagnosticSettingType = {
  name: string?
  workspaceResourceId: string?
  storageAccountResourceId: string?
  eventHubAuthorizationRuleResourceId: string?
  eventHubName: string?
  logCategoriesAndGroups: {
    category: string?
    categoryGroup: string?
    enabled: bool?
  }[]?
  metricCategories: {
    category: string
    enabled: bool?
  }[]?
}

param diagnosticSettings diagnosticSettingType[]?
```

## Role Assignments

```bicep
type roleAssignmentType = {
  principalId: string
  roleDefinitionIdOrName: string
  principalType: ('Device' | 'ForeignGroup' | 'Group' | 'ServicePrincipal' | 'User')?
  description: string?
  conditionVersion: '2.0'?
  condition: string?
}

param roleAssignments roleAssignmentType[]?
```

## Private Endpoints

```bicep
type privateEndpointType = {
  name: string?
  subnetResourceId: string
  privateDnsZoneResourceIds: string[]?
  service: string
  customNetworkInterfaceName: string?
  tags: object?
}

param privateEndpoints privateEndpointType[]?
```

## Nested Type Composition

Build complex types from simpler ones:

```bicep
type addressType = {
  street: string
  city: string
  country: string
  postalCode: string?
}

type contactType = {
  name: string
  email: string
  phone: string?
  address: addressType?
}

type organizationType = {
  name: string
  contacts: contactType[]
}

param organization organizationType
```
