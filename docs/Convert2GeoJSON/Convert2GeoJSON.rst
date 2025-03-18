Convert2GeoJSON
===============
Introduction
------------
This document will outline the way in which the convert to geojson code should be used with a variety of different data sources. This will mainly be achieved by explaining the input options/arguments that can be used and providing examples. It should be noted that this code is still under development, due to this there are caveats about how the code can and should be used. I have detailed some areas for improvement that are planned and will hopefully be addressed int he near future.

Environment
-----------
The environment that I am currently using for the development of the Convert2GeoJSON python code is described in the yaml file linked `here <https://github.com/earajr/convert2GeoJSON/blob/main/convert2geojson.yml>`_.

Key dependencies are:
    - **numpy**
    - **argparse**
    - **shapely**
    - **matplotlib**
    - **scipy**
    - **netCDF4**
    - **wrf-python**
    - **skimage**
    - **pyart**
    - **satpy**
    - **yaml**
    - **cf-python**
    - **iris**
    - **json**

Arguments
---------
The arguments to the convert2GeoJSON.py script are very important in defining how the code behaves. Below are detailed the possible arguments and descriptions of how they should be used.

|

.. list-table:: 
   :widths: 60 250
   :header-rows: 1

   * - Argument
     - Description
   * - **--input_file**
     - | The path of the input file that is being processed.
       |
   * - **--output_dir**
     - | The path of the output directory. This is the location that the resultant GeoJSONs will be stored. Output files will be named based on the input filename via an automated process.
       |
   * - **--var_name**
     - | The variable to be read from the input file. Depending on the type of file being converted to GeoJSON this might be a variable name from the netcdf or a calculated variable that is generated from other variables. This is specific to the file/product and what python libraries are used in the reader for that dataset (e.g. wrf-python).
       |
   * - **--source**
     - | The source of the data being converted to GeoJSON. For each possible source of data a reader function must be created. The reader functions allow for the sometimes complicated input files to be read in whatever python method that best suits them (e.g. wrf-python, pyart, satpy, cf-python) and a 2-dimensional array of values (and lat, lon values) to be passed to the rest of the conversion code.
       |
       | As the readers are separate and modular there is potential for the Convert2GeoJSON code to have very broad applications in which a reader can be produced for any new input and then the resultant data be passed on to the rest of the functions for further processing.
       |
       | The current readers that are available are:
       | **WRF2d** - single layer WRF variables such as 2m temperature, wrf-python getvar variables or variables included in netcdf files can all be read.
       | **WRF3dp** - three dimensional WRF variables interpolated onto a pressure level, wrf-python getvar variables or variables included in netcdf files can all be read.
       | **WRF3dh** - three dimensional WRF variables interpolated onto a height level (above sea-level), wrf-python getvar variables or variables included in netcdf files can all be read.
       | **HYSPLIT** - HYSPLIT netcdf files split into all levels contained in the file and all levels converted to GeoJSON format, variables included in netcdf files can be read.
       | **CRR** - NWCSAF convective rainfall rate - used within the FASTA workflow, variables included in netcdf files can be read (both retrievals and extrapolations).
       | **NCASradar** - horizontal data from NCAS mobile radars where data has been written to cfradial format, variables included in netcdf files can be read.
       | **MTG_LI_ACC** - data from Meteosat Third Generation Lightning Imager accumulated/gridded products. Possible variables are “accumulated_flash_area”, “flash_radiance” and “flash_accumulation”.
       | **UM3dh** - three dimensional data from Met Office Unified model interpolated onto a height level (above sea-level). This is currently not a complete reader. It sometimes relies heavily on extra information from separate files. However this needs to be modified so that if required information is not present then the code should fail verbosely highlighting the required information that is missing.
       |
       | In addition to these readers I have several additional readers planned. These are:
       | **UM2d** - single level Met Office Unified Model data.
       | **UM3dp** - three dimensional Met Office Unified model data interpolated onto pressure levels (will face similar issues as described above for UM3dh).
       | **RoA** - Rain over Africa reader with the aim of being able to process RoA retrievals and pysteps generated extrapolations to replace existing CRR data in FASTA.
       |
   * - **--level**
     - | The level identifier for 3d data fields, e.g. for WRF3dp **800** will return an array of values valid for 800 hPa. If a 2d reader or a reader that loops over all levels within the files are being used (e.g WRF2d or HYSPLIT) the level argument is ignored and the behaviour of the reader is unchanged.
       |
   * - **--contour_start**
     - | Numeric giving the start of the contour range.
       |
   * - **--contour_stop**
     - | Numeric giving the end of the contour range
       |
   * - **--interval**
     - | Numeric giving the step between the contour start and stop points.
       |
       | **NB.** the arguments **contour_start**, **contour_stop** and **interval** can be used to provide specific contour levels. This can also be achieved by specifying the contour thresholds directly using the **contour_thresholds** argument. If no specification is made then the convert2GeoJSON code will read the requested variable and produce “sensible” contour levels based on the min and max values in the array. This is obviously not ideal if the range of interest does not match the min max range or if there is a requirement for consistency across multiple files to be converted.
       |
   * - **--contour_thresholds**
     - | A list of numeric values to explicitly set the contour levels. This can be used in place of **contour_start**, **contour_stop** and **interval**.This can be a useful argument to make use of if the contours required are not evenly spaced e.g.
       | ``--contour_thresholds 0.2 1.0 2.0 3.0 5.0 7.0 10.0 15.0 20.0 30.0 50.0 200.0``
       |
   * - **--contour_names**
     - | A list of strings to explicitly set the contour names that will be used in the GeoJSON file. Similar in use to the **contour_thresholds** argument. The number of contour names must be 1 less than the number of contour thresholds as each contour name is assigned to the filled region between thresholds values. If no contour names are set they are generated based on the contour levels being produced (either based on **contour_thresholds** argument or **contour_start**, **contour_stop** and **interval arguments**).
       |
   * - **--smooth**
     - | A flag to indicate whether the data array passed to the contouring code should be smoothed. If the smooth argument is not used then data will not be smoothed, but if the argument is invoked (just by including --smooth as an argument) then a gaussian smoothing will be applied. This will make use of either a user denied sigma value or a suggested sigma value (based on the grid spacing of the data being supplied).
       |
   * - **--sigma**
     - | A user defined sigma value to be used in the smoothing of data prior to contours being calculated.
       |
   * - **--colormap**
     - | The name of matplotlib colormap to generate colours for contours. A list of hex values (from the provided colormap) for each filled contour region is generated and provided within the GeoJSON file. This can then be used as the default colour fill for the contours when files are used for online mapping or can be ignored and a different range of colours used. If no **colormap** or **colors** (see below) are specified then the default matplotlib colormap selected is viridis.
       |
   * - **--colors**
     - | A list of strings to define hex code colors explicitly. This can only be used if contours have been explicitly set and must have 1 fewer values than the contour_thresholds argument. Also make sure that you remove the leading # from hex codes e.g. 
       | 
       | ``--colors 1e50d2 0294fe 00d2fc 448622 02c403 6dec02 fefe02 fdc102 ff5000 b31b26 7f7f7f``
       |
   * - **--contour_method**
     - | A choice of contouring methods, this can be left unset and the **standard** method will be employed (skimage.measure.find_contours) or set to **pixel** where a bespoke contouring method will be called. The pixel option draws square edged contours around the pixel edges and can position values next to each other physically that are multiple contours away. This means that in regions where the values in the displayed array change very rapidly compared to the horizontal spacing of the data no gaps are generated. At the moment this approach is quite slow (compared to the previously employed method) and so should be used with caution. However, it means that data can be displayed with no smoothing and without any gaps or artifacts which is not true for the previous method. Currently the pixel method is too slow to be applied to large arrays with many contour line segments (such as CRR). However for WRF simulations which have a relatively small number of grid points this approach is possible.


