# DP LoRA Fine-Tuning with Opacus (Minimal Script)

This script performs **differentially private (DP) fine-tuning** of a causal language model using:

- **🤗 Transformers** (`AutoModelForCausalLM`, `AutoTokenizer`)
- **PEFT / LoRA** (`get_peft_model`, `LoraConfig`)
- **Opacus** (DP-SGD via `PrivacyEngine`)
- Optional **bitsandbytes 8-bit** loading (`BitsAndBytesConfig(load_in_8bit=True)`)

## What the script does

1. **Load dataset** with `datasets.load_dataset(DATASET_NAME)`.
2. **Select a subset** (`SUBSET_SIZE`) from the split `DATASET_SPLIT`.
3. **Tokenize** to a fixed length (`MAX_LENGTH`) and keep `input_ids`.
4. **Load the base model**, optionally in **8-bit** using bitsandbytes.
5. **Attach LoRA adapters** so only a small subset of parameters is trainable.
6. **Enable DP-SGD** using Opacus:
   - Target privacy budget: `EPSILON`
   - Failure probability: `DELTA`
   - Clipping: `MAX_GRAD_NORM`
7. **Train** for `EPOCHS` with `BatchMemoryManager` to reduce peak memory usage.
8. **Save** the trained weights (`state_dict`) to disk.

## Key configuration variables

### Model
- `BASE_MODEL_ID`: model identifier on Hugging Face (or a local model path).
- `MAX_LENGTH`: max token length for truncation/padding.
- `DEVICE_INDEX`: GPU index (if CUDA is available).

### Training / DP
- `BATCH_SIZE`: logical batch size.
- `MAX_PHYSICAL_BATCH_SIZE`: microbatch size used by `BatchMemoryManager` to avoid OOM.
- `EPOCHS`, `LR`, `ADAM_EPS`: optimizer configuration.
- `EPSILON`: target DP budget (smaller means stronger privacy).
- `DELTA`: DP failure probability (commonly `1 / N`).
- `MAX_GRAD_NORM`: clipping norm for per-sample gradients.

### LoRA
- LoRA is configured using:
  - `r`, `lora_alpha`, `lora_dropout`
  - `target_modules` (must match your model architecture)

Only LoRA parameters are trained, which reduces compute and memory compared to full fine-tuning.
