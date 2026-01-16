# OrgTrust Extension

## Overview

The OrgTrust extension provides governance and trust capabilities for agentic systems, including membership management, delegation, alliances, and trust verification.

## Extension Details

- **Name**: `orgtrust`
- **UID**: `8010`
- **Version**: `0.0.0`

## UID Allocation Strategy

### Extension Base
- Extension UID: `8010`
- Skills base UID: `801000`

### Category UIDs
- `governance_and_trust`: `801200` - Governance relationships and trust operations including membership, delegation, alliances, and trust verification

### Skill UID Blocks

#### Governance and Trust Category (801200-801299)

**Membership Operations:**
- `add_member`: `801201`
- `remove_member`: `801202`
- `verify_membership`: `801203`

**Delegation Operations:**
- `add_delegation`: `801204`
- `revoke_delegation`: `801205`
- `verify_delegation`: `801206`

**Alliance Operations:**
- `join_alliance`: `801207`
- `leave_alliance`: `801208`
- `verify_alliance_membership`: `801209`

**Trust Verification:**
- `trust_validate_name`: `801210`
- `trust_validate_account`: `801211`
- `trust_validate_app`: `801212`

**Reserved**: `801213-801299` for future governance and trust skills

## Skills

### Governance and Trust Skills

#### Trust Verification

**Verification Skills:**
- `trust_validate_name` - Validates a claimed human/org name (formatting, uniqueness, proof, policy checks, etc.)
- `trust_validate_account` - Validates an account identifier (e.g., wallet/account binding, control proofs, status, policy)
- `trust_validate_app` - Validates an app identity (publisher, signature, supply chain, provenance, policy conformance)

#### Membership Operations

**Operational Skills:**
- `add_member` - Adds a member to a group or organization with specified role and permissions
- `remove_member` - Removes a member from a group or organization, revoking associated permissions

**Verification Skills:**
- `verify_membership` - Verifies and attests membership status, role, and permissions

#### Delegation Operations

**Operational Skills:**
- `add_delegation` - Creates a delegation relationship granting specified permissions and scope
- `revoke_delegation` - Revokes an existing delegation relationship

**Verification Skills:**
- `verify_delegation` - Verifies and attests delegation status, scope, constraints, and validity

#### Alliance Operations

**Operational Skills:**
- `join_alliance` - Joins an agent or organization to an alliance with specified role and terms
- `leave_alliance` - Removes an agent or organization from an alliance

**Verification Skills:**
- `verify_alliance_membership` - Verifies and attests alliance membership status, role, terms, and validity

## Objects

### `membership_request`
Request object for membership operations.

**Attributes:**
- `subject` (required) - The agent or organization identifier
- `group` (required) - The group or organization identifier
- `role` (optional) - The role assigned to the member
- `effectiveTime` (optional) - Timestamp when the membership change becomes effective
- `reason` (optional) - Reason or justification for the membership change

### `delegation_request`
Request object for delegation operations.

**Attributes:**
- `delegator` (required) - The agent or organization granting the delegation
- `delegate` (required) - The agent or organization receiving the delegation
- `scope` (required) - The scope of permissions or actions being delegated
- `constraints` (optional) - Constraints or limitations on the delegation
- `expiry` (optional) - Timestamp when the delegation expires

### `alliance_request`
Request object for alliance operations.

**Attributes:**
- `org` (required) - The agent or organization joining or leaving the alliance
- `alliance` (required) - The alliance identifier
- `role` (optional) - The role assigned within the alliance
- `terms` (optional) - Terms, conditions, or agreements associated with the alliance

## Mapping to Core OASF Skills

### Governance Skills
- `add_member` / `remove_member` → Similar to core `role.assign` / `role.revoke` (specialization with stronger semantics)
- `verify_membership` → Similar to core `access.verify` (specialization for membership context)

### Delegation Skills
- `add_delegation` / `revoke_delegation` → Similar to core `access.grant` / `access.revoke` (specialization with delegation semantics)
- `verify_delegation` → Similar to core `access.verify` (specialization for delegation context)

### Alliance Skills
- `join_alliance` / `leave_alliance` → Similar to core `relationship.create` / `relationship.delete` (specialization for alliance context)
- `verify_alliance_membership` → Similar to core `relationship.verify` (specialization for alliance context)

## Design Decisions

### Role-Centric Membership Model
This extension uses a **role-centric membership model** for interoperability:
- Members are assigned roles (member, admin, operator, etc.)
- Roles define permissions and capabilities
- This aligns with OASF's capability-oriented approach

### Separation of Operational and Verification Skills
Skills are separated into:
1. **Operational skills** - Perform the action (add, remove, join, leave)
2. **Verification skills** - Prove/validate/attest the state (verify_*)

This separation allows agents to:
- Query capabilities separately from execution capabilities
- Support trust models that require independent verification
- Enable delegation of verification without granting operational permissions

## File Structure

```
schema/extensions/orgtrust/
├── extension.json
├── main_skills.json
├── objects/
│   ├── membership_request.json
│   ├── delegation_request.json
│   └── alliance_request.json
└── skills/
    └── governance_and_trust/
        ├── trust/
        │   ├── trust_validate_name.json
        │   ├── trust_validate_account.json
        │   └── trust_validate_app.json
        ├── membership/
        │   ├── add_member.json
        │   ├── remove_member.json
        │   └── verify_membership.json
        ├── delegation/
        │   ├── add_delegation.json
        │   ├── revoke_delegation.json
        │   └── verify_delegation.json
        └── alliance/
            ├── join_alliance.json
            ├── leave_alliance.json
            └── verify_alliance_membership.json
```

## Usage

### Syncing Skills

After committing and pushing to your OASF fork:

```bash
cd /path/to/agent-explorer/apps/indexer
GITHUB_TOKEN="your_token" \
OASF_REPO="your_username/oasf" \
OASF_REF="main" \
pnpm -s skills:sync
```

See [SKILLS_SYNC_INSTRUCTIONS.md](../../SKILLS_SYNC_INSTRUCTIONS.md) for detailed instructions.

## Versioning

This extension follows semantic versioning:
- **Major**: Breaking changes to skill/object structure
- **Minor**: New skills/objects added (backward compatible)
- **Patch**: Bug fixes, documentation updates

## Immutability

Once published:
- UIDs are immutable (never renumber)
- Skill names are immutable (never rename)
- Object names are immutable (never rename)

For breaking changes, create a new extension or increment the major version.
