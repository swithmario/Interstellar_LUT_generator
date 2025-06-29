Excellent point. You are absolutely right to correct this. A README must reflect the *current, working reality* of the project, not just a potential or idealized workflow. My previous versions glossed over the meticulous manual labor involved in the alignment phase, which is a critical part of your current process.

Let's rewrite the relevant sections with the precision and detail they deserve, explicitly describing your manual `Difference` mode technique and clearly stating the vision for a future automated tool.

Here is the revised, more accurate, and more detailed `README.md`.

---

# Interstellar LUT Generator

**Restoring IMAX 70mm fidelity to the 4K HDR Blu-ray of *Interstellar***

This project provides a complete toolchain to restore the theatrical color fidelity of *Interstellar* by generating high-quality 3D LUTs. It mathematically matches the color science of authentic IMAX film cell scans with the precision and detail of the 4K HDR Blu-ray, using a combination of manual alignment in DaVinci Resolve and an automated Python pipeline.

---

## 🎬 The Philosophy: Why This Project Exists

Digital masters of *Interstellar*, particularly the 4K Blu-ray, often feel overly crisp, synthetic, and color-biased when compared to the organic, filmic beauty of the original IMAX 15/70 projection. This project aims to bridge that gap.

The workflow achieves this by:

1.  **Sourcing** a bit-perfect digital master from the 4K HDR Blu-ray.
2.  **Referencing** a curated library of genuine, high-resolution film cell scans sourced from the collector's market (e.g., eBay).
3.  **Manually Aligning** a reference scan to its corresponding Blu-ray frame with pixel-level precision.
4.  **Composite-Grading** a single "ground truth" frame using Resolve’s **Luminosity** blend mode to capture the film look.
5.  **Automating** a Python script that analyzes the "before" (Blu-ray) and "after" (Composite) image pair and builds a DaVinci-compatible `.cube` LUT to replicate the transformation.

By generating LUTs on a per-group or per-shot basis, this pipeline recovers the original color intent without sacrificing the sharpness and dynamic range of the digital master.

---

## 📦 Repository Contents

```text
Interstellar_LUT_Generator/
├── assets/
│   ├── scans/              # Your library of web-scraped film cell stills, organized by shot or group
│   └── tiff_pairs/         # A suggested folder to keep your exported Original/Corrected TIFFs
├── scripts/
│   ├── make_resolve_lut_OK.py  # The main production LUT generator (NumPy + PIL)
│   └── align_frames_cv2.py     # Proof-of-concept for the future automated alignment tool
├── .gitignore
└── README.md                  # You are here
```

---

## 🛠️ Phase 1: Prerequisites & Setup

### 1. Source the 4K Blu-ray
Use **MakeMKV** to create a lossless backup of your `Interstellar` 4K UHD disc. This provides the pristine digital master.

### 2. Analyze Media with MediaInfo
Extract the exact technical metadata from the resulting `.mkv` file. This is **non-negotiable** for setting up a correct color pipeline.

| Property | Value |
| :--- | :--- |
| Format | Matroska |
| Video Codec | HEVC (H.265) |
| Resolution | 3840×2160 |
| Frame Rate | 23.976 fps (or 24.000) |
| Color Space | BT.2020 |
| Transfer Characteristics | ST.2084 (PQ) |
| Bit Depth | 10 bits |

### 3. Collect & Organize Film Scans
This project is powered by a library of ~2,000 high-quality IMAX 70mm film cell scans, scraped from eBay sellers like "interstellar-film-cellar". For this workflow to function, you must organize your own collection of scans into a logical folder structure, grouped by scene or shot look.

---

## 📺 Phase 2: DaVinci Resolve Workflow

### A. Project & Timeline Settings
Set up your project to match the source media exactly.

| Setting | Value |
| :--- | :--- |
| Color Science | `DaVinci YRGB Color Managed` |
| Timeline Color Space | `Rec.2100 ST2084` (PQ) |
| Timeline Resolution | 3840×2160 |
| Timeline Frame Rate | 23.976 fps |

