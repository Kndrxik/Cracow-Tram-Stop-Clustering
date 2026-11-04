# Kraków Tram Stop Clustering

**Exploratory spatial data analysis and unsupervised machine learning using Kraków's public transport GTFS data.**

This project explores how tram stops in Kraków differ in terms of the number of routes serving them, the density of nearby stops, and their distance from the city centre. The workflow combines **Pandas**, **GeoPandas**, and **scikit-learn** to engineer spatial features and group stops with **K-Means**.

## Project goals

- Explore the distribution of tram routes across stops.
- Calculate spatial characteristics of each stop.
- Compare K-Means clustering solutions with different numbers of clusters.
- Visualize the clusters geographically and discuss their meaning and limitations.

## Data

**Source:** [Zarząd Transportu Publicznego w Krakowie — GTFS](https://gtfs.ztp.krakow.pl/)

The project uses the tram feed (`GTFS_KRK_T.zip`). The main input tables are:

| File | Purpose |
| --- | --- |
| `stops.txt` | Stop identifiers, names and coordinates |
| `stop_times.txt` | Stops visited by individual trips |
| `trips.txt` | Association between trips and routes |

The tables are joined through `stop_id` and `trip_id`. The number of **unique** `route_id` values is calculated for each stop, so repeated trips on the same route are not counted as additional lines.

> GTFS data can change between downloads. Results shown in the notebooks refer to the particular feed used during the analysis.

## Workflow

```mermaid
flowchart LR
    A["GTFS tables"] --> B["Data loading & joins"]
    B --> C["Exploratory analysis"]
    C --> D["Spatial feature engineering"]
    D --> E["StandardScaler"]
    E --> F["K-Means clustering"]
    F --> G["Evaluation & maps"]
```

### 1. Data loading — `01_loading.ipynb`

- Read and combine GTFS data with Pandas.
- Create point geometries from longitude and latitude.
- Count the distinct tram routes serving each stop.
- Export the prepared data as `stops_with_routes.gpkg`.

### 2. Exploratory data analysis — `02_EDA.ipynb`

- Examine the distribution of `number_of_routes`.
- Visualize the data using a histogram, boxplot and point map.
- Check descriptive statistics and identify stops with many routes.

### 3. Spatial analysis — `03_spatial.ipynb`

Create two new features using GeoPandas and Shapely:

| Feature | Description |
| --- | --- |
| `nearby_stops_500m` | Number of *other* stops within 500 metres |
| `distance_to_center` | Straight-line distance to an approximate point in Kraków's city centre, in metres |

The analysis uses **EPSG:2180** for distance-based operations. It also explores correlations between the three features and exports `stops_spatial.gpkg`.

### 4. Machine learning — `04_clustering.ipynb`

The model uses the following inputs:

```python
features = [
    "number_of_routes",
    "nearby_stops_500m",
    "distance_to_center",
]
```

- Remove observations with missing values in the selected features.
- Standardize the features with `StandardScaler`.
- Evaluate K-Means for `k = 2, ..., 10` using **inertia (Elbow Method)** and **Silhouette Score**.
- Compare spatial patterns for `k = 4` and `k = 10`.
- Assign cluster labels back to the corresponding GeoDataFrame rows.

## Results

The modeling dataset contains **353 stops** with complete values for the three selected features.

| Number of clusters | Approx. silhouette score | Interpretation |
| --- | ---: | --- |
| `k = 4` | `0.295` | Simpler, more interpretable segmentation |
| `k = 10` | `0.339` | More detailed segmentation; highest score among the tested `k` values |

*Scores are approximate values read from the project's comparison plot.*

The Elbow plot suggests diminishing improvements in inertia around `k = 4`, while the Silhouette Score is higher for `k = 10`. Neither metric alone establishes the best geographic interpretation. The **four-cluster model is used as an interpretable baseline**, and the ten-cluster model serves as a comparison.

The cluster maps show spatial structure, but the groups need to be interpreted using the mean and median values of the input features—not just their colors or locations.

## Repository structure

```text
klastry_przystankow/
├── data/
│   ├── GTFS_KRK_T/             # Downloaded and extracted GTFS files (local)
│   └── processed/              # Generated GeoPackage files
├── notebooks/
│   ├── 01_loading.ipynb
│   ├── 02_EDA.ipynb
│   ├── 03_spatial.ipynb
│   └── 04_clustering.ipynb
├── requirements.txt
└── README.md
```

Some data files may be excluded from version control. The source GTFS feed can be downloaded separately.

## Getting started

### 1. Clone the repository

```bash
git clone https://github.com/Kndrxik/klastry_przystankow.git
cd klastry_przystankow
```

### 2. Create an environment and install dependencies

```bash
python -m venv .venv
```

Activate it:

**Windows PowerShell**
```powershell
.\.venv\Scripts\Activate.ps1
```

**macOS / Linux**
```bash
source .venv/bin/activate
```

Install the project dependencies:

```bash
pip install -r requirements.txt
```

### 3. Download the GTFS data

Download `GTFS_KRK_T.zip` from [the ZTP Kraków GTFS website](https://gtfs.ztp.krakow.pl/) and extract the text files into:

```text
data/GTFS_KRK_T/
├── stops.txt
├── stop_times.txt
└── trips.txt
```

Create the output folder if necessary:

```text
data/processed/
```

### 4. Run the notebooks

Open the repository in VS Code or Jupyter and run the notebooks in order:

1. `01_loading.ipynb`
2. `02_EDA.ipynb`
3. `03_spatial.ipynb`
4. `04_clustering.ipynb`

The notebook paths assume that the working directory is `notebooks/`. Run the cells sequentially, as later notebooks depend on GeoPackage files generated earlier.

## Limitations

- A GTFS `stop_id` may identify an individual platform or boarding position, not an entire interchange. Nearby stops may therefore represent different platforms of the same stop complex.
- Route count is **not** the same as service frequency, ridership or passenger demand.
- The 500 m neighborhood is based on straight-line distance rather than pedestrian network distance.
- Distance to the centre is a simplified geometric feature, not travel time.
- Because distance to the centre is a model input, some spatial separation of clusters is expected.
- Silhouette values of approximately `0.30–0.34` indicate that the resulting clusters are not strongly separated.

## Tech stack

**Python · Pandas · GeoPandas · Shapely · scikit-learn · Matplotlib · Jupyter Notebook**

---

**Author:** [Kndrxik](https://github.com/Kndrxik)  
**Data attribution:** Zarząd Transportu Publicznego w Krakowie (ZTP Kraków)
