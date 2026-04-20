# Editable Template Creation — Static Template → `/conf` Editable Template

**Agent:** The parent skill loads this file when the user asks to create editable templates from existing static templates. This is a **prerequisite** for running `aem-modernization.md` — structure rewrite rules reference editable templates that must already exist under `/conf`.

No BPA pattern ID required. The agent discovers inputs by reading the project's static templates and page structure components.

---

## What This Pattern Does

Generates the JCR node tree for each editable template under:
```
ui.content/.../jcr_root/conf/<appId>/settings/wcm/templates/<templateName>/
```

Each editable template is a 4-node set:

| Node | File | Purpose |
|------|------|---------|
| Template root | `.content.xml` | Declares the `cq:Template`, title, status, allowed content paths |
| `structure/` | `structure/.content.xml` | Page structure with responsive grid layout — what authors see in the template editor |
| `initial/` | `initial/.content.xml` | Content copied into new pages created from this template |
| `policies/` | `policies/.content.xml` | Policy mapping tree — maps container nodes to content policies |

Also updates the parent `templates/.content.xml` index node to register each new template name.

---

## Prerequisites — Read the Project First

**STOP. Before generating any file, complete this discovery checklist.**

### Discovery checklist

1. **App ID:** Find `<appId>` from `/apps/<appId>/` paths in `ui.apps`. Confirm via `appId` in `ui.apps/pom.xml` or filevault plugin config.

2. **Static templates:** Glob `**/jcr_root/apps/<appId>/templates/**/.content.xml`. For each, record:
   - Template name (folder name)
   - `jcr:title` from `jcr:content`
   - `allowedPaths` from the root node
   - `sling:resourceType` from `jcr:content` (this is the page structure component)

3. **Page structure components:** For each static template's `sling:resourceType`, read the component at `ui.apps/.../apps/<appId>/components/structure/<templateName>/`. Note any parsys slots, fixed zones, or named child includes in the HTL files — these map to child nodes in `structure/.content.xml`.

4. **Template type:** Check `**/jcr_root/conf/<appId>/settings/wcm/template-types/` — find the template type node (usually `page`). The path will be `/conf/<appId>/settings/wcm/template-types/page`. Confirm it exists.

5. **Existing editable templates:** Glob `**/jcr_root/conf/<appId>/settings/wcm/templates/**/.content.xml`. Do not recreate templates that already exist — only generate missing ones.

6. **App content container resourceType:** Find the project's responsive grid container — the component that extends `wcm/foundation/components/responsivegrid` or `core/wcm/components/container`. Typically `<appId>/components/content/container`. Verify by searching `ui.apps` for a component with `layout="responsiveGrid"`.

7. **Breakpoints:** Check an existing editable template's `structure/.content.xml` for the `<cq:responsive>` breakpoints. If found, use the same values for all new templates (do not invent new breakpoints).

8. **Targeting / fixed structural children:** Read each structure component's HTL files for `data-sly-resource` calls that reference non-parsys children (e.g. `targeting`, `LiveSyncConfig`, `header`). These named nodes must either be:
   - Present in `initial/.content.xml` as a pre-placed child of `jcr:content` (for required runtime nodes like `targeting`)
   - Or excluded from both `structure` and `initial` if the editable template's page component no longer renders them

8. **Allowed paths pattern:** For each static template, check if `allowedPaths` is set. Use it directly in the editable template root. If absent, derive from the template's known content subtree (e.g. `/content/<appId>(/.*)?`). Ask the user to confirm if the pattern is not obvious.

Report findings before generating files. Ask the user to confirm if any static template lacks a corresponding structure component, or if the template type path cannot be found.

---

## File 1: Template root — `.content.xml`

**Path:**
```
ui.content/src/main/content/jcr_root/conf/<appId>/settings/wcm/templates/<templateName>/.content.xml
```

