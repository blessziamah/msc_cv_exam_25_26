# Feature-Based Image Matching and Panorama Construction

**CSCD608 — Advanced Computer Vision, University of Ghana**

`imp.ipynb` is a compact comparison of **SIFT** and **ORB** for feature matching and panorama construction. It uses the local datasets already in `data/`; it has no download, archive-extraction, or network code.

The pipeline detects features, applies Lowe's ratio test, estimates a homography with OpenCV RANSAC, and feather-blends warped images into a panorama. `cv2.Stitcher_create` is not used.

## Run the notebook

```bash
uv sync                       # or: pip install -r requirements.txt
uv run jupyter lab 22424151_bless_ziamah.ipynb
```

For a non-interactive run:

```bash
uv run jupyter nbconvert --to notebook --execute --inplace 22424151_bless_ziamah.ipynb \
  --ExecutePreprocessor.timeout=1800
```

## Required local data

The notebook validates these files before starting and raises a clear error if any are absent.

| Folder | Required content | Used for |
|---|---|---|
| `data/opencv_samples/boat/` | `boat1.jpg`–`boat6.jpg` | consecutive matching and a six-image panorama |
| `data/opencv_samples/newspaper/` | `newspaper1.jpg`–`newspaper4.jpg` | consecutive matching and a four-image panorama |
| `data/oxford_vgg/{bark,boat,graf,wall,leuven,bikes}/` | `img1`–`img6` and `H1to6p` | SIFT/ORB robustness comparison against ground truth |

The Oxford study compares `img1 → img6` for each of the six sequences and both detectors (12 comparisons). Oxford images remain at their original resolution because their supplied homographies use original pixel coordinates. OpenCV panorama inputs are resized to a 600-pixel longest side to keep the blended canvases manageable.

## Outputs

Running the notebook updates the existing `results/` directory:

```text
results/
  comparison_table.csv                         28 pairwise comparison rows
  metrics.json                                 configuration and run records
  panoramas/
    imp_opencv_boat_{SIFT,ORB}.jpg
    imp_opencv_newspaper_{SIFT,ORB}.jpg
  summary/
    imp_boat1_boat2_{SIFT,ORB}_inliers.png
    imp_oxford_img1_to_img6_reprojection_error.png
```

Each row records detector, keypoint counts, ratio-test matches, RANSAC inliers, inlier ratio, runtime, and status. Oxford rows also include mean reprojection error against `H1to6p`.
