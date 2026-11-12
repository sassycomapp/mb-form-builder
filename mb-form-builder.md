---
title: "mb-form-builder"
doc-id: "mb-895c5b30"
updated: "2026-11-12"
---
# mb-form-builder

Writes the forms of one feature of a Mybizz Anvil application. It is the only way forms are produced, as `mo-ui-production` section 5 sets out. It builds; it does not wire. The Anvil agent fills the event handlers afterwards.

This file is the canonical specification. The installed runnable copy is `~/.config/opencode/skills/mb-form-builder/SKILL.md`; change both together.

## 1. Rules It Follows

Read these before writing anything. Where they and this skill differ, they are right and this skill is corrected.

- `project-library-global/specifications-global/spec-ui-standards.md`: every rule a form meets.
- `project-library-global/specifications-global/mo-ui-production.md` section 5: what this skill reads, writes and checks.
- `project-library-global/specifications-global/spec-component-properties.md`: how properties, bindings and events are written.
- `project-library-global/specifications-global/spec-nomenclature.md` section 1.6: component prefixes and handler names.
- `project-library-global/specifications-global/spec-translation.md`: phrase keys.
- Anvil's own `client-forms` skill: HTML template syntax.

## 2. Input

The developer names the application and the feature, for example "mb-3-cs App 1, feature `bookings`".

The skill reads:

1. every screen requirement of the feature: `{slug}-project-library/ui/<package path>/<FormName>/<FormName>-requirement.md`;
2. from the project library: the scaffold specification, the user flows, the authoritative schema and the SRS;
3. the theme's component documentation: `m3-custom-theme-project-library/docs/base-inventory.md`, `docs/components/<Component>.md`, and the application's copy of `docs/component-inventory.yaml` in `tests/`;
4. `approved_components` in the application's `tests/ts-config.yaml`;
5. the feature's server module in `server_code/`;
6. the application's `access-matrix.yaml` and `client_code/shared/translations/en.py`.

## 3. Gate

Stop and report, writing nothing, when any of these holds:

1. a form of the feature has no screen requirement, or its screen requirement's front matter is not `status: approved`;
2. the feature's server module does not exist;
3. a server function that a screen requirement names does not exist in that module;
4. the application's `tests/component-inventory.yaml` is missing, or differs from the theme library's `docs/component-inventory.yaml`;
5. a screen requirement asks for a component, property, event or slot that is not in the inventory, and is not an Anvil component in `approved_components`.

The report names each failed condition and the file it concerns. Never guess a missing fact.

## 4. Output per Form

For each form of the feature, at its module path:

1. `client_code/<package path>/<FormName>/form_template.html`:
   - the root is `<anvil-form layout="<app package>.layouts.<LayoutForm>">` for a page form, `<anvil-form layout="m3.Layouts.<ThemeLayout>">` for a layout form, or `<anvil-form container="ColumnPanel">` for a form with no layout; `<app package>` is `package_name` in `anvil.yaml`;
   - every theme component is `<anvil-component type="m3.<inventory path>" name="<prefix><purpose>"></anvil-component>`, for example `type="m3._Components.Button"`; never the legacy `form:dep_<id>:...` spec;
   - every property is `prop:<name>="<value>"`, its value written as JSON (`true`, `false`, `null`, numbers; anything else is a string) and one the inventory allows;
   - every visible text and tooltip is a phrase key;
   - every editing input has `writeback:<property>="self.item['<key>']"` on its writeback property;
   - no `on:` attributes: events are wired in Python;
   - no colour value, no sample data, no hyphen in any name, and every Anvil element closed with its own closing tag, never `/>`.
2. `client_code/<package path>/<FormName>/__init__.py`:
   - `from anvil import *` at the top, then the form class, with `self.init_components(**properties)` and then `translate_form(self)`;
   - every event the screen requirement uses, as `@handle("<component name>", "<event>")` above `def <component name>_<event>(self, **event_args):`, with the event exactly as the inventory names it, a body of `pass`, and a one-line docstring naming the server function the spec sheet gives it;
   - no other logic.
3. `theme/assets/docs/app{n}/<package path>/<FormName>.md`, the spec sheet, with the six contents `spec-ui-standards` section 14.2 lists. It holds no secret, key, password or user data, because `theme/assets/` is public.
4. The form's entry in `access-matrix.yaml` under `forms`, keyed by module path, with the scope of every role from the screen requirement.
5. The form's phrase keys in `en.py`, each with the English text the screen requirement gives.

## 5. Old Forms

In the same run, for each old form of the feature:

1. a new form at the same module path replaces the old form's files in place;
2. an old form with no new form at its module path is removed: its own folder, its spec sheet and its `access-matrix.yaml` entry;
3. every reference to a removed form is updated, including `startup` in `anvil.yaml`, routes and navigation code;
4. package folders and their `__init__.py` files are never removed.

## 6. Checks

1. Run, from the application folder:
   - `anvil --json validate client_code`
   - `python3 <project-library-global>/test-suite/tools/ts_scan.py . --config tests/ts-config.yaml`
2. Fix every failure in the feature's files and rerun both until both pass.
3. Stop and report when the same failure happens twice, or when a fix would touch a file outside the feature.

## 7. Report

End with:

1. the forms written, replaced and removed, by module path;
2. the other files changed;
3. the result line of each check;
4. anything the screen requirements left open.

## 8. Never

- Never fills an event handler body; that is the Anvil agent's task.
- Never changes server code, the data table schema or another feature's files.
- Never commits; only the developer commits.
- Never uses `Label`, `HtmlTemplate`, `HtmlPanel`, Anvil Extras, a YAML form template, or a component, property or value that is not in the theme's documentation.
