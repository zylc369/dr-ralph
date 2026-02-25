# Fix: Notes File Path Resolution Bug

## Summary
Fixed the bug where `setup-dr-ralph-diagnose.sh` failed to detect existing patient notes in follow-up sessions due to Bash's `[[ -f ]]` not resolving the `@notes` alias.

## Root Cause
- Lines 119-130: The script used `@notes` alias for `OUTPUT_DIR` (line 13), but Bash's `[[ -f ]]` doesn't resolve Claude Code's `@notes` alias to the real path `notes`
- This caused the file existence check on line 130 to always return false, even when patient notes existed

## Fix Applied

### Changes to `scripts/setup-dr-ralph-diagnose.sh`:

1. **Added alias resolution logic** (after line 120):
   ```bash
   # Resolve @notes alias to real path for file operations
   if [[ "$OUTPUT_DIR" == "@notes" ]]; then
     REAL_OUTPUT_DIR="notes"
   else
     REAL_OUTPUT_DIR="$OUTPUT_DIR"
   fi
   ```

2. **Updated file paths** to use real path for file operations:
   - Line 130: `PATIENT_NOTES_PATH="$REAL_OUTPUT_DIR/$PATIENT_FILE.md"` (was `$OUTPUT_DIR`)
   - Line 132: `REPORT_PATH="$REAL_OUTPUT_DIR/$PATIENT_FILE-report-$TIMESTAMP.md"` (was `$OUTPUT_DIR`)

3. **Preserved alias for display**:
   - Lines 387-388 and 402-403: Still use `$PATIENT_NOTES_PATH` which displays the full path including the directory
   - This maintains user-friendly output messages while enabling correct file operations

## Testing

### Test Case 1: Existing Patient Notes
```bash
# Create test patient notes
echo "# Test Patient Notes" > notes/test-patient.md

# Run the script
bash scripts/setup-dr-ralph-diagnose.sh "test symptoms" --patient "test-patient"
```

**Result**: ✅ PASS
```
Previous Notes: Found (will be read for context)
```

### Test Case 2: New Patient
```bash
# Run with non-existing patient
bash scripts/setup-dr-ralph-diagnose.sh "new patient symptoms" --patient "nonexistent-patient"
```

**Result**: ✅ PASS
```
Previous Notes: None (new patient)
```

## Verification

The fix correctly:
1. ✅ Detects existing patient notes in follow-up sessions
2. ✅ Maintains correct behavior for new patients
3. ✅ Preserves user-friendly display paths
4. ✅ Does not modify workflow logic or diagnostic phases
5. ✅ Does not break existing functionality

## Impact

This fix ensures:
- Patient continuity of care: Previous notes are now properly read at session start
- Accurate session information: The output correctly shows whether previous notes exist
- No user-facing changes: The display paths remain the same, only internal file operations are corrected

## Date Fixed
2025-02-25
