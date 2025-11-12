# Managed Namespace Template

Backstage scaffolder template for creating Kubernetes namespaces with automated RBAC configuration via Crossplane.

## Overview

This template creates a `ManagedNamespace` Crossplane composite resource (XR) that automatically provisions:

- **Kubernetes Namespace**: A dedicated namespace for your team/project
- **RBAC Configuration**: Automated role-based access control with three permission levels:
  - **Owners**: Full control including namespace deletion (minimum 2 required)
  - **Maintainers**: Create/edit/delete resources within the namespace
  - **Readers**: Read-only access to namespace resources

## Features

- ✅ **Entra ID Integration**: Validates users against Microsoft Entra ID (Azure AD)
- ✅ **Case-Sensitive Email Handling**: Preserves exact email casing from Entra ID
- ✅ **GitOps Workflow**: Creates PR or commits directly to catalog-orders repository
- ✅ **Kubernetes Built-in Roles**: Uses standard `admin`, `edit`, and `view` ClusterRoles
- ✅ **XR-Level Permissions**: Owners can manage the ManagedNamespace resource itself

## User Journey

### 1. **Configure Namespace**
   - Enter namespace name (e.g., `team-platform`)
   - Add at least 2 owners (validated against Entra ID)
   - Optionally add maintainers and readers

### 2. **Select Repository**
   - Choose target GitHub repository (typically `catalog-orders`)
   - Specify target path (default: `system/ManagedNamespace`)
   - Choose PR creation or direct commit

### 3. **Review & Create**
   - Template validates all users against Entra ID
   - Generates unique XR name
   - Creates ManagedNamespace XR manifest
   - Publishes to GitHub

### 4. **Automatic Provisioning**
   - GitHub Action triggers on PR merge/commit
   - Crossplane reconciles the ManagedNamespace XR
   - Kubernetes namespace created with RBAC

## Required Backstage Configuration

### Secrets

Add these secrets to your Backstage `app-config.yaml`:

```yaml
secrets:
  AZURE_TENANT_ID: ${AZURE_TENANT_ID}
  AZURE_CLIENT_ID: ${AZURE_CLIENT_ID}
  AZURE_CLIENT_SECRET: ${AZURE_CLIENT_SECRET}
```

### Custom Scaffolder Actions

This template requires custom scaffolder actions:

1. **`openportal:entra:validate-users`**: Validates users against Microsoft Entra ID
2. **`portal:utils:generateId`**: Generates unique identifiers
3. **`kubernetes:wait`**: Monitors Kubernetes resource readiness (optional)

See `app-portal/packages/backend/src/scaffolder/` for implementation.

### Custom UI Field

The template uses a custom UI field for Entra ID user selection:

- **`EntraIdUserPicker`**: Autocomplete field that searches Entra ID users

This field should be implemented in `app-portal/packages/app/` as a custom scaffolder field extension.

## Example XR Output

```yaml
apiVersion: openportal.dev/v1alpha1
kind: ManagedNamespace
metadata:
  name: team-platform-a1b2c3d4
  labels:
    managed-by: backstage
    namespace-name: team-platform
spec:
  name: team-platform
  owners:
    - alice.admin@openportal.dev
    - bob.manager@openportal.dev
  maintainers:
    - charlie.developer@openportal.dev
  readers:
    - dave.viewer@openportal.dev
```

## RBAC Permissions

### Owners
- **ClusterRole**: Custom role allowing XR management
  - Verbs: `get`, `list`, `watch`, `patch`, `update`, `delete`
  - Resources: `managednamespaces`
- **Namespace RoleBinding**: Bound to `admin` ClusterRole
  - Full control within the namespace

### Maintainers
- **Namespace RoleBinding**: Bound to `edit` ClusterRole
  - Create, update, delete most resources
  - Cannot modify RBAC or quotas

### Readers
- **Namespace RoleBinding**: Bound to `view` ClusterRole
  - Read-only access to most resources
  - Cannot view secrets

## Crossplane Configuration

This template works with the following Crossplane configuration:

- **XRD**: [template-namespace/configuration/xrd.yaml](https://github.com/open-service-portal/template-namespace/blob/main/configuration/xrd.yaml)
- **Composition**: [template-namespace/configuration/composition.yaml](https://github.com/open-service-portal/template-namespace/blob/main/configuration/composition.yaml)

These must be deployed to your cluster before using this template.

## Development

### Testing Locally

1. Start Backstage:
   ```bash
   cd app-portal
   yarn start
   ```

2. Navigate to: http://localhost:3000/create/templates/managed-namespace

3. Fill in the form and create a test namespace

### Modifying the Template

- **Template metadata**: Edit `template.yaml`
- **XR manifest**: Edit `content/managednamespace.yaml`
- **Parameters**: Modify `template.yaml` parameters section
- **Steps**: Update `template.yaml` steps section

## Troubleshooting

### Users Not Found in Entra ID

- Verify `AZURE_TENANT_ID`, `AZURE_CLIENT_ID`, `AZURE_CLIENT_SECRET` are correct
- Ensure the service principal has `User.Read.All` permission in Microsoft Graph
- Check that email addresses match Entra ID exactly

### Template Not Appearing

- Verify repository name matches pattern: `service-*-template`
- Check that `template.yaml` exists in repository root
- Wait for GitHub provider refresh (30 minutes) or trigger manually

### XR Not Created

- Check GitHub Action logs in the catalog-orders repository
- Verify Crossplane XRD and Composition are deployed
- Check Crossplane controller logs: `kubectl logs -n crossplane-system deployment/crossplane`

## Related

- **Crossplane XRD**: [template-namespace](https://github.com/open-service-portal/template-namespace)
- **Catalog Orders**: [catalog-orders](https://github.com/open-service-portal/catalog-orders) repository for XR instances
- **Backstage App**: [app-portal](https://github.com/open-service-portal/app-portal) for custom scaffolder actions

## License

MIT
