# Swift/iOS Best Practices Review Reference

## Platform Conventions
- Apple APIs are used as intended.
- Availability checks exist for version-gated APIs.
- Deprecated APIs are avoided unless justified.
- App/platform configuration matches intended runtime behavior.

## Architecture & Boundaries
- UI, domain, and data concerns remain understandable.
- Side effects are not scattered arbitrarily.
- Singletons are minimized and justified.
- Navigation/presentation logic has a clear owner.

## Release Hygiene
- Debug code is gated away from release.
- Build configuration differences are explicit and safe.
- Logging in release is restrained and does not leak internal details.

## UX / Resilience
- User-visible failure modes have a reasonable fallback or message path when relevant.
- Permission prompts are contextual and not premature.
- Accessibility identifiers/labels are preserved where affected by the diff.
- Empty/loading/error states are not accidentally broken by the change.

## Testability-Friendly Design
- Side effects are isolatable where it matters.
- Network/storage/time dependencies are not unnecessarily hard-wired.
- Pure logic is not entangled with platform plumbing without reason.

## Review Heuristic
Only flag best-practice issues when they have a practical downside:
- correctness risk
- maintainability cost
- release safety problem
- platform misuse
- observably worse user experience