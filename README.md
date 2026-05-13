# Prithvi-EO Burn Scar Segmentation 

End-to-end demo applying the Prithvi-EO burn scar model to HLS imagery. MTBS perimeters is used as a ground truth for comparison and validation.

Notebook: [notebook/Prithvi_Burnscar_Park_Fire_2024_nb.ipynb](notebook/Prithvi_Burnscar_Park_Fire_2024_nb.ipynb)

## What the notebook does

1. Authenticates with NASA Earthdata and searches HLS granules for the target date.
2. Downloads HLS scenes, builds AOI-clipped tile mosaics, and previews them.
3. Loads the Prithvi-EO burn scar checkpoint and runs tiled inference (512x512 chips).
4. Masks cloud, shadow, snow, and water using the HLS Fmask band.
5. Composites tile predictions onto a common AOI grid.
6. Fetches the MTBS fire perimeter as ground truth and rasterizes it to the AOI grid.
7. Computes accuracy, precision, recall, F1, IoU, and mIoU.
8. Saves a confusion matrix, a 5-panel overview, and an interactive folium map.

## Outputs

Written to `$HOME/r2o_burn/burn_demo_park_2024_nb/`:

- `01_mosaic_preview.png`, `02_cm_metrics.png`, `03_overview.png`
- `map_aoi.html` — standalone interactive map
- `metrics_aoi.csv`, `metrics_aoi.json`
- `tile_mosaics/`, `predictions_final/`, `hls_data/`

## Prerequisites

- NASA Earthdata account: https://urs.earthdata.nasa.gov/users/new
- GPU strongly recommended. CPU works but inference is slow.

## Run on Google Colab (recommended)

1. Open the notebook in Colab.
2. `Runtime` -> `Change runtime type` -> select **T4 GPU**.
3. Run the first cell to install dependencies (add a cell if missing):
   ```
   !pip install -q earthaccess rasterio geopandas folium gdown terratorch pyyaml
   ```
4. When prompted by `earthaccess.login(persist=True)`, enter your Earthdata username and password.
5. Run all cells top to bottom.

## Run locally

1. Confirm a CUDA-capable GPU is available (`nvidia-smi`). The notebook falls back to CPU if not.
2. Create and activate an environment (Python 3.10+):
   ```
   conda create -n prithvi-burn python=3.10 -y
   conda activate prithvi-burn
   ```
3. Install dependencies:
   ```
   pip install earthaccess rasterio geopandas folium gdown terratorch pyyaml \
               numpy pandas torch matplotlib shapely pillow requests
   ```
4. Configure Earthdata credentials once (pick one):
   - Create `~/.netrc`:
     ```
     machine urs.earthdata.nasa.gov
       login YOUR_EARTHDATA_USERNAME
       password YOUR_EARTHDATA_PASSWORD
     ```
     then `chmod 600 ~/.netrc`
   - Or export `EARTHDATA_USERNAME` and `EARTHDATA_PASSWORD`
   - Or run `earthaccess.login(persist=True)` once interactively
5. Launch Jupyter and open the notebook:
   ```
   jupyter lab notebook/Prithvi_Burnscar_Park_Fire_2024_nb.ipynb
   ```
6. Run all cells top to bottom.

## Configuration

The event is configured in section 2 of the notebook:

```python
cfg = {
    "name":      "Park Fire (CA, 2024)",
    "bbox":      [-122.128, 39.771, -121.631, 40.385],
    "date":      "2024-08-04",
    "product":   "HLSL30",
    "mtbs_name": "PARK",
}
```

To run on a different fire, change `bbox`, `date`, `mtbs_name`, and (optionally) `product`. The search falls back from HLSS30 to HLSL30 (and vice versa) automatically.

## Model artifacts

The notebook downloads these once and caches them in `~/.cache/prithvi_burn/`:

- `Prithvi-EO-burnscars-config.yaml`
- `Prithvi-EO-burnscars-config.ckpt`

## Expected result (Park Fire 2024)

```
F1 = 0.929   IoU = 0.868   precision = 0.993   recall = 0.874   accuracy = 0.927
```