**Format:**
```xml
<?xml version="1.0" encoding="UTF-8"?>
<jcr:root xmlns:jcr="http://www.jcp.org/jcr/1.0" xmlns:cq="http://www.day.com/jcr/cq/1.0"
    jcr:primaryType="cq:Template"
    allowedPaths="[<allowedPathsPattern>]">
    <jcr:content
        cq:templateType="/conf/<appId>/settings/wcm/template-types/page"
        jcr:primaryType="cq:PageContent"
        jcr:title="<humanTitle>"
        status="enabled"/>
</jcr:root>
```

**Field mapping:**
| Field | Source |
|-------|--------|
| `allowedPaths` | Copied from static template's `allowedPaths`, or derived from content path pattern |
| `cq:templateType` | Always the discovered template-types path (`/conf/<appId>/settings/wcm/template-types/page`) |
| `jcr:title` | Copied from static template's `jcr:content/jcr:title` |
| `status` | Always `"enabled"` for active templates |

**Do not include** `cq:lastModified` or `cq:lastModifiedBy` — these are author-set timestamps, not source-controlled.

---

## File 2: Structure node — `structure/.content.xml`

The structure defines the fixed page layout authors see in the template editor. It mirrors the static template's page component structure.

**Path:**
```
ui.content/src/main/content/jcr_root/conf/<appId>/settings/wcm/templates/<templateName>/structure/.content.xml
```

**Format:**
```xml
<?xml version="1.0" encoding="UTF-8"?>
<jcr:root xmlns:jcr="http://www.jcp.org/jcr/1.0"
          xmlns:nt="http://www.jcp.org/jcr/nt/1.0"
          xmlns:cq="http://www.day.com/jcr/cq/1.0"
          xmlns:sling="http://sling.apache.org/jcr/sling/1.0"
    jcr:primaryType="cq:Page">
    <jcr:content
        cq:deviceGroups="[/etc/mobile/groups/responsive]"
        cq:template="/conf/<appId>/settings/wcm/templates/<templateName>"
        jcr:primaryType="cq:PageContent"
        sling:resourceType="<appId>/components/structure/<templateName>">
        <root
            jcr:primaryType="nt:unstructured"
            sling:resourceType="wcm/foundation/components/responsivegrid">
            <responsivegrid
                jcr:primaryType="nt:unstructured"
                sling:resourceType="<appId>/components/content/container"
                editable="{Boolean}true"
                layout="responsiveGrid"/>
        </root>
        <cq:responsive jcr:primaryType="nt:unstructured">
            <breakpoints jcr:primaryType="nt:unstructured">
                <phone
                    jcr:primaryType="nt:unstructured"
                    title="Smaller Screen"
                    width="{Long}768"/>
                <tablet
                    jcr:primaryType="nt:unstructured"
                    title="Tablet"
                    width="{Long}1200"/>
            </breakpoints>
        </cq:responsive>
    </jcr:content>
</jcr:root>
```

**Key rules:**
- `cq:template` points back to the template's own path (self-reference)
- `sling:resourceType` on `jcr:content` = `<appId>/components/structure/<templateName>` — must match the page structure component discovered in step 3
- `<root>` uses `wcm/foundation/components/responsivegrid` — the WCM layout container
- `<responsivegrid>` inside root uses the **app's own content container** (`<appId>/components/content/container`) with `editable="{Boolean}true"` — this marks it as the author-editable zone
- `<cq:responsive>` breakpoints: use values discovered from existing templates; default to phone=768, tablet=1200 if none found
- `cq:deviceGroups="[/etc/mobile/groups/responsive]"` — always include on structure

**If the static template has multiple parsys / named zones:**
Read the structure component HTL for `sling:include` / `data-sly-resource` calls. Each named child parsys that should remain editable gets its own child node inside `<root>`:
```xml
<leftpar
    jcr:primaryType="nt:unstructured"
    sling:resourceType="<appId>/components/content/container"
    editable="{Boolean}true"
    layout="responsiveGrid"/>
```
Nodes that are fixed (header, footer, navigation) are **not** marked `editable="{Boolean}true"`.

