---
name: blender-addon-scaffolder
description: >
  Scaffold a new Blender Python addon for Rockson with his preferred file structure, bl_info,
  panel UI, operator patterns, version stamping, and dev-folder workflow. Generates files
  directly into the Blender addons folder on Mac, with a ZIP at the end for archiving.
  Use this skill whenever the user wants to create a Blender addon, Blender tool, Blender
  plugin, or Blender script that needs a UI panel — including phrases like "make a Blender
  addon for X", "build a Blender tool", "scaffold a new addon", or "create a Blender panel".
---

# Blender Addon Scaffolder

Rockson is a theatre technical director and venue designer. His Blender addons support theatre production visualization (Cycles for quality renders, EEVEE for speed) and architectural design — things like rigging layout tools, equipment placement, geometry node parameter controllers, and production planning utilities. He controls Geometry Nodes from addon UI (sliders/inputs that drive modifier parameters), not by building node trees in Python.

He is not a professional developer. Keep code readable. Explain choices briefly using Blender concepts he already knows, not Python jargon.

---

## Step 1: Ask three questions

Before generating anything, ask:

1. **What does this addon do?** (One sentence)
2. **What operators does it need?** List them — e.g., "Generate Layout", "Clear All", "Export Count". For each, note if it needs sliders/inputs or just a button.
3. **Where should the panel live?**
   - **N-panel** (press N in viewport) — good for tools you use while working in 3D
   - **Properties panel** (right side, e.g., Object or Scene tab) — good for per-object or scene settings
   - **Header menu** — good for one-off actions
   - **Unsure** — you decide based on what the addon does

That's all. Do not ask about Python version, Blender internals, or file structure.

---

## Dev Workflow (always use this)

Write files directly to Blender's addons folder — no ZIP install loop during development.

**Mac path for Blender 5.0+:**
```
~/Library/Application Support/Blender/5.0/scripts/addons/<addon_name>/
```

After writing files, tell Rockson:
> "Files are in your Blender addons folder. In Blender: go to Edit → Preferences → Add-ons, find `<addon_name>`, enable it. To reload after changes, press F8 (requires Developer Extras: Edit → Preferences → Interface → Developer Extras)."

At the end, also generate a ZIP of the addon folder for archiving:
```bash
cd ~/Library/Application\ Support/Blender/5.0/scripts/addons/
zip -r <addon_name>_v<version>.zip <addon_name>/
```

Tell Rockson where the ZIP was saved.

---

## File Structure

Use a **package** (folder) structure for all addons — even simple ones. This makes reloading and future expansion easier.

```
<addon_name>/
├── __init__.py       ← entry point: bl_info, imports, register/unregister
├── operators.py      ← all operator classes
├── panels.py         ← all panel classes
└── properties.py     ← all custom properties (PropertyGroup)
```

Add `utils.py` only if there are shared helper functions used across files.

---

## `__init__.py` Template

```python
# ============================================================
# <ADDON NAME>
# Version: 0.1.0
# Blender: 5.0+
# Author: Rockson Chan
# Description: <one-line description>
# ============================================================
# CHANGELOG
# v0.1.0 - Initial scaffold
# ============================================================

bl_info = {
    "name": "<Addon Name>",
    "author": "Rockson Chan",
    "version": (0, 1, 0),
    "blender": (5, 0, 0),
    "location": "<Panel location — e.g., View3D > Sidebar > My Tab>",
    "description": "<Short description>",
    "category": "<Category — e.g., Object, Mesh, Scene>",
}

import bpy
import importlib
from . import operators, panels, properties

# Hot-reload support (F8 in Blender with Developer Extras)
if "bpy" in dir():
    importlib.reload(operators)
    importlib.reload(panels)
    importlib.reload(properties)


def register():
    properties.register()
    operators.register()
    panels.register()
    print(f"[{bl_info['name']}] v{'.'.join(str(v) for v in bl_info['version'])} loaded")


def unregister():
    panels.unregister()
    operators.unregister()
    properties.unregister()
    print(f"[{bl_info['name']}] unregistered")


if __name__ == "__main__":
    register()
```

The `print()` calls on register/unregister show the version in Blender's System Console — this is the primary version debug marker.

---

## `properties.py` Template

All addon settings live in a `PropertyGroup` attached to the Scene (or Object, if per-object makes more sense).