Examples
--------
These examples are for processing WRF NWR data (I intend to expand this section in future to make a more comprehensive set of examples).

|

**900 hPa relative humidity**

``python convert2GeoJSON.py --input_file /home/force-nwr/nwr/uk/data/2025031800/wrfout_d01_2025-03-18_06:00:00 --output_dir . --var_name rh --source WRF3dp --contour_thresholds 0 5 10 15 20 25 30 35 40 45 50 55 60 65 70 75 80 85 90 95 100 105 --level 900 --contour_method pixel``

|

**5000 m temperature**

``python convert2GeoJSON.py --input_file /home/force-nwr/nwr/uk/data/2025031800/wrfout_d01_2025-03-18_11:00:00 --output_dir . --var_name tc --source WRF3dh --contour_thresholds -20 -18 -16 -14 -12 -10 -8 -6 -4 -2 0 2 4 6 8 10 12 14 16 18 20 22 24 26 28 30 32 34 36 38 40 50 --level 5000 --contour_method pixel``

|

**Maximum reflectivity (a 2d variable)**

``python convert2GeoJSON.py --input_file /home/force-nwr/nwr/uk/data/2025031800/wrfout_d01_2025-03-18_19:00:00 --output_dir . --var_name mdbz --source WRF2d --contour_thresholds 0.0 10.0 20.0 30.0 40.0 50.0 60.0 200.0 --contour_method pixel``

Processing WRF
--------------
I have also produced a bash script to generate the commands to process WRF data. This only needs the directory where the wrfout files are stored as an argument. From this it creates a text file list of python jobs that can be executed in parallel. This script can be found here (`create_json_WRF.sh) <https://github.com/earajr/convert2GeoJSON/blob/main/create_json_WRF.sh>`_.
