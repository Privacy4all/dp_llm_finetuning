# (Potential) PHI Leakage Checker (Exact-Match Scan)

This script performs a **verbatim (exact-string) leakage scan** of synthetic clinical notes against **i2b2-style XML PHI annotations**.

## What it does

For each real note:

1. Loads the corresponding **i2b2 XML** file and reads:
   - The original note text from `<TEXT>`.
   - PHI tag spans from `.//TAGS/*` (attributes like `TYPE`, `text`, `start`, `end`).

2. Loads the **synthetic note** for that same note index from a pickle file.

3. For each PHI tag whose `TYPE` is in `PHI_LIST`, it searches the synthetic note for **all exact occurrences** of the PHI `text`.

4. If a match is found, it prints:
   - The file name
   - The PHI string and type
   - A short context window around the PHI in the **original note**
   - A short context window around each match in the **synthetic note**

At the end, it prints a summary count: **how many files contained at least one exact-match hit**.

## Inputs

- `SYNTH_PKL_PATH`: Pickle containing `synth_notes`  
  Expected shape: a list (or list-like) where `synth_notes[i]` is the synthetic note for the *i-th* real note.
- `PROMPT_PKL_PATH`: Pickle containing `prompts` (a dict)  
  The script uses `list(prompts.keys())` to define file ordering.  
  Each key `k` maps to an XML file name `{k}.xml` in `I2B2_XML_DIR`.
- `I2B2_XML_DIR`: Directory containing i2b2 XML files.

## Output

Console output only (prints matches and contexts), plus:
- `Total files with any leakage: N`

## Important caveat: exact matches are not always true leakage

This scan is **string-based**. It flags a PHI item whenever the annotated `text` appears verbatim in the synthetic note.

However, **multiple exact matches are not always true leakage**:
- A short PHI string can occur naturally as part of other words/phrases.

Therefore, **every flagged match should be manually reviewed** using the printed context to confirm whether it is a real PHI reappearance.