```python
import bpy


class <AddonName>Properties(bpy.types.PropertyGroup):
    # --- Example properties — replace with actual ones ---

    count: bpy.props.IntProperty(
        name="Count",
        description="Number of elements to generate",
        default=10,
        min=1,
        max=200,
    )

    spacing: bpy.props.FloatProperty(
        name="Spacing",
        description="Distance between elements in meters",
        default=1.0,
        min=0.0,
        soft_max=50.0,
        unit='LENGTH',
    )

    label: bpy.props.StringProperty(
        name="Label",
        description="Name or identifier",
        default="Element",
    )

    enabled: bpy.props.BoolProperty(
        name="Enable Feature",
        description="Toggle this feature on/off",
        default=True,
    )

    mode: bpy.props.EnumProperty(
        name="Mode",
        description="Operating mode",
        items=[
            ('CYCLES', "Cycles", "Use Cycles render settings"),
            ('EEVEE', "EEVEE", "Use EEVEE render settings"),
        ],
        default='EEVEE',
    )


def register():
    bpy.utils.register_class(<AddonName>Properties)
    bpy.types.Scene.<addon_name>_props = bpy.props.PointerProperty(type=<AddonName>Properties)


def unregister():
    del bpy.types.Scene.<addon_name>_props
    bpy.utils.unregister_class(<AddonName>Properties)
```

**Access properties in operators/panels with:**
```python
props = context.scene.<addon_name>_props
value = props.count
```

---

## `operators.py` Template

### Standard Operator (button click)

```python
import bpy


class <ADDON_NAME>_OT_<action_name>(bpy.types.Operator):
    """Tooltip shown on hover"""
    bl_idname = "<addon_name>.<action_name>"     # e.g., "theatre_layout.generate"
    bl_label = "<Button Label>"
    bl_options = {'REGISTER', 'UNDO'}             # UNDO allows Ctrl+Z

    def execute(self, context):
        props = context.scene.<addon_name>_props

        # --- Your logic here ---

        self.report({'INFO'}, "Done: <action description>")
        return {'FINISHED'}

    def invoke(self, context, event):
        # Called when operator is triggered from UI
        # Use this to show a confirmation dialog if needed
        return self.execute(context)
```

### Modal Operator (stays active while user interacts)

Use for tools where the user drags, clicks in the viewport, or adjusts in real time.

```python
class <ADDON_NAME>_OT_<modal_action>(bpy.types.Operator):
    """Tooltip — press Escape or right-click to cancel"""
    bl_idname = "<addon_name>.<modal_action>"
    bl_label = "<Modal Tool Label>"
    bl_options = {'REGISTER', 'UNDO'}

    def modal(self, context, event):
        if event.type == 'MOUSEMOVE':
            # React to mouse movement
            pass

        elif event.type == 'LEFTMOUSE' and event.value == 'PRESS':
            # Confirm action
            self.finish(context)
            return {'FINISHED'}

        elif event.type in {'RIGHTMOUSE', 'ESC'}:
            # Cancel — undo any changes
            self.cancel(context)
            return {'CANCELLED'}

        return {'RUNNING_MODAL'}

    def invoke(self, context, event):
        context.window_manager.modal_handler_add(self)
        return {'RUNNING_MODAL'}

    def finish(self, context):
        pass  # finalize your changes

    def cancel(self, context):
        pass  # undo any in-progress changes


# Register all operators in this file
classes = [
    <ADDON_NAME>_OT_<action_name>,
    <ADDON_NAME>_OT_<modal_action>,
]

def register():
    for cls in classes:
        bpy.utils.register_class(cls)

def unregister():
    for cls in reversed(classes):
        bpy.utils.unregister_class(cls)
```

**Naming convention:**
- `bl_idname`: `"<addon_name>.<action>"` — lowercase, underscores
- Class name: `<ADDON_NAME>_OT_<action>` — uppercase prefix, `OT` = Operator Type

---

## `panels.py` Template

