## ManagedNamespace: ${{ values.namespaceName }}

**XR Name:** `${{ values.xrName }}`

**Created by:** @${{ values.userName }} (${{ values.userEmail }})

### RBAC Configuration

**Owners:**
{%- for owner in values.owners %}
- {{ owner }}
{%- endfor %}

{%- if values.maintainers and values.maintainers.length > 0 %}

**Maintainers:**
{%- for maintainer in values.maintainers %}
- {{ maintainer }}
{%- endfor %}
{%- endif %}

{%- if values.readers and values.readers.length > 0 %}

**Readers:**
{%- for reader in values.readers %}
- {{ reader }}
{%- endfor %}
{%- endif %}

---

🤖 Generated with Backstage
