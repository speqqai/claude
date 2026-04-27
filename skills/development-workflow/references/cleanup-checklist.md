# Code cleanup checklist

## Step 1: Automated cleanup

Run the project's quality checks:

```bash
npm run quality
```

This runs lint (strict), CSS lint, format check, and deadcode detection (knip). Fix everything it reports.

## Step 2: Manual cleanup

Go through every file you changed or created and verify:

- [ ] No commented-out code blocks (delete them, git has the history)
- [ ] No unused imports
- [ ] No unused variables, functions, or types
- [ ] No orphaned files (files created during implementation that are no longer needed)
- [ ] No leftover debug logging (`console.log`, `console.debug`)
- [ ] No temporary workarounds that were superseded by the final implementation
- [ ] No duplicate logic — if you wrote the same pattern in two places, consolidate
- [ ] Backup files removed (`.bak`, `.old`, `-backup` suffixes)

## Step 3: Verify cleanup didn't break anything

After cleanup:
1. `npm run build` succeeds
2. `npm test` passes
3. `npm run quality` passes with zero issues

## Gate

`npm run quality` passes. No dead code, no commented-out code, no orphaned files.
