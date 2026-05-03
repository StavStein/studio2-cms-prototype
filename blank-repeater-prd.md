# Blank Repeater Product Requirements

## Overview

This document defines the core product requirements for connecting a blank Repeater to data.
It is based on the prototype flow `Repeater page -> Blank Repeater (Updated)`.

The primary user intent is simple: a builder adds a plain Repeater, designs the item layout, and later connects it to a data source so each repeated item can render different data.

## Status Key

- `Covered in prototype`: The behavior exists in the current prototype.
- `Partially covered in prototype`: The prototype shows the direction, but product or UX details still need to be completed.
- `Need to be covered`: The behavior is important but is not yet represented clearly in the prototype.
- `Specified separately`: Part of the product area, but detailed requirements live in a dedicated section or follow-up PRD.



## Core Concepts

| Concept | Requirement |
| --- | --- |
| Repeater | The visual component that repeats item UI. |
| `Items` property | The Repeater data property. It must bind to an array field from a context. |
| Context | A data source instance available to the Repeater, section, page, or parent Repeater. |
| Attached context | A context directly connected to the Repeater. Its fields are available for binding. |
| Item fields | Fields available inside each repeated item after `Items` is connected. |
| Inner bindings | Bindings on elements inside the Repeater item, such as title text or image source. |

## Primary Flow: Blank Repeater Updated

Status: `Covered in prototype`

1. User adds a blank Repeater to the page.
2. Repeater appears with 3 empty items.
3. Canvas communicates that the Repeater is not connected to data.
4. User clicks `Connect data`.
5. A floating binding panel opens.
6. User selects a context.
7. System auto-selects the first available array field from that context.
8. User can keep the auto-selected array or choose another array field.
9. User clicks `Apply`.
10. Repeater `Items` is bound to the selected context and array field.
11. Inspector opens on the Repeater with `Properties` expanded, Context collapsed.
13. `Items` shows the selected context and array field.
14. User can inspect available item fields from the item-level settings panel.

## Requirements By Flow

### 1. Blank Repeater Initial State

Status: `Covered in prototype`

Requirements:

- A newly added blank Repeater has no context and no `Items` binding.
- All repeated items show 3 identical items (blank).
- Repeater header shows a disconnected state, for example `not connected`.
- Action bar shows a clear `Connect data` entry point.
- Canvas shows a clear `Connect data` entry point.
- Selecting the Repeater opens Repeater settings.
- Selecting a Repeater item opens item-level settings.

Acceptance criteria:

- The user can understand that the Repeater is designable but not data-driven yet.
- The user can start the connection flow from the canvas or Repeater action surface.

### 2. Context Attachment For Blank Repeater

Status: `Partially covered in prototype`

Requirements:

- The user can attach a context directly to the Repeater.
- A context attached to the Repeater appears in the Repeater `Context` section.
- A context attached to the Repeater is available as a source for `Items`.
- The attached context should be visually scoped as `This repeater`.
- If the attached context contains an array field, the system should auto-select the first available array for `Items`.
- If the attached context has no array fields, the user must see a clear empty state and recovery path.

Current prototype coverage:

- Repeater can own contexts through the Repeater `Context` section.
- Repeater context cards can be displayed in the inspector.
- Connecting through the updated blank Repeater flow also attaches the selected context to the Repeater.

Needs coverage:

- The attach-context panel should clearly show when a listed context is already connected to this Repeater.
- The binding source list for `Items` should show a `This repeater` indication for contexts attached to the selected Repeater.

### 3. Attach Context Panel

Status: `Partially covered in prototype`

Requirements:

- The panel lists available contexts grouped or labeled by type.
- Contexts already attached to the selected Repeater show `This repeater`.
- Contexts without usable array fields should be disabled and placed at the bottem of the list.

Current prototype coverage:

- Context lists show source and type information.
- Contexts are grouped in the broader Add Context panel.
- Some binding panels support scope badges such as `This repeater`.

Needs coverage:

- The updated blank Repeater `Items` picker should consistently show scope badges, especially `This repeater`.
- Context rows should indicate whether they expose an array usable for Repeater `Items`, or the context is disabled for selection.

### 4. Binding Repeater Items

Status: `Covered in prototype`

Requirements:

- `Items` can bind only to array-like fields.
- The user first selects a source context.
- After source selection, the system auto-selects the first available array field.
- The user can change the selected array field before applying.
- The `Apply` action is enabled only when both source and array field are selected.
- Applying the binding updates:
  - Repeater `Items` binding.
  - Repeater context ownership.
  - Repeater item count.
  - Repeater canvas hat.
  - Inspector `Items` row.

Acceptance criteria:

- The user can connect a blank Repeater without manually wiring inner item fields.
- The connected Repeater still preserves existing item layout and static placeholder content until item fields are bound.

### 5. Auto Binding And Auto Selection To First Array

Status: `Covered in prototype`

Requirements:

- When a source context is selected in the `Items` binding panel, the first array field is selected automatically.
- Picker auto-selection happens before the user clicks `Apply`.
- When a list context is attached through the context panel and `Items` is still unbound, the Repeater should auto-bind `Items` to the first available array field.
- Auto-binding should not overwrite an existing `Items` binding.
- The user can override the selected array field before applying when using the picker flow.
- Nested arrays and multi-reference fields can appear as selectable array options.
- Array labels should show enough hierarchy to distinguish nested fields.

Acceptance criteria:

- A context with a single array can be applied with minimal clicks.
- A context with multiple arrays does not hide the fact that another array can be selected.
- Attaching a usable list context to an unbound Repeater produces a connected Repeater without extra manual setup.

