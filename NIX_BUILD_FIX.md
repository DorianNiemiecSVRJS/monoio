# Monoio Nix Build Fix

## Purpose

This fork fixes a Nix build issue in monoio to enable building Ferron 2.x with Nix.

## The Problem

When building monoio with Nix, the build fails because:

```rust
#![doc = include_str!("../../README.md")]
```

This tries to include `README.md` during documentation generation, but Nix's vendored dependencies don't include the README file, causing:

```
error: couldn't read `../../README.md`: No such file or directory
```

## The Fix

**File**: `monoio/src/lib.rs`

**Change**:
```diff
-#![doc = include_str!("../../README.md")]
+//! Monoio async runtime
```

**Commit**: b959d4094b224234d78c20f67db6edcaa3c0ad61

## Impact

- ✅ Nix builds now succeed
- ✅ No functional changes to monoio
- ✅ Only affects documentation
- ✅ Runtime behavior unchanged

## Usage

In `Cargo.toml`:
```toml
[patch.crates-io]
monoio = { git = "https://github.com/meta-introspector/monoio.git", branch = "fix-nix-build" }
```

## Upstream

This fix is specific to Nix builds. For upstream consideration:

**Option 1**: Use `#[cfg_attr]` to conditionally include README
```rust
#![cfg_attr(not(feature = "nix-build"), doc = include_str!("../../README.md"))]
```

**Option 2**: Copy README during Nix build
```nix
postPatch = ''
  cp README.md monoio/
'';
```

**Option 3**: Use inline documentation instead of include_str

## Issue Discovered

While using this fork with Ferron 2.x, we discovered that monoio fails to bind to ANY port with "Permission denied (os error 13)", even high ports and even as root.

**Environment**:
- Kernel: 6.8.0-94-generic  
- io_uring: enabled
- SELinux: disabled

**Question**: Does monoio require specific Linux capabilities beyond standard socket binding?

## Related

- **Ferron Integration**: https://github.com/meta-introspector/ferron/blob/develop-2.x/ZOS_INTEGRATION.md
- **Upstream Monoio**: https://github.com/bytedance/monoio
- **Original Fork**: https://github.com/DorianNiemiecSVRJS/monoio

---

**Status**: Working for Nix builds ✅  
**Date**: 2026-03-07  
**Branch**: fix-nix-build
