# cellranger_vdj_multi

**Fork of [lilab-bcb/cumulus](https://github.com/lilab-bcb/cumulus) adding CellRanger Multi support for non-multiplexed RNA + VDJ experiments.**

---

## What this fork adds

The upstream cumulus workflow routes VDJ data to `cellranger vdj`. This is correct for VDJ-only runs, but **fails for combined GEX + VDJ experiments** because those require `cellranger multi`. This fork fixes that.

### Fix 1 — Route RNA + VDJ combinations to `cellranger multi`

When a sample sheet links an RNA library with a VDJ library (`vdj`, `vdj_t`, `vdj_b`, or `vdj_t_gd`), the workflow now correctly routes to `cellranger multi` instead of `cellranger vdj`.

### Fix 2 — No multiplexing requirement for simple RNA + VDJ runs

The original `cellranger_multi.wdl` raised an error if no OCM / HTO / CMO / Flex sample file was provided:

```
Exception("Cannot locate OCM, HTO, CMO, or Flex sample file!")
```

This blocked non-multiplexed GEX + VDJ runs. This fork removes that requirement. CellRanger Multi supports running without a `[samples]` section, and now so does this workflow.

### Fix 3 — VDJ reference tarball extraction

The original workflow passed the `.tar.gz` path directly to CellRanger, causing a `regions.fa not found` error. This fork extracts the tarball to a local directory and points the `[vdj]` section to that directory:

```bash
tar xf vdj_ref.tar.gz -C vdj_dir --strip-components 1
# multi.csv [vdj] section → /path/to/vdj_dir
```

---

## Upstream changes included (v10.0.0 sync, May 2026)

This fork is synced to upstream cumulus as of May 2026 and includes:

- **CellRanger 10.0.0** — updated default version
- **Flex_v2 support** — `flex-v1` and `flex-v2` data types alongside the existing `frp`
- **Unzipped `.fastq` input** — accepts uncompressed FASTQ files in addition to `.fastq.gz`
- **Library-level chemistry for Flex** — per-library chemistry column in `[libraries]` section of `multi.csv`; needed when Flex and CITE-Seq libraries use different probe barcode read pairs
- **`--create-bam` flag** — CellRanger 8.0+ uses `create-bam,true/false` instead of `no-bam`

---

## Usage (Terra / Dockstore)

Import the workflow from Dockstore:
[https://dockstore.org/workflows/github.com/Shts2123/cellranger_vdj_multi:master](https://dockstore.org/workflows/github.com/Shts2123/cellranger_vdj_multi:master)

### Sample sheet for a simple RNA + VDJ run

```
Sample,Reference,Flowcell,DataType,Link
PBMC_GEX,GRCh38_2020-A,gs://bucket/fastqs/gex,rna,PBMC
PBMC_VDJ,GRCh38_vdj_v7.1.0,gs://bucket/fastqs/vdj,vdj,PBMC
```

- The `Link` column groups GEX and VDJ libraries — samples sharing a link are processed together with `cellranger multi`.
- `GRCh38_2020-A` and `GRCh38_vdj_v7.1.0` are acronyms resolved via `index.tsv` to GCS paths.
- No `[samples]` section is required for non-multiplexed runs.

---

## Original repository

**Upstream:** [https://github.com/lilab-bcb/cumulus](https://github.com/lilab-bcb/cumulus)
**Upstream documentation:** [https://cumulus.readthedocs.io](https://cumulus.readthedocs.io)
**License:** BSD 3-Clause (same as upstream)

The upstream cumulus project supports a much broader set of workflows (STARsolo, Spaceranger, Demultiplexing, Cumulus, etc.). This fork focuses exclusively on the CellRanger workflow with the RNA + VDJ multi fix.
