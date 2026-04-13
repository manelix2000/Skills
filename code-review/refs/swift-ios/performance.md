# Swift/iOS Performance Review Reference

## Review Goal
Catch diff-relevant performance regressions affecting responsiveness, memory, rendering, concurrency, and data access.

## UI Responsiveness
- No blocking file I/O, decoding, image processing, crypto, compression, or heavy computation on the main thread.
- Expensive work is scheduled off the main thread when appropriate.
- UI refreshes are not triggered more often than needed.
- Repeated formatting/parsing/layout work is avoided in hot UI paths.

## Memory
- Watch for retain cycles in closures, delegates, timers, notifications, Combine chains, and async tasks.
- Large images/data blobs are not retained longer than necessary.
- Caches are bounded or clearly scoped.
- View/controller lifetime is not accidentally extended by background work.

## Rendering / Lists / Reuse
- Collection/table cell configuration is reuse-safe and efficient.
- Expensive layout or image transformation work is not repeated during scrolling without need.
- Re-render triggers are minimized in SwiftUI where identifiable.
- Avoid repeated work in computed properties used during rendering.

## Concurrency
- Task creation is bounded and cancellable.
- Duplicate concurrent requests are coordinated where needed.
- Actor/queue hopping is not excessive.
- No obvious contention or accidental serialization bottleneck on critical paths.

## Data Access
- Avoid N+1-style fetch patterns where repository context shows repeated fetching.
- Batch or cache repeated expensive reads where appropriate.
- Disk/network access on UI path is justified.

## Common Swift/iOS Performance Smells
- repeated DateFormatter/NumberFormatter creation in render paths
- image decoding/resizing on main thread
- unbounded task spawning
- retain cycles from self capture
- repeated diff-insensitive recomputation
- blocking synchronous APIs on UI path

## Severity Guidance
HIGH:
- main-thread blocking on likely user path
- significant retain-cycle risk on common path
- obvious repeated expensive work on scrolling/render path
MEDIUM:
- avoidable inefficiency with user-visible downside under load
LOW:
- small optimization opportunity