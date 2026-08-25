# Feature-Based Image Matching and Automatic Panorama Construction

**CSCD608 — Advanced Computer Vision, University of Ghana**

A complete feature-matching and image-stitching pipeline implemented from first principles, used
to compare **SIFT** against **ORB** across three datasets. The entire deliverable is a single
notebook, `main.ipynb`, which writes every result it produces into `results/`.

`cv2.Stitcher_create` is not used anywhere — the homography solver, the robust estimator, the
canvas geometry and the blending are all written out so that each intermediate stage is
inspectable. No deep learning and no pre-trained recognition models are involved.

---

## Quick start

```bash
uv sync                       # or: pip install -r requirements.txt
uv run jupyter lab main.ipynb # then: Restart Kernel and Run All Cells
```

Headless, exactly as the committed run was produced:

```bash
uv run jupyter nbconvert --to notebook --execute --inplace main.ipynb \
    --ExecutePreprocessor.timeout=5400
```

A full run takes roughly **45 seconds** on an M-series Mac once the datasets are cached, plus
about a minute the first time for the ~98 MB of downloads. `nbconvert --execute` aborts on the
first cell that raises, so a zero exit status is itself the top-to-bottom guarantee.

---

## Datasets

The notebook downloads everything it needs; nothing has to be fetched by hand. Downloads are
idempotent, so re-running does not re-fetch.

| Folder | Contents | Source |
|---|---|---|
| `data/opencv_samples/` | `boat1..6.jpg`, `newspaper1..4.jpg` | OpenCV `opencv_extra` stitching test data |
| `data/oxford_vgg/` | `bark`, `boat`, `graf`, `wall`, `leuven`, `bikes` — each `img1..6` plus ground-truth homographies `H1to2p..H1to6p` | [Oxford VGG Affine Covariant Regions](https://www.robots.ox.ac.uk/~vgg/research/affine/) |
| `data/custom/` | your own photographs, one folder per scene | you |

`data/` is git-ignored apart from `data/custom/README.md`.

### Adding your own photos (Experiment 2)

1. Create `data/custom/<scene_name>/`.
2. Add **3 or more overlapping images** (`.jpg`, `.jpeg`, `.png`).
3. Name them so alphabetical order is capture order: `img_01.jpg`, `img_02.jpg`, `img_03.jpg`.
4. Re-run Section 14 (or the whole notebook). Every later summary table and chart picks the
   scene up automatically.

Shooting tips: rotate the camera about its own optical centre rather than walking sideways (a
homography exactly models a pure rotation; translation past objects at different depths creates
parallax that no homography can undo), aim for 30–50 % overlap, and lock exposure and focus if
you can. See `data/custom/README.md`.

**If `data/custom/` is empty the notebook does not fail** — Experiment 2 prints a skip notice and
everything else runs to completion. That is the state of the committed run.

---

## What gets produced

```
results/
  metrics.json              nested dataset -> scene -> pair -> detector -> metrics (incl. every H)
  comparison_table.csv      flat, one row per run — the table to paste into a report
  log.txt                   append-only human-readable trace
  panoramas/                <dataset>_<scene>_<detector>.jpg
  visualizations/<exp_id>/  7 PNGs per (pair, detector): keypoints A/B, initial matches,
                            RANSAC inliers, before/after comparison, warped A, pairwise panorama
  summary/                  keypoint / inlier-ratio / timing bar charts, stage-time breakdown,
                            per-sequence Oxford degradation plots, RANSAC trajectory,
                            feather-vs-paste blending demo
  robustness/               per-pair match figures + <sequence>_summary.csv for Task 12
  failures/                 the deliberate failure-case figures from Section 17
```

Section 18 audits this tree and asserts that every promised artifact exists, so a successful run
ends with `ALL EXPECTED ARTIFACTS PRESENT`.

`results/` is **git-ignored**: a full run writes about 270 MB, most of it lossless PNG
photographs, which version control handles poorly. Nothing is lost by this — `main.ipynb` is
committed *with its outputs*, so every table and figure is readable in a fresh clone without
running anything, and Restart & Run All regenerates the whole tree in about 45 seconds.

Metrics recorded per (detector, image pair): keypoints in A and B, matches after the ratio test,
RANSAC inliers, inlier ratio, detection / description / matching / RANSAC / total time in ms, the
estimated 3×3 homography, and — for Oxford, where ground truth exists — the mean reprojection
error in pixels over a 20×20 grid.

---

## Reproducibility

`numpy`, `random` and `cv2` RNGs are seeded with 42 in Section 1 and re-seeded at the top of every
experiment, so individual sections can be re-run in isolation and still reproduce. `CFG` in
Section 2 is the single control surface: detectors, ratio-test threshold, RANSAC threshold and
iteration cap, resize policy, blend mode and dataset paths all live there, and everything
downstream reads from it.

Set `CFG["homography_method"] = "scratch"` to route the bulk experiments through the from-scratch
RANSAC instead of OpenCV's. Both are verified against each other in Sections 7.1 and 8.1.

---

## Notes and deviations

1. **`opencv-python`, not `opencv-contrib-python`.** SIFT moved into the OpenCV main module in
   4.4 when its patent expired, so contrib is not needed; Section 1 asserts that
   `cv2.SIFT_create` is constructible. Verified against OpenCV 5.0.0 on Python 3.14.
2. **Oxford images are never resized.** The ground-truth homographies are expressed in original
   pixel coordinates, so `CFG["resize_datasets"]["oxford_vgg"]` is `False` and the Oxford loader
   asserts `scale == 1.0` for all 36 images. Resizing without conjugating
   `H' = S₂ · H · S₁⁻¹` would produce reprojection errors that look plausible and mean nothing.
   Non-Oxford images are capped at 1000 px on the longest side to keep runtime sane.
3. **Inline figures are JPEG.** Almost every figure here is a photograph, where PNG's lossless
   encoding costs roughly 5× the bytes for no visible benefit — which matters because the
   committed notebook keeps its outputs. Every figure is *also* written to `results/` as a
   lossless PNG, so the archived artifacts are unaffected.
4. **Inline figure budget.** Experiments 1 and 2 display all seven per-pair artifacts inline.
   Experiment 3 is 60 pair runs, so it writes every match figure to `results/robustness/` and
   displays per-sequence summary plots plus a representative montage. Nothing is missing from
   `results/` either way.
5. **Failure-case substitution.** The brief suggests a near-uniform-texture region from a custom
   scene. Section 17 Case C1 covers that by *searching* the OpenCV sample image for its
   lowest-variance window (with the highest-variance window as a control), and Case C2 adds the
   Oxford `bikes` defocus-blur sequence, which starves the detectors the same way while keeping
   ground truth available.
6. **Failure-case runs are tagged** `dataset == "failure_cases"` and are recorded after Section
   16's aggregates are computed, so these deliberately degenerate scenes do not contaminate the
   headline comparison. They are still written to `metrics.json` and `comparison_table.csv`.