**If the template has a locked (always-present) component alongside the editable zone:**
When the page structure component always renders a specific component (not a parsys) that must exist on every page — including pages created before the template was deployed — place it as a sibling to `<responsivegrid>` inside `<root>` **without** `editable`. Do **not** put it in `initial/.content.xml` — initial content is only copied to new pages, so existing pages would lose the component if it were moved there:
```xml
<root jcr:primaryType="nt:unstructured"
    sling:resourceType="wcm/foundation/components/responsivegrid">
    <fixedcomponent jcr:primaryType="nt:unstructured"
        sling:resourceType="<appId>/components/content/<componentName>"/>
    <responsivegrid jcr:primaryType="nt:unstructured"
        sling:resourceType="<appId>/components/content/container"
        editable="{Boolean}true"
        layout="responsiveGrid"/>
</root>
```

---

## File 3: Initial content node — `initial/.content.xml`

The initial content is copied into every new page created from this template. It should be structurally identical to `structure` but without `editable`, `cq:deviceGroups`, and `<cq:responsive>`.

**Path:**
```
ui.content/src/main/content/jcr_root/conf/<appId>/settings/wcm/templates/<templateName>/initial/.content.xml
```

**Format:**
```xml
<?xml version="1.0" encoding="UTF-8"?>
<jcr:root xmlns:jcr="http://www.jcp.org/jcr/1.0"
          xmlns:nt="http://www.jcp.org/jcr/nt/1.0"
          xmlns:cq="http://www.day.com/jcr/cq/1.0"
          xmlns:sling="http://sling.apache.org/jcr/sling/1.0"
    jcr:primaryType="cq:Page">
    <jcr:content
        cq:template="/conf/<appId>/settings/wcm/templates/<templateName>"
        jcr:primaryType="cq:PageContent"
        sling:resourceType="<appId>/components/structure/<templateName>">
        <root
            jcr:primaryType="nt:unstructured"
            sling:resourceType="wcm/foundation/components/responsivegrid">
            <responsivegrid
                jcr:primaryType="nt:unstructured"
                sling:resourceType="<appId>/components/content/container"
                layout="responsiveGrid"/>
        </root>
    </jcr:content>
</jcr:root>
```

**Differences from `structure`:**
- No `cq:deviceGroups`
- No `editable="{Boolean}true"` on any child node
- No `<cq:responsive>` breakpoints block

**Pre-placed components:**
For templates where new pages must start with a specific component already present (e.g. a single-artifact template where the primary component is always required), add the component as a child of `<responsivegrid>` in initial content:
```xml
<responsivegrid jcr:primaryType="nt:unstructured"
    sling:resourceType="<appId>/components/content/container"
    layout="responsiveGrid">
    <primarycomponent jcr:primaryType="nt:unstructured"
        sling:resourceType="<appId>/components/content/<componentName>"/>
</responsivegrid>
```
Only pre-place components that belong in the editable zone. Components locked in `structure` must **not** be duplicated here.

**Required runtime nodes (e.g. `targeting`):**
If the page structure component renders a named child via `data-sly-resource` that is not a parsys (discovered in checklist step 8), add it as a direct child of `jcr:content` in initial — not inside `<root>`:
```xml
<jcr:content ...>
    <targeting jcr:primaryType="nt:unstructured"
        sling:resourceType="<appId>/components/content/<targetingComponent>"/>
    <root ...>
        ...
    </root>
</jcr:content>
```

---

## File 4: Policies mapping node — `policies/.content.xml`

The policies node maps each container in the template to a content policy. For new templates this is a minimal placeholder — actual policy assignments are done after deployment via the template editor or can be copied from an existing template.

**Path:**
```
ui.content/src/main/content/jcr_root/conf/<appId>/settings/wcm/templates/<templateName>/policies/.content.xml
```

**Format:**
```xml
<?xml version="1.0" encoding="UTF-8"?>
<jcr:root xmlns:jcr="http://www.jcp.org/jcr/1.0"
          xmlns:nt="http://www.jcp.org/jcr/nt/1.0"
          xmlns:cq="http://www.day.com/jcr/cq/1.0"
          xmlns:sling="http://sling.apache.org/jcr/sling/1.0"
    jcr:primaryType="cq:Page">
    <jcr:content
        jcr:primaryType="nt:unstructured"
        sling:resourceType="wcm/core/components/policies/mappings">
        <root
            jcr:primaryType="nt:unstructured"
            sling:resourceType="wcm/core/components/policies/mapping">
            <responsivegrid
                jcr:primaryType="nt:unstructured"
                sling:resourceType="wcm/core/components/policies/mapping"/>
        </root>
    </jcr:content>
</jcr:root>
```

