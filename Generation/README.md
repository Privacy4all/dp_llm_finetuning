# Synthetic Note Generation from a DP-LoRA Checkpoint

This script loads a **base causal language model** and attaches **LoRA adapters** (PEFT), then loads a saved **DP-LoRA checkpoint** (`state_dict`) produced during finetuning (e.g., from an Opacus DP-SGD run). It then generates synthetic notes from a set of prompts and saves the outputs as pickle files.

## What the script does

1. **Loads prompts** from `PROMPTS_PATH` (a pickle dictionary, values are strings).
2. **Builds tokenizer + model**:
   - Loads tokenizer with left padding (recommended for decoder-only generation with batching).
   - Loads the base model in **8-bit** using bitsandbytes.
   - Reconstructs the LoRA adapter modules (must match training).
3. **Loads checkpoint** from `CKPT_PATH` into the LoRA-wrapped model.
   - Strips Opacus `"_module."` key prefixes if present.
   - Logs missing/unexpected keys.
4. **Generates outputs in batches** using `model.generate(...)`.
5. **Repeats generation** `NUM_REPEATS` times (useful for producing multiple synthetic samples per prompt).
6. **Saves outputs** to `synth_{RUN_TAG}_{rep}.pkl` in the same directory as the checkpoint.

## Inputs

### Model + checkpoint
- `BASE_MODEL_ID`: HF model id or local model path (must match the base model used in training).
- `CKPT_PATH`: path to a saved `model.state_dict()` containing LoRA weights (and potentially base model keys depending on how it was saved).

### Prompts
- `PROMPTS_PATH`: pickle file containing a dict of prompts (values are prompt strings).

## Outputs

For each repeat `rep = 1..NUM_REPEATS`, the script writes:
- `synth_{RUN_TAG}_{rep}.pkl`

Each output pickle contains:
- a list of generated strings (one per prompt, in prompt order).

## Generation behavior

### Tokenization
- Uses **left padding** (`padding_side="left"`) for decoder-only models.
- Truncates prompts to `MAX_PROMPT_LEN`.
- Uses `add_special_tokens=False` to avoid injecting BOS/EOS into the prompt.

### Sampling controls
- `DO_SAMPLE`: sampling vs greedy decoding.
- `TEMPERATURE`, `TOP_K`, `REPETITION_PENALTY`: typical sampling knobs.
- `MAX_NEW_TOKENS`: number of tokens to generate per prompt.
