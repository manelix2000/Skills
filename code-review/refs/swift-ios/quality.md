# Swift/iOS Quality Review Reference

## Review Goal
Catch logic defects, maintainability issues, anti-patterns, state-management mistakes, and unsafe Swift/iOS coding patterns.

## Correctness
- Optional handling is safe; avoid obvious crash paths from force unwraps or invalid assumptions.
- Branching logic covers failure and empty states.
- State transitions are explicit and coherent.
- Async workflows preserve correctness across suspension points and callbacks.
- UI state is updated consistently after success, failure, cancellation, and retry paths.

## Error Handling
- Errors are handled intentionally, propagated meaningfully, or surfaced to the caller.
- Empty catch blocks do not swallow failures silently.
- Retry logic is bounded and observable.
- Cancellation is handled correctly in async code.
- Cleanup logic executes on failure paths where relevant.

## Maintainability
- Types have clear responsibilities.
- View controllers / view models / coordinators are not overloaded.
- No God objects, copy-paste logic, magic numbers without context, or sprawling switch/if trees without decomposition.
- Naming supports comprehension of behavior and ownership.
- Complex logic is decomposed when readability or testability suffers.

## SOLID / Design Signals
- SRP: each type has a coherent reason to change.
- OCP: extension points are used where appropriate, not giant conditional dispatch.
- DIP: side-effecting dependencies are injected where useful.
- ISP: protocols are not bloated beyond real needs.

## Swift-Specific Checks
- Force unwraps and force casts are justified or avoided.
- Avoid unnecessary Any / type erasure without reason.
- Access control is intentional.
- Value vs reference semantics are chosen deliberately.
- Extensions remain cohesive rather than becoming dumping grounds.
- Property observers and lazy properties do not hide surprising side effects.

## UIKit-Specific Checks
- Correct lifecycle hook usage.
- No heavy side effects in viewDidLoad/viewWillAppear without justification.
- UI updates occur on the main thread.
- Reusable views/cells are configured idempotently.
- Navigation, dismissal, and presentation logic are coherent.

## SwiftUI-Specific Checks
- State ownership is correct for @State / @StateObject / @ObservedObject / @EnvironmentObject.
- Business logic is not buried inside views unless intentionally tiny.
- View identity and state reset risks are understood.
- Side effects in task/onAppear/onChange are bounded and appropriate.

## Concurrency & State
- MainActor boundaries are respected for UI-affecting code.
- Shared mutable state is protected appropriately.
- Async task creation is not unbounded.
- No obvious race between callback/task completion and view lifetime.

## Positive Signals to Reward
- clear decomposition
- intentional dependency injection
- explicit state modeling
- graceful error handling
- cancellation-aware async code

## Severity Guidance
CRITICAL:
- obvious crash/corruption path in normal usage
HIGH:
- likely production bug, broken state transition, type-safety/build issue
MEDIUM:
- maintainability risk, partial error handling, design smell with real downside
LOW:
- minor non-blocking cleanup