**If copying an existing policy reference:** If the user has an existing template with a `cq:policy` value (e.g. `wcm/foundation/components/responsivegrid/policy_31297318674600`) on the `<root>` node, that policy can be referenced here. Only copy it if the user explicitly asks to inherit the same policy — otherwise leave the mapping empty (no `cq:policy` attribute).

---

## File 5: Templates index — `templates/.content.xml`

The parent `templates/` folder node lists all template names as empty child elements. **Read the existing file first** and add only the new template names.

**Path:**
```
ui.content/src/main/content/jcr_root/conf/<appId>/settings/wcm/templates/.content.xml
```

**Format (existing entries preserved, new names appended):**
```xml
<?xml version="1.0" encoding="UTF-8"?>
<jcr:root xmlns:jcr="http://www.jcp.org/jcr/1.0"
          xmlns:rep="internal"
          xmlns:cq="http://www.day.com/jcr/cq/1.0"
    jcr:mixinTypes="[rep:AccessControllable]"
    jcr:primaryType="cq:Page">
    <rep:policy/>
    <existing-template-1/>
    <existing-template-2/>
    <new-template-name/>
</jcr:root>
```

**Rule:** Preserve every existing `<childName/>` entry. Only append new ones. Do not remove or reorder existing entries.

---

## Packaging in `ui.content`

Confirm the `filter.xml` for `ui.content` includes the templates subtree:

```xml
<filter root="/conf/<appId>/settings/wcm/templates"/>
```

If missing, add it. Also verify the template-types path is filtered if it was just created:

```xml
<filter root="/conf/<appId>/settings/wcm/template-types"/>
```

---

## Relationship to Modernization Rules

Once editable templates are created and deployed, **run `aem-modernization.md`** to generate the structure rewrite rules that reference them. The structure rewrite rule's `editableTemplate` property must exactly match the path used in `cq:template` in the template's `.content.xml`.

The complete migration sequence is:

```
1. [This skill]      Create editable templates → ui.content/.../conf/.../templates/
2. [aem-modernization.md]  Create structure/component/policy rewrite rules → ui.apps + ui.config
3. [Manual]          Deploy both packages (or via Cloud Manager pipeline)
4. [Manual]          Run AEM Modernize Tools UI jobs against content paths
```

---

## Output summary (report to user)

```
Editable templates created:
  <templateName>  →  conf/<appId>/settings/wcm/templates/<templateName>/
    .content.xml         (cq:Template root, status=enabled)
    structure/           (page layout with responsive grid)
    initial/             (initial page content)
    policies/            (policy mapping placeholder)

templates/.content.xml   updated — added: <list of new template names>
filter.xml               <added entry / already present>

Skipped (already exist):
  <list of templates that were found and not overwritten>

Review required:
  <list any templates where structure component was not found>
  <list any templates where allowedPaths could not be determined>
```

---

## Critical rules

- **Read before writing** — complete the discovery checklist before creating any file
- **Do not overwrite** existing editable templates — check for existence first
- **`sling:resourceType` on `jcr:content`** must match the actual structure component path — wrong value causes blank page rendering
- **`editable="{Boolean}true"`** is required on the editable container in `structure` — without it, authors cannot edit the zone in the template editor
- **Do not add `editable` to `initial`** — it belongs only on `structure`
- **Do not invent breakpoint values** — copy from an existing template or ask the user
- **`cq:template` is a self-reference** — it points to the template's own `/conf` path, not to the static template
- **Do not create the template type** — `template-types/page` must already exist in the project; ask the user if it is missing
- **Preserve `templates/.content.xml`** — read it first, add new entries only, never delete existing ones
