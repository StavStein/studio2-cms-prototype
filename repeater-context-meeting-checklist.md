# Repeater And Context Meeting Checklist

## 1. Context Attachment To Repeater

- [ ] Show how a context is attached directly to a repeater.
- [ ] Confirm context is collapsed for Repater use-case.
- [ ] Clarify context on repeater vs. context inherited from parent.
- [ ] Review context hat and action bar actions.

## 2. Binding Repeater Items

- [ ] Cover manual binding of `Items`.
- [ ] Cover binding from the Add Panel.
- [ ] Cover binding using AI.
- [ ] Cover binding using inspector suggestions.
- [ ] Confirm source selection behavior.
- [ ] Confirm field or array selection behavior.
- [ ] Confirm inspector state after binding.

## 3. Item-Level Settings Panel

- [ ] Open item-level settings from a repeater item.
- [ ] Review which settings apply per item.
- [ ] Clarify what is inherited from the repeater (only item array's fileds)
- [ ] Confirm the behavior of the inspector settings in the item selection.


## 4. Disconnect Repeater

- [ ] Disconnect the `Items` binding.
- [ ] Confirm warning and confirmation copy.
- [ ] Confirm what happens to inner element bindings. 
- [ ] Confirm visual fallback state on canvas.
- [ ] Disconnect context from repeater.
- [ ] Compare disconnecting `Items` vs. removing the context.
- [ ] Confirm recovery and reconnect path.

## 5. Switch Context Experience

- [ ] Switch context from the context card.
- [ ] Switch context from the `Items` binding controller.
- [ ] Confirm warning when inner bindings will be disconnected.
- [ ] Review button labels and action hierarchy.
- [ ] Confirm whether layout and styles are preserved.
- [ ] Confirm remapping expectations after switching.

## 6. Binding, Auto-Binding, And Auto-Fill

- [ ] Show regular binding flow.
- [ ] Show auto-binding existing components to matching fields.
- [ ] Show auto-fill flow for blank repeater.
- [ ] Confirm the dreamy experience: user clicks `Connect data`, picks a collection, and the repeater fills with real data.
- [ ] Confirm item count updates automatically.
- [ ] Confirm components are created or populated correctly.
- [ ] Confirm no manual field wiring is needed.
- [ ] Identify where the experience still feels too magical or unclear.

## Open Questions

- [ ] Should context attachment and `Items` binding happen in one flow or remain separate concepts?
- [ ] When should the system auto-bind vs. ask for confirmation?

