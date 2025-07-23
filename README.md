# Treeline-Mapper
Developed by Declan Knies as part of CAIC Collaboration to Map Treeline using LIDAR Point Cloud and NVDI Satellite Data.

## Introduction
This tool takes ESA Copernicus Satellite Imagery and LiDAR Point-Cloud data to predict the exent of Above, Near, and Below treeline elevation bands.

## Setup
To set up the tool, connect the downloaded arctoolbox to your current project through *Toolboxes, Add Toolbox, CAIC Project.atbx*.

## Usage
Open up the arctoolbox, you will be presented some options:

### NVDI Raster
This raster is acquired from the [Copernicus Web Browser](https://browser.dataspace.copernicus.eu).  
First, you must create a free account to access analytical imagery.  
With an account logged in, find a date in your desired area with 0% (or lowest possible) cloud cover, as this can distort the index.  
Select the NVDI layer on the left, then, find the image download icon on the right side of the screen.   
Select the Analytical download setting, then set the image format to TIFF. Keep WGS 84 projection, and download the NVDI layer.  

Select the downloaded NVDI layer as your NVDI raster for the toolbox.

Keep in mind, the NVDI changes a lot depending on time of year, the original values were calibrated for January. Using another time of year may provide inconsistency. For best results, study the NVDI in a small area to find out the optimal values.

### DEM (In Feet)
The DEM can be acquired however preferred. The [DEM-Clipper](https://github.com/cologeosurvey/DEM-Clipper/tree/main) can be accessed for this purpose.
  
Select your preffered DEM as your DEM (In Feet). Ensure projection is WGS 84 and elevation is in feet.

### Input Canopy Height (Optional)
Canopy height is created via raw LiDAR data. Use preferred method. The process should include subtracting the DEM from the first detection. This will give the impression of canopy height at any elevation. While this can help prevent small patches of meadows or skinny slide paths from showing up as Near treeline, it is not strictly necessary and requires a large amount of processing power to create. Here is an example workflow for ArcGIS:
  
1. Convert LAZ dataset to LAS dataset for data manipulation (Convert LAS tool)
2. Convert LAS dataset to first detection raster. (LAS Dataset to Raster tool)
3. Subtract DEM from first detection raster to get canopy height (Minus tool)
4. Convert raster to TIFF format (Raster to Other Format tool)
  
Use this product or one through another method as your canopy height TIFF.

### Output Geodatabase
The products will be created where specified, so choose a desired Geodatabase.

## Tweaking values
In order to calibrate the model for specific areas, it is useful to note average elevations, maximum and minimum extents, and notable edge cases. More work needs to be done to develop a workflow for calibration. In this section, various dials will be mentioned in order to provide flexibility in mapping. While hard-coded, these values could easily be made variables if deemed important.

### Rolling Average
The first step of the process involves applying a 15x15 foot rolling average. The goal of this is to smooth out the canopy height and remove any small, unimportant outliers. Additionally, this can help large below treeline clearings (slide paths, meadows), be more obviously visible. Applying a larger or smaller filter will scale the resolution of the mapping. 15x15 was found to be ideal for various parts of the front range (Dry Gultch, Rollins Pass).

### Classification Variables

#### Below Treeline
Below treeline is defined by having an NDVI < 0.5, an Elevation < 11000 ft, or a Canopy > 70 ft.

#### Near Treeline
Near treeline is defined by having an NVDI in the range of 0.5 ≤ x < 0.9 and an Elevation in the range of 11000 ft ≤ x < 12500 ft.

#### Above Treeline
Above treeline is defined by having an NDVI ≥ 0.9 and an Elevation ≥ 11000 ft or anywhere with an Elevation ≥ 12500 ft.

#### Basics of Tweaking

The primary values you may want to change include the hard-coded Below and Above Treeline boundaries. Default 11,000 ft and 12,500 ft. Different areas of the state will require changing these values. More work needs to be done to create a uniform process to choose these values.

#### Understanding These Classifications
Basically, anywhere below 11,000 feet, with an average canopy above 70 feet, or with a vegetation index of above .5 is considered below treeline. In our test case of Dry Gultch, anywhere with any of these characteristics were evidently below treeline. In lower parts of the state (Marble, Steamboat, Cameron Pass), adjusting elevation may be necessary. In higher parts (Sawatch, San Juans, Elks), fine tuning may be necessary as well.
  
Near treeline is where nuance begins to creep in. Even between forecasters, extent of near treeline is often blurry and unclear. In Dry Gultch, the elevation range of 11,000 to 12,500 feet is where a majority of cases exist. Within the range of uncertainty, NVDI is used to inform the results. Generally, .5 to .9 NVDI values fit dense bushes, sparse trees, and tall grasses. Where dense forest exists, NVDI values tend to stay below .5, whereas alpine tundra tends to stay above .9. This may need to be tweaked based on available vegetation, but should stay similar within large areas of Colorado. One way to visualize the uncertainity in this mapping for end users could be a wider extent that overlaps with Below or Above Treeline. 

Above 12,500 feet, above treeline conditions prevail. Within 11,000 to 12,500 feet, a NVDI value of above .9 shows a gross lack of vegeation; the prevelance of alpine tundra. These conditions tend to represent Above treeline, however, large valleys with meadows may need manual tweaking.
