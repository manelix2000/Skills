# Swift/iOS Security Review Reference

## Scope
Review security risks relevant to Swift and iOS application code, app configuration, local storage, networking, permissions, and client-side trust boundaries.

## Secrets
- No API keys, client secrets, private tokens, passwords, signing materials, or internal endpoints hardcoded in source, plist, xcconfig, or fixtures.
- No secrets committed in repository history if history inspection is available.
- Environment/config separation is explicit for dev/staging/prod.

## Sensitive Data Handling
- Tokens, credentials, and other secrets are stored in Keychain when persisted.
- Sensitive data is not stored in UserDefaults unless explicitly non-sensitive.
- PII, tokens, session identifiers, or credentials are not logged.
- Sensitive files use data protection/file protection when applicable.
- Clipboard usage for secrets is avoided or tightly justified.

## Network & Transport Security
- HTTPS is used unless an explicit exception is justified.
- ATS exceptions are minimal and justified.
- URLSession trust evaluation does not disable certificate validation.
- No insecure custom certificate bypass or allow-all trust logic.
- Web content loading is constrained and origin-aware.

## Authentication & Authorization
- Client-side checks do not replace server-side authorization.
- Tokens/session material are refreshed, invalidated, and cleared safely.
- Logout clears local auth state and cached sensitive material where relevant.
- Biometric gating protects local UX access but is not treated as sufficient authorization by itself.

## Input Validation & External Entry Points
- Deep links / universal links validate host, path, parameters, and routing intent.
- File import/share/open-in-place flows validate type and origin.
- Untrusted input is validated before being used in file paths, URLs, web content, or commands.
- WKWebView JS bridges are tightly scoped and do not expose privileged native actions unsafely.

## Privacy & Permissions
- Info.plist usage descriptions match actual data access.
- Camera, microphone, location, photos, contacts, notifications, and tracking permissions are least-privilege and contextually justified.
- Analytics and crash reporting do not leak sensitive fields.

## Local Attack Surface
- Debug-only code is not reachable in release builds.
- Feature flags do not unlock privileged behavior solely on the client.
- Jailbreak detection, if present, is treated as a heuristic and not a trust boundary.

## Supply Chain
- New SPM/CocoaPods/Carthage dependencies are reviewed for legitimacy and maintenance risk.
- Security-sensitive dependencies are not added casually.
- Dependency changes affecting auth, crypto, or networking get extra scrutiny.

## Swift/iOS Security Patterns to Check
- hardcoded secrets in source or config
- insecure UserDefaults storage for secrets
- unsafe URL construction from untrusted input
- permissive deep link routing
- insecure certificate handling
- sensitive logs
- unsafe WKWebView bridges
- release builds containing debug behavior

## Severity Guidance
CRITICAL:
- hardcoded production secrets
- auth bypass
- certificate validation bypass in production path
- sensitive data exposure with broad impact

HIGH:
- insecure token storage
- permissive deep link / web bridge leading to meaningful abuse
- release debug behavior exposing privileged operations

MEDIUM:
- overly broad logging, weak privacy hygiene, risky config defaults