# Security Advisory

## CVE-XXXX-XXXX: Anthropic Claude Code Windows Command Injection RCE

| Item | Details |
|------|---------|
| **Vulnerability Type** | Command Injection (CWE-78) |
| **Impact** | Remote Code Execution (RCE) |
| **Affected Product** | Anthropic Claude Code |
| **Affected Versions** | <= 2.1.88 |
| **Affected Platforms** | Windows only |
| **Discoverer** | [@realguoxiufeng](https://github.com/realguoxiufeng) |
| **Discovery Date** | April 1, 2026 |

## Description

In Anthropic Claude Code for Windows, the `openFileInExternalEditor()` function does not properly sanitize filenames before constructing a shell command with `shell: true`. An attacker who can create a file with a malicious filename can execute arbitrary commands when the victim opens the file with Claude Code.

## Proof of Concept

```powershell
# Create malicious file
New-Item -Path '"; calc;"' -ItemType File

# In Claude Code
/edit "; calc;"

# Result: calc.exe opens → RCE confirmed
```

## Reproduction Reference

https://github.com/realguoxiufeng/claude-code-sourcemap

## CVSS Score

**CVSS:3.1/AV:L/AC:L/PR:N/UI:R/S:U/C:H/I:H/A:H** → **7.8 (High)**

## Fix

The fix is to use argument array invocation instead of string interpolation with `shell: true`, matching the existing safe implementation on non-Windows platforms:

```typescript
// Fixed code
const args = [base, ...editorArgs, ...(useGotoLine ? [`+${line}`, filePath] : [filePath])];
result = spawnSync(base, args, syncOpts);
```

## Contact

For more information, contact the discoverer via GitHub issues.
