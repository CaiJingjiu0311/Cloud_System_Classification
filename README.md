# Cloud System Classification README:
## Convective/Stratiform masking and Storm Mode Classification modules
[![DOI](https://zenodo.org/badge/469831792.svg)](https://zenodo.org/badge/latestdoi/469831792)

The Cloud System Classification modules include 
1. the **Convective/Stratiform Masking** module and (`./Conv_Stra_Mask`)
2. the **Storm Mode Classification** module (`./Storm_Mode_Class`).

------

### 1. Convective/Stratiform Masking:

`conv_stra_mask.py`

* The module indentifies (masks) the Convective and Stratiform echos at a 2D level in the 3D gridded radar relectivity from observations or model outputs.
* The methodology is based on [Steiner, Houze, and Yuter 1995 (SHY95)](https://journals.ametsoc.org/view/journals/apme/34/9/1520-0450_1995_034_1978_ccotds_2_0_co_2.xml). The similar methodology from [Yuter and Houze 1998 (YH98)](https://atmos.uw.edu/MG/PDFs/QJR98_yute_natural.pdf) can also be applied by choice, which allows more tuning on the parameters.
* The module can handle either the Cartesian-gridded data (Lat-Lon coordinates) or the Lambert-conformal-gridded data (constant distance grid), depends on the data coordinates and/or usage. The Cartesian coordinate is common for observational data, and the Lambert-conformal coordinate is common among models output (e.g. WRF CONUS runs).

#### How to run?
```
CS_mask, Conv_Cent, Bkgnd_REFL = conv_stra_sep( dbz_data, lat, lon, coor_type, grid_res )
```

#### Input:
  *  **dbz_data** is the specified level of interpolated reflectivity, for example at 2-km height for observational data, or the 12th &sigma;-level for model output. 
  * **lat** and **lon** are the corresponding latitude and longitude coordinates.
  * **coor_type**: `C` or `L`; Cartesian or Lambert-conformal coordinates type.
  * **grid_res**: grid resolution in degree or km, depends on the coordinates type.

#### Output:
  * **CS_mask** is the derived convective/stratiform mask with the same size as **dbz_data**.
    * -1: None of the identified categories (un-masked grid points)
    * 0: stratiform (where data is good & reflectivity >= 15 dBZ)
    * 1: convective (background reflectivity <= 25 dBZ)
    * 2: convective (25 < background reflectivity <= 30 dBZ)
    * 3: convective (30 < background reflectivity <= 35 dBZ)
    * 4: convective (35 < background reflectivity <= 40 dBZ)
    * 5: convective (background reflectivity > 40 dBZ)

#### Processing Steps:
![](https://github.com/yuhungjui/Cloud_System_Classification/blob/main/Conv_Stra_Mask/Conv_Stra_Mask_steps.png)

*:heavy_exclamation_mark: Please see the codes header for more details on output **Conv_Cent** and **Bkgnd_REFL**.*

------

### 2. Storm Mode Classification:

`storm_mode_class5.py`

* The module classifies the storm modes (or cloud types) from 3D radar relectivity (dBZ) gridded observations or (WRF) model outputs. 
* The classification is based on the methodology in [Houze et al. 2015](https://agupubs.onlinelibrary.wiley.com/doi/10.1002/2015RG000488) which identifies four types of clouds from the TRMM satellite reflectivity, including **Deep convective cores (DCCs)**, **Wide convective cores (WCCs)**, **Deep and Wide convective cores (DWCCs)**, and **Broad stratiform regions (BSRs)**. Each category are identified according to two thresholds: **moderate** or **strong**.
* This module specifically identifies the 5th category: the **Ordinay Convective Cores (OCCs)** to represents those convective cores that are neither deep or wide enough according to the thresholds (likely convection during intensifying or decaying stages).

#### How to run?
```
DCC_mask, OCC_mask, WCC_mask, DWCC_mask, BSR_mask = storm_mode_class5.storm_mode_c5( 3d_refl, reflc, CS_mask, geo_H, grid_res, thresholds_type)
```
```
Storm_Mode = storm_mode_class5.merge_to_Storm_Mode(DCC_mask, OCC_mask, WCC_mask, DWCC_mask, BSR_mask)
```

#### Input:
  * **3d_refl** is the 3D gridded reflectivity, in which the the 2D maximum composite **reflc** is calculated.
  * **CS_mask**: the convective/stratiform masks as identified by the Convective/Stratiform Masking module.
  * **geo_H**: the 3D gridded geopotential heights which serves as the height coordinate.
  * **grid_res**: the horizontal resolution of the input reflectivity [km].
  * **thresholds_type**: `moderate` or `strong`.

#### Output:
  * Each **_masks** are indicated in 1 as masked, and 0 as none.
  * **Storm_Mode** is merged from every masks and indicated as 
    * 0: others
    * 1: DCC
    * 2: OCC
    * 3: WCC
    * 4: DWCC
    * 5: BSR

#### Flow Chart:
![](https://github.com/yuhungjui/Cloud_System_Classification/blob/main/Storm_Mode_Class/Storm_Mode_Class_flow.png)

------

### Specific Dependencies:

* [wrf-python](https://wrf-python.readthedocs.io/en/latest/index.html)
* Numpy
* Scipy

### References:

* Houze, R. A., Rasmussen, K. L., Zuluaga, M. D., and Brodzik, S. R. (2015), The variable nature of convection in the tropics and subtropics: A legacy of 16 years of the Tropical Rainfall Measuring Mission satellite, Rev. Geophys., 53, 994– 1021. ([link](https://agupubs.onlinelibrary.wiley.com/action/showCitFormats?doi=10.1002%2F2015RG000488))
* Steiner, M., R. A. Houze, Jr., and S. E. Yuter, 1995: Climatological characterization of three-dimensional storm structure from operational radar and rain gauge data. J. Appl. Meteor., 34, 1978-2007. ([link](https://atmos.uw.edu/MG/PDFs/JAM95_stei_climatological.pdf))
* Yuter, S. E., and R. A. Houze, Jr., 1997: Measurements of raindrop size distributions over the Pacific warm pool and implications for Z-R relations. J. Appl. Meteor., 36, 847-867. ([link](https://atmos.uw.edu/MG/PDFs/JAM97_yute_measurements.pdf))
* Yuter, S. E., and R. A. Houze, Jr., 1998: The natural variability of precipitating clouds over the western Pacific warm pool. Quart. J. Roy. Meteor. Soc., 124, 53-99. ([link](https://atmos.uw.edu/MG/PDFs/QJR98_yute_natural.pdf))

### Acknowledgement:

* Department of Atmospheric Science at Colorado State University: 
  * Nick Guy, Brody Fuchs for initialization of the Conv/Stra mask procedure.
  * Zach Bruick for structure of the algorithms.
  * Prof. Kristen L. Rasmussen for advising, and Rung Panasawatwong and Marqi Rocque for testing and analyses.

### Cite this module:
[![DOI](https://zenodo.org/badge/469831792.svg)](https://zenodo.org/badge/latestdoi/469831792)

------

Last update - 20220314 - Hungjui Yu