### 6. Item-Level Settings Panel
Open question-Do we have to show the Settings tab? Builder question!
This requiremnt is based on an asuumption it is need to be shown:
Status: `Covered in prototype`

Requirements:

- Selecting a Repeater item opens item-level settings.
- If `Items` is not connected, the item-level panel explains that the Repeater is not showing data yet and offers `Connect to data`.
- If `Items` is connected, the item-level panel shows fields available inside the selected array.(read only)
- Existing layout and styling remain editable through design controls.
- Event handler requirements for Repeater items are not central to this PRD unless they are affected by `Items` connection state.

Current prototype coverage:

- Connected item settings show available item fields.
- The unconnected item state offers a connect-to-data path.


### 7. Replace Context

Status: `Covered in prototype`

Requirements:

- A connected Repeater can switch `Items` to another context and array field.
- If inner bindings reference the old context, the system must warn the user before switching.
- The warning must explain that inner bindings will be disconnected or need remapping.
- The warning must include the number of affected inner bindings.
- The Repeater layout and styling are preserved.
- After switching, the Repeater uses the new context and selected array.
- Inner elements that cannot be safely preserved become unbound.

Acceptance criteria:

- The user can repurpose a Repeater layout for another collection without rebuilding the layout.
- The user is warned before losing inner bindings.

Needs coverage:


- The final product should define whether users can keep old bindings when switching context, and how that state is communicated.
- Do we have a way to revert this changes?

### 8. Disconnect Repeater: Items

Status: `Covered in prototype`

*Open question: do we need revert? maybe we should show a toast with revert action-what is the Editor pattern?*

Requirements:

- User can disconnect the `Items` binding from the Repeater.
- Disconnecting `Items` is destructive for item-level inner bindings.
- The system must show a confirmation dialog before disconnecting.
- Confirmation copy must include:
  - The context being disconnected.
  - The number of inner bindings that will be removed.
  - A clear statement that item fields cannot stay connected without `Items`.
- On confirm:
  - `Items` binding is removed.
  - Inner bindings that depend on the old item context are removed.
  - Repeater returns to placeholder content.
  - Repeater item count returns to the disconnected state (3).
  - Repeater layout and styling are preserved.

Acceptance criteria:

- The user understands the cascading effect before confirming.
- Cancel leaves all bindings unchanged.

### 9. Disconnect Repeater: Context

Status: `Covered in prototype`

Requirements:

- User can remove a context from the Repeater context card.
- Removing the context is distinct from disconnecting only `Items`.
- Inner bindings that reference the removed context are removed.
- The system must warn before removal when bindings will be affected.
- After removal, the Repeater returns to the disconnected placeholder state.

Acceptance criteria:

- The user can distinguish between removing a data source from the Repeater and only disconnecting the Repeater's `Items`.
- The user sees what will break before confirming.

Needs coverage:

- Final confirmation copy should be differentiated from the `Items` disconnect copy.
- Recovery behavior should be specified: reconnect same context, connect a different context, or undo.

### 10. Context Mismatch: Attached Context vs Items Source

Status: `Need to be covered`

Requirement:

- The product must handle the case where the Repeater has one context attached, but `Items` is bound to a different context.

Expected behavior:

- The `Context` section shows attached contexts.
- The `Items` row clearly shows the actual source used to render Repeater items.
- If the attached context is not the `Items` source, the inspector should make that clear.
- Canvas indications should reflect the actual `Items` data source, not only the attached context.

Why this matters:

- Users may assume the attached context is what feeds the Repeater.
- A mismatch can be valid, but must not be invisible.

### 11. No Array Fields Available

Status: `Partially covered in prototype`

Requirements:

- If a selected context has no array fields, the user cannot apply it to `Items`.
- The panel shows a clear message that this context has no array fields.
- The user can choose another context or create/add a context with a list/array.
- suggestion: maybe contexts that has no Array-will apper disables?

Needs coverage:

- The message / disabled indication in the connect source panel.

### 12. Empty Data After Connection

Status: To be coverd by May


## AI Interactions

Status: `Partially covered in prototype`

AI interactions are part of the broader Repeater data-connection experience, but they need a dedicated requirements section because they introduce different entry points, decision logic, and success states.

The following AI flows should be specified separately:

- Action bar AI context menu behavior when a Repeater is selected.
- Action bar AI context menu behavior when an element inside a Repeater is selected.
- Binding an element to data using AI.
- AI-assisted field insertion or auto-layout suggestions.

This PRD covers the non-AI baseline behavior that those AI flows should build on.

## Coverage Matrix

| Flow / Requirement | Prototype Status |
| --- | --- |
| Blank Repeater initial state | Covered in prototype |
| Canvas `Connect data` entry point | Covered in prototype |
| Floating `Items` binding panel | Covered in prototype |
| Select source context | Covered in prototype |
| Auto-select first array field | Covered in prototype |
| Auto-bind first array when attaching list context | Covered in prototype |
| Apply `Items` binding | Covered in prototype |
| Repeater context ownership | Covered in prototype |
| Inspector opens after binding | Covered in prototype |
| Item-level settings panel | Covered in prototype |
| Read-only item field list for updated flow | Covered in prototype |
| Disconnect `Items` confirmation | Covered in prototype |
| Remove context from Repeater | Covered in prototype |
| Switch context warning | Covered in prototype |
| Attach context panel type indication | Partially covered in prototype |
| `This repeater` indication in updated `Items` picker | Need to be covered |
| Context already attached to this Repeater indication | Need to be covered |
| Context with no array fields in blank flow | Partially covered in prototype |
| Attached context differs from `Items` source | Need to be covered |
| Empty connected data state | Need to be covered |
| Smart field remapping on switch | Need to be covered |
| AI action bar and AI binding flows | Covered in prototype |

