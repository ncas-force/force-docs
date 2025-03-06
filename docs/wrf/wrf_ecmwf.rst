Running WRF with ECMWF input data
######################################
RR Burton, March 2025

1. Summary
==========

This describes the steps needed to ingest ECMWF data into WPS.

* Download the relevant fields from Copernicus

* Create separate namelist files for upper-air and surface fields. The namelist files need changing so that metgrid needs to know which pre-processed data to read.

* Link the ECMWF Vtable
* Run ungrib twice, once for the upper-air and once for the surface fields
* Run metgrid
* Run WRF as normal - no further changes need to be made.

2. Data required from Copernicus
================================

The following fields are required from Copernicus, either through the 
user interface or via the API. This has to be done in two stages as the
upper-air and surface fields are accessed from different locations on the
copernicus server.

2.1. Upper-air fields
---------------------

From the Copernicus server go to 

https://cds.climate.copernicus.eu/datasets/reanalysis-era5-pressure-levels?tab=download

The required fields are:

.. code-block::

	geopotential
	relative_humidity 
	specific_humidity 
	temperature 
	u_component_of_wind
	v_component_of_wind

.. note::

	Select "Reanalysis" 
.. note::

	Select **all** pressure levels for this. Select the times required
	for the run.

.. note::
	The Copernicus page will show you the API request for the values you have selected
	(bottom of the web page).

.. note::
	You can choose the geographical area - I haven't tried this. No reason why it won't work though.

.. note:: 
         
	Request the data as a grib file

2.2. Surface fields
-------------------

From the copernicus server go to

https://cds.climate.copernicus.eu/datasets/reanalysis-era5-single-levels?tab=download

The required fields are:

.. code-block::

	10m_u_component_of_wind
	10m_v_component_of_wind
	2m_dewpoint_temperature
	2m_temperature
	land_sea_mask
	mean_sea_level_pressure
	sea_ice_cover
	sea_surface_temperature
	skin_temperature
	snow_depth
	soil_temperature_level_1
	soil_temperature_level_2
	soil_temperature_level_3
	soil_temperature_level_4
	surface_pressure
	volumetric_soil_water_layer_1
	volumetric_soil_water_layer_2
	volumetric_soil_water_layer_3
	volumetric_soil_water_layer_4

.. note::

	Select the times required for the run.

.. note::
	The Copernicus page will show you the API request for the values you have selected
	(bottom om the web page).

**Request the data as a grib file.**

Once the two grib files have been downloaded (either from the user interface
or via API) copy them to the WPS directory.

3. Create two namelist files
============================

We need separate namelist files for the upper-air and surface fields.

.. code-block::

	cp namelist.wps namelist.wps.upper
	cp namelist.wps namelist.wps.surface

Edit the ungrib section of ``namelist.wps.surface``

.. code-block::

	&ungrib
 	out_format = 'WPS',
 	prefix = 'SFILE',
	/

Also, change the ``metgrid`` section of the namelist:

.. code-block::

	&metgrid
 	fg_name = 'UFILE','SFILE'

Edit the ungrib section of ``namelist.wps.upper``

.. code-block::

	&ungrib
 	out_format = 'WPS',
 	prefix = 'UFILE',
	/

Also, change the ``metgrid`` section of the namelist:

.. code-block::

	&metgrid
 	fg_name = 'UFILE','SFILE'

4. Link the ECMWF Vtable
========================

Simply do

.. code-block::

	ln -sf ./ungrib/Variable_Tables/Vtable.ECMWF ./Vtable

5. Run ungrib

We have to run ungrib twice, once for each of the sets of grib files.

.. code-block::

	./link_grib.sh ./your_surface_grib_files

where ``your_surface_grib_files`` are the grib file(s) from Copernicus.
(This is exactly as we would do to link GFS files).

.. code-block::

	cp namelist.wps.surface namelist.wps
	./ungrib.exe

.. note::

	If successful, you should have ``SFILE`` present in this directory.

Now do the same for upper-air:
 
.. code-block::

	./link_grib.sh ./your_upper-air_grib_files

where ``your_upper-air_grib_files`` are the grib file(s) from Copernicus.
(This is exactly as we would do to link GFS files).

.. note::
	When I have done this I have deleted/unlinked the previous set of GRIB files - not sure if this is necessary. But check that the GRIB files are now linked to the upper-air files.


.. code-block::

	cp namelist.wps.upper namelist.wps
	./ungrib.exe

.. note::

	If successful, you should have ``UFILE`` present in this directory.

Now, you can run metgrid and from the changes to the namelist files
WPS knows to read in both the ``SFILE`` and the ``UFILE``.

.. code-block::
	
	./metgrid.exe

.. note::
	check for the usual signs of successful completion.

5. Running WRF
==============

No further changes need be made - go to the WRF directory, 
link the ``met_em`` files you have just created, etc.
