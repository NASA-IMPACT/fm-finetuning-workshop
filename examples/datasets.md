# Datasets to try

Public Hugging Face datasets the studio has official fine-tuning examples for. Paste the
link in your first message, with the problem in your own words. The chat fetches a sample of
the dataset itself, so there is nothing to download or upload.

## Earth observation (Prithvi-EO)

### BioMassters: forest biomass

https://huggingface.co/datasets/ibm-nasa-geospatial/BioMassters

Sentinel-1 radar and Sentinel-2 images of Finnish forests, with 12 months of imagery for each
of 11,462 above-ground biomass maps (256 x 256 pixels). The model estimates biomass for each
pixel.

```
I want to estimate forest above-ground biomass from satellite image time series, for a forestry agency. Data: https://huggingface.co/datasets/ibm-nasa-geospatial/BioMassters
```

### HLS Burn Scars: burned areas

https://huggingface.co/datasets/ibm-nasa-geospatial/hls_burn_scars

Six-band HLS chips (512 x 512 pixels, 30 m) with a burned or not-burned mask for each. The
model marks which pixels burned. Step by step: [burn-scars.md](burn-scars.md).

```
I want to map burned areas after a wildfire from satellite images, for a forest service that plans recovery. Data: https://huggingface.co/datasets/ibm-nasa-geospatial/hls_burn_scars
```

## Weather and climate (Prithvi-WxC)

### Gravity wave parameterization

https://huggingface.co/datasets/ibm-nasa-geospatial/gravity-wave-parameterization

Input and output pairs computed from ERA5 on a 64 x 128 global grid: the background
atmospheric state goes in, and the gravity-wave momentum fluxes at each level come out. The
repository holds one month for testing. The model learns the fluxes a climate model would
otherwise approximate with a hand-written scheme.

```
I want to predict gravity-wave momentum fluxes from the atmospheric state, for a climate modelling team that wants a learned parameterization. Data: https://huggingface.co/datasets/ibm-nasa-geospatial/gravity-wave-parameterization
```

### WxC-Bench: six weather and climate tasks

https://huggingface.co/datasets/nasa-impact/WxC-Bench

A benchmark with a dataset for each of six tasks:
1. gravity-wave momentum flux;
2. aviation turbulence;
3. weather analogs;
4. natural-language weather forecasts;
5. long-term precipitation forecasting;
6. hurricane track and intensity.

Name the one task you want in your first message.

```
I want to <one of the six tasks>, for <who will use it>. Data: https://huggingface.co/datasets/nasa-impact/WxC-Bench
```

## Fusion: satellite images plus weather

### GPP flux: carbon uptake at flux towers

https://huggingface.co/datasets/ibm-nasa-geospatial/hls_merra2_gppFlux

975 samples from 37 flux towers worldwide, 2018 to 2021. Each sample has:
- an HLS chip (50 x 50 pixels, 6 bands) centred on the tower;
- 10 MERRA-2 weather values for that tower (temperature, soil moisture, heat flux,
  radiation, precipitation);
- the daily Gross Primary Productivity (GPP) measured at the tower.

The model combines the image and the weather to predict one GPP value per sample.

```
I want to predict daily carbon uptake (gross primary productivity) at flux towers from satellite images and weather data, for an ecosystem science team. Data: https://huggingface.co/datasets/ibm-nasa-geospatial/hls_merra2_gppFlux
```
