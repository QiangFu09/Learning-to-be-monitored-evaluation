# Separate checkpoint releases

Publish model artifacts separately from this general code repository. A model hosting repository can provide an immutable download revision; the code README can link to it after upload and verification. Do not commit weights, optimizer states or private experiment directories to the code repository.

## Proposed initial scope

Release one author-approved evaluated adapter for each supported task model, rather than automatically publishing an entire tuning sweep. Keep the original base model as an external pinned dependency. This avoids distributing a second full copy of the base and preserves the distinction between an adapter and a merged model.

Each model release should contain:

- The LoRA `adapter_model.safetensors` and portable `adapter_config.json`.
- The saved tokenizer/chat-template files needed by its loading profile.
- A sanitized checkpoint manifest with base model ID/revision, adapter method/configuration, optimizer step, training objective and relevant data provenance.
- SHA-256 for all release payload files and an immutable model-repository revision.
- A model card describing intended use, limitations, licensing, citation and the exact evaluation loading command.
- A reference to the compatible code release and evaluation profile, not an unversioned instruction to use whichever configuration is latest.

Checkpoints make evaluation possible without retraining. Exact numerical replication additionally depends on the evaluation configuration, sampling seeds, hardware and inference runtime. The public CLI records user-supplied seeds; do not claim bitwise-identical results from weights alone.

## Packaging checks

1. Confirm the selected source checkpoint against the evaluation provenance. Never select a different adapter merely because its directory name looks similar.
2. Hash the original tensors and preserve their bytes. Do not merge, quantize or convert precision for convenience without an explicitly separate release identity.
3. Inspect metadata for absolute paths, account names, access tokens and private data references. Replace deployment-only base paths with the public model ID where needed, record the metadata-only transformation, and regenerate the release manifest. Preserve original provenance privately.
4. Verify the exported package with the evaluator's checkpoint command. Qwen's runtime key-mapped copy remains a derived deployment artifact; its mapping report must prove tensor equality.
5. Run a bounded loading/evaluation canary from the **downloaded** immutable release, using the base revision and tokenizer specified by the model card.
6. Review base-model terms and training-data redistribution restrictions. Adapter release and tokenizer redistribution are separate questions from the code license.
7. Publish only with the author's approval, then replace proposed links with verified immutable model-repository links.

This file is a release checklist, not an assertion that weights have already been uploaded or licensed for redistribution. Checkpoint selection details belong in the individual model cards; the evaluation implementation remains general.
