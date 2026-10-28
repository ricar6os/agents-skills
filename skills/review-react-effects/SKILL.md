---
name: review-react-effects
description: Review React Effects as synchronization escape hatches. Use when writing, reviewing, or refactoring React components or custom Hooks that contain useEffect, derived state, event-driven side effects, subscriptions, or Effect-based data fetching.
---

# Review React Effects

Treat every Effect as a synchronization boundary, not as a lifecycle callback. Base the audit on React's official [You Might Not Need an Effect](https://react.dev/learn/you-might-not-need-an-effect) guidance.

## 1. Establish coverage

Inspect the requested files or diff and the custom Hooks that own relevant behavior. Locate direct and aliased `useEffect` imports and calls. Identify the project's framework data APIs, query libraries, and external integrations before recommending replacements.

Finish with an inventory that accounts for every Effect definition in scope.

## 2. Name the synchronization

Complete this sentence for each Effect:

> When `[reactive values]` change, synchronize `[external system]`.

External systems include network resources, subscriptions, timers, browser APIs, non-React DOM, third-party widgets, storage, and analytics. An Effect is justified only when its behavior is caused by rendering and the external system must remain synchronized while the component is present.

If the sentence cannot name an external system, apply the removal ladder. If a specific user interaction causes the behavior, place it in that interaction's event handler.

Finish when every Effect has a stated cause and classification: removable or externally synchronizing.

## 3. Apply the removal ladder

Choose the first fitting replacement:

1. Calculate pure derived values during render.
2. Cache a genuinely expensive pure calculation with `useMemo` when measurement or a referential-stability contract justifies it.
3. Run interaction-specific work in the event handler that caused it.
4. Reset a component subtree by changing its `key`.
5. Store stable source state, such as an item ID, and derive the selected object during render.
6. Lift shared state to the nearest common owner or make the child controlled.
7. Fetch shared data in the parent and pass it down.
8. Subscribe to an external store with `useSyncExternalStore`.
9. Put once-per-application initialization in the application entry point or root module, with the environment guard required by the project's rendering model.
10. Use a framework loader, cache, or purpose-built custom Hook for reusable data synchronization.
11. As a last resort, adjust local state during render behind an identity guard when only part of the state must follow a prop.

Prefer the replacement that removes mirrored state and cascading renders while preserving observable behavior.

Finish when every removable Effect has a concrete replacement and the proposed change preserves loading, error, empty, and reset behavior that users can observe.

## 4. Audit the Effects that remain

Verify each remaining synchronization boundary:

- The Effect owns one independent synchronization process.
- Its dependency list contains every reactive value read by that process.
- Setup and cleanup are symmetrical across dependency changes and unmounts.
- Async work prevents stale responses from winning, through cancellation or an ignore guard.
- Setup tolerates React development remounts and setup-cleanup cycles.
- External writes are idempotent where repeated setup could duplicate work.
- Raw data-fetching Effects defer to the project's framework or cache when one exists.
- Repeated synchronization mechanics are hidden behind a purpose-built custom Hook.

Treat dependency linting as a minimum check, not proof that the Effect is necessary.

Finish when every retained Effect passes every applicable check, or has an actionable finding for each failed check.

## 5. Report or implement

For review requests, report findings first, ordered by user impact. Each finding must include:

- the file and line;
- what currently triggers the Effect;
- why the Effect is unnecessary or unsafe;
- the smallest behavior-preserving replacement.

Then list retained Effects with their external synchronization purpose and summarize the audited files. If there are no findings, say so explicitly and name any behavior that could not be verified.

For implementation requests, make the replacements, run the narrowest relevant tests and static checks, and summarize both removed and retained Effects. Preserve a valid Effect rather than forcing its removal.