### B. Frame Alignment: The Manual Method (Current Workflow)
This is currently the most labor-intensive part of the process, requiring both frame discovery and precise alignment.

**Part 1: Frame Discovery**
1.  Import your Blu-ray `.mkv` file into the Media Pool.
2.  Create a timeline from this clip.
3.  Take one of your film cell scans (e.g., from the "Farm" group).
4.  You must now **manually scrub through the entire timeline** to find the single, exact frame that visually matches your film scan. Use character positions, background details, and motion to guide you.

**Part 2: Precision Alignment via Difference Mode**
Once you have located the correct frame on the timeline:
1.  Place the Blu-ray clip on **Track 2 (V2)**.
2.  Place your film scan image (PNG, JPEG, etc.) on **Track 1 (V1)**, directly below the Blu-ray clip at the matching frame.
3.  Select the Blu-ray clip on **V2**. In the **Inspector**, navigate to the **Composite** settings.
4.  Change the **Composite Mode** from `Normal` to **`Difference`**.
5.  The viewer will now display a near-black image, highlighting only the mathematical differences between the two layers.
6.  Select the film scan clip on **V1**. In the **Inspector**, use the **Transform** controls (Zoom, Position, Rotation Angle).
7.  Meticulously adjust these transform properties. Your goal is to **minimize the total brightness of the screen**, making the viewer as close to pure black as possible. This indicates that the two images are perfectly aligned structurally.
8.  **CRITICAL:** Once alignment is perfect, set the Blu-ray clip's **Composite Mode** back to **`Normal`** before proceeding.

### C. Creating the Reference Stills for the LUT Generator
With the frames aligned, you can now export the "before" and "after" data.

| Export | Composite Mode | Filename | Format | Bit Depth | Compression |
| :--- | :--- | :--- | :--- | :--- | :--- |
| Corrected | **Luminosity** | `Corrected_<shot>.tif` | TIFF | 16-bit | None |
| Original | **Normal** | `Original_<shot>.tif` | TIFF | 16-bit | None |

1.  On the **Color Page**, select the **V2** Blu-ray clip. Set its **Composite Mode** to **`Luminosity`**.
2.  Right-click the viewer → **Grab Still** → Export from Gallery as a 16-bit TIFF named `Corrected_<shot>.tif`.
3.  Set the **Composite Mode** back to **`Normal`**.
4.  Right-click the viewer → **Grab Still** → Export from Gallery as a 16-bit TIFF named `Original_<shot>.tif`.

---
## 🐍 Phase 3: Python LUT Generation

1.  **Install dependencies:**
    ```bash
    pip install numpy pillow
    ```
2.  **Run the script:** Place your newly exported `Original_*.tif` and `Corrected_*.tif` files in the repository folder. Then run:
    ```bash
    python scripts/make_resolve_lut_OK.py
    ```
3.  The script will generate a `.cube` file, correctly formatted for DaVinci Resolve.

---
## 🚀 Future Development: The Automated Aligner

The manual alignment process described above is a significant bottleneck. A separate, future project is planned to create a dedicated tool that will automate this entire pipeline. The vision for this repository includes:

*   **Automated Frame Detection:** A script that takes any film cell scan as input and uses computer vision (e.g., OpenCV feature matching) to instantly locate its corresponding frame number within the full-length movie file.
*   **Automated Geometric Alignment:** Once the frame is identified, the tool will calculate and apply the precise transformation (position, scale, rotation, and perspective) required for a perfect overlay.

The `align_frames_cv2.py` script in this repository serves as an early proof-of-concept for this future goal.

---

## ❤️ Credits

-   Built by **Swithin Feely**, inspired by a deep respect for color science and the pursuit of preserving Christopher Nolan’s original IMAX vision.
-   This project would not be possible without the dedicated efforts of the eBay film cell scanning community.
-   Contributions, bug reports, and ideas are welcome. Please open an issue or pull request on GitHub.