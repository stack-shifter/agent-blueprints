## Accessibility

Every interface meets WCAG 2.2 AA. Accessibility is part of building the feature, not a pass that happens afterwards.

- Use the semantic element for the job. Something that performs an action is a button; something that navigates is a link. Never attach a click handler to a `div`, `span`, or icon element.
- Every interactive element is reachable and operable by keyboard in a logical order, with a visible focus indicator. Never remove a focus outline without replacing it with something at least as clear.
- Every control has an accessible name. An icon-only button needs an explicit label. Every image carries alt text, or empty alt when it is purely decorative.
- Associate each input with its label, mark invalid fields programmatically, and link the error message to the field it describes. An error conveyed only by styled text is invisible to a screen reader.
- A dialog identifies itself as a dialog, is labelled by its title, moves focus inward when it opens, keeps focus inside while it is open, closes on Escape, and returns focus to whatever opened it.
- Announce asynchronous outcomes — loading, success, failure — in a live region. A message that only appears visually is never heard.
- Meet AA contrast for text and for interactive controls. Never let colour alone carry meaning; pair it with text or a shape.
- Honour the reduced-motion preference for animation and transitions.
- Use real table markup for tabular data. `scope` belongs on a header cell, never on a data cell.
