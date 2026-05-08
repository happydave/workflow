# Versioning Guidelines

## Purpose
These guidelines define the versioning policy for projects managed under this framework. Consistent versioning ensures that progress is trackable and that releases (even minor internal ones) are clearly identified.

## Core Policy: One Ticket, One Patch
For most projects (see Exemptions below), every completed work item SHALL result in exactly one patch version increment.

- **Trigger**: Successful completion of all implementation and verification steps for a work item.
- **Increment**: Increment the patch version (e.g., `1.2.3` -> `1.2.4`).
- **Timing**: The version bump should be the final change made during the **Implement** phase, just before closing the work item.

## Project-Specific Implementation
The exact mechanism for versioning depends on the project's language and tooling:

- **TypeScript / Node.js**: Increment the `version` field in `package.json`.

## Exemptions

### Go Projects
Go projects are **exempt** from this specific file-based versioning policy.
- **Reason**: Go versioning is idiomatic via git tags (`v1.2.3`).
- **Policy**: Unless the plan specifies otherwise, skip tagging for Go Projects.
