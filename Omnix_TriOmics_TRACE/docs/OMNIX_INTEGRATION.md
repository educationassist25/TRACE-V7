# Adding Integrix to Omnix (when ready)

Integrix v1 is fully standalone. The plan below adds it to the Omnix portal without changing how any existing module analyses data.

## Step 1 — Copy the engine

```text
Omnix_upload_part5_triomics/
  apps/triomics/
    app.py                 <- this repo's app.py (portal wrapper, step 2)
    trace_core/            <- unchanged
    kb/                    <- unchanged (about 1 MB)
    demo_data/             <- unchanged (about 2 MB)
    assets/omnix_logo_web.png
```

Add `mofapy2` and `h5py` to the portal `requirements.txt`. Everything else is already in Omnix.

## Step 2 — Register it as a portal page

1. In `omnix_portal/config.py`, add a platform entry: `"triomics"`, labelled "Tri-Omics Integration".
2. Decide which plans include it. The Phase 1 recommendation is Multi-Omics and Lab.
3. In `portal.py`, route to `apps/triomics/app.py` like the other apps.
4. In `app.py`:
    - remove `st.set_page_config` (the portal sets it);
    - keep everything else;
    - gate **Run Integrix** behind the existing key check (`keysess`), as in the other modules;
    - keep the demo free, like the other modules.

## Step 3 — "Export to Integrix" button (the only change in the existing modules)

Each module writes one zip with its **normalized** matrix, the sample metadata and a small `info.json` sidecar
(module, scale, normalization, imputation, batch correction). The format is defined in
[`OMNIX_EXPORT_CONTRACT.md`](OMNIX_EXPORT_CONTRACT.md); the app's **Example of an Omnix export** button downloads a
complete example.

Add the button next to the existing downloads in:

- `apps/metabolomics/app.py` (Metabolomics and Lipidomics)
- the proteomics app
- the transcriptomics app

No analysis code changes. Inside the Omnix platform, Integrix reads these zips with `trace_core/omnix_io.py`, skips the log transform,
imputes only remaining gaps, and switches batch correction off for layers the module already corrected.

## Step 4 — Tests

Copy `tests/test_trace.py` to `tests/test_triomics.py`. The existing metabolomics, fluxomics and portal tests should still pass unchanged.

## Optional knowledge-base add-ons (any internet-connected computer)

```bash
python -m trace_core.kb_build --skip-core --kegg --collectri --string --out apps/triomics/kb
```

Commit the new parquet files and the updated `kb_manifest.json`.