```python
import bpy


class <ADDON_NAME>_PT_<panel_name>(bpy.types.Panel):
    bl_label = "<Panel Title>"
    bl_idname = "<ADDON_NAME>_PT_<panel_name>"
    bl_space_type = 'VIEW_3D'           # 3D viewport
    bl_region_type = 'UI'               # N-panel sidebar
    bl_category = "<Tab Name>"          # Tab label in N-panel


    def draw(self, context):
        layout = self.layout
        props = context.scene.<addon_name>_props

        # --- Version stamp (always show at top) ---
        from . import bl_info
        version_str = ".".join(str(v) for v in bl_info["version"])
        row = layout.row()
        row.label(text=f"v{version_str}", icon='INFO')

        layout.separator()

        # --- Properties ---
        layout.prop(props, "count")
        layout.prop(props, "spacing")
        layout.prop(props, "label")

        layout.separator()

        # --- Operators ---
        layout.operator("<addon_name>.<action_name>", text="Generate", icon='MESH_GRID')
        layout.operator("<addon_name>.<action_name_2>", text="Clear All", icon='TRASH')


# For Properties panel instead of N-panel, use:
# bl_space_type = 'PROPERTIES'
# bl_region_type = 'WINDOW'
# bl_context = 'scene'   (or 'object', 'render', etc.)


classes = [
    <ADDON_NAME>_PT_<panel_name>,
]

def register():
    for cls in classes:
        bpy.utils.register_class(cls)

def unregister():
    for cls in reversed(classes):
        bpy.utils.unregister_class(cls)
```

### Useful Layout Patterns

```python
# Section header (collapsible)
box = layout.box()
box.label(text="Section Title", icon='SETTINGS')
box.prop(props, "spacing")

# Two items side by side
row = layout.row(align=True)
row.prop(props, "count")
row.prop(props, "enabled", text="", icon='CHECKMARK')

# Slider (default for Float/Int props)
layout.prop(props, "spacing", slider=True)

# Full-width button
col = layout.column()
col.scale_y = 1.4  # taller button
col.operator("<addon_name>.<action>", text="Run", icon='PLAY')

# Disabled/grayed out button
row = layout.row()
row.enabled = props.enabled   # greyed out when False
row.operator("<addon_name>.<action>", text="Only when enabled")
```

---

## Geometry Node Parameter Control

To drive a Geometry Nodes modifier input from the addon UI:

```python
def set_geonode_input(obj, modifier_name, input_name, value):
    """Set a named input on a Geometry Nodes modifier."""
    mod = obj.modifiers.get(modifier_name)
    if not mod or mod.type != 'NODES':
        return
    # Blender 4.0+ uses node_group.interface for inputs
    node_group = mod.node_group
    for item in node_group.interface.items_tree:
        if item.item_type == 'SOCKET' and item.in_out == 'INPUT' and item.name == input_name:
            mod[item.identifier] = value
            break

# Example usage in an operator:
def execute(self, context):
    props = context.scene.<addon_name>_props
    obj = context.active_object
    set_geonode_input(obj, "GN_Modifier_Name", "Count", props.count)
    set_geonode_input(obj, "GN_Modifier_Name", "Spacing", props.spacing)
    obj.data.update_tag()
    context.view_layer.update()
    return {'FINISHED'}
```

---

## Versioning Convention

Use semantic versioning: `MAJOR.MINOR.PATCH`

| Change type | Example | Bump |
|---|---|---|
| New feature or panel | Added export operator | MINOR: 0.1.0 → 0.2.0 |
| Bug fix | Fixed crash on empty scene | PATCH: 0.1.0 → 0.1.1 |
| Breaking change / rewrite | Restructured all properties | MAJOR: 0.1.0 → 1.0.0 |

**Always update in three places:**
1. `bl_info["version"]` tuple in `__init__.py`
2. File header comment `# Version: X.Y.Z`
3. `# CHANGELOG` block — add a line for every version

Version is shown in the panel UI via the `draw()` method — Rockson can always see which version is running.

---

## Uninstall

Blender removes the addon cleanly via:
**Edit → Preferences → Add-ons → find addon → ▼ → Remove**

For this to work, `unregister()` must:
- Unregister all classes (in reverse order of registration)
- Delete any `bpy.types.*` properties added during `register()`

The templates above already handle this correctly. Do not skip `del bpy.types.Scene.<addon_name>_props` in `properties.unregister()` — failing to remove it causes errors when the user reinstalls.

---

## Notes for Claude

- Rockson is not a professional developer. Keep code readable, avoid clever tricks.
- He works on Mac with Blender 5.0+. Mac path is `~/Library/Application Support/Blender/5.0/scripts/addons/`.
- Always write files to the dev folder path, not just generate them in conversation.
- Always include the version stamp in the panel UI — this is a hard requirement.
- Always include the CHANGELOG block in `__init__.py`.
- Theatre context: he models rigging systems, stage layouts, lighting/audio equipment positions, venue geometry. Use these as examples when clarifying.
- If the addon will control Geometry Nodes, use the `set_geonode_input()` helper above.
- After writing files, give clear instructions for enabling and reloading in Blender.
