# selectionrange

## Credits

This package is very, very heavily informed by
[`shadow-selection-polyfill`](https://github.com/GoogleChromeLabs/shadow-selection-polyfill).
 Where that package seeks principally to determine selected text, this one
attempts to more precisely identify selected DOM elements so that it can be
safely used with `contenteditable` elements. 

## The problem

Browsers are inconsistent when a `Selection` includes content that is inside a
`ShadowRoot`

* Chrome implements `getSelection` on `ShadowRoot` which returns the real selection range inside the ShadowRoot.

* Firefox doesn't implement `getSelection` on `ShadowRoot`, but `Document.getSelection` sees into a `ShadowRoot` as if it wasn't hiding anything.

* Safari's `Document.getSelection` gives a boundary point which is just before the `ShadowRoot`.

This package aspires to paper over these differences with three functions.

## The API

### `getSelectionRange`

`getSelectionRange(root)` gets the current selection if it is under the
`root`.  For `root`, it takes either a `ShadowRoot` or a `Document`.  If there
is no selection in the given `root`, it returns `null`.  Otherwise, it returns
an object with this shape:

```
{
    anchorNode: Node,
    anchorOffset: Int,
    focusNode: Node,
    focusOffset: Int
} 
```

The fields of this object follow the fields of the same name in a [`Selection`](https://developer.mozilla.org/en-US/docs/Web/API/Selection).

Note that this is not a formal [`Range`](https://developer.mozilla.org/en-US/docs/Web/API/Range) and will not update dynamically.

On Chrome and Firefox this should be about as fast as regular `getSelection`.  On Safari it could be considerably slower -- to determine position in text nodes, it often has to compute the entire string represented by the selection region, and also sometimes has to split text nodes one letter at a time (idea again from `shadow-selection-polyfill`).  The cost is at least `O(N)` and could be `O(N^2)` in text node length -- I don't actually know how (in)efficient text node splitting is under the hood in Safari.

### `setSelectionRange`

`setSelectionRange(root, range)` sets the current selection under the `root`.  It takes either a `Document` or a `ShadowRoot` as `root`.  For `range` it takes either `null` to indicate no selection range, or an object of the same shape as shown above.

### `isSquelchingEvents()`

On Safari, `getSelectionRange` generates a shower of `selectionchange` events and triggers mutation observers, even though by the time their handlers are run the selection and DOM are restored to original state.  `isSquelchingEvents` returns true during a period that is intended to cover all of these byproduct events.  Clients that don't want to respond to all of these can check `isSquelchingEvents` in their event handlers and exit early when it is true.  There is a theoretical possibility that some non-byproduct events could get hidden behind this squelch, please let me know if you run into this.

## Automated testing

There is no automated testing of the package yet, so there are likely many quirky cases as yet undiscovered.

## Request for feedback

This package is still in alpha.  I'd love any and all reports of field experiences with it.  Finding out that there are actual users will be the best motivation for me to improve it :)