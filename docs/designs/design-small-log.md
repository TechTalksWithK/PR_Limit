# Design Document

## Metadata
- Design ID: DESIGN-20260407-001
- Title: Add one debug log for failed login parsing
- Author: Viper
- Date: 2026-04-07
- Status: approved

## Summary
Add a single debug log when login payload parsing fails, without changing authentication behavior.

## Goal
Add one narrow logging improvement.

## Non-Goals
- No auth logic changes
- No schema changes
- No dependency changes

## Relevant Files / Areas
### Likely Relevant Reads
- src/auth/login.ts
- src/utils/logger.ts

### Likely Edit Areas
- src/auth/login.ts

### Known Forbidden / Sensitive Areas
- src/db/
- src/config/