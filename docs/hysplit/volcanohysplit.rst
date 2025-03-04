Running HYSPLIT for volcanic eruptions
######################################
RR Burton, March 2025

1. Summary
==========

There are three stages to running HYSPLIT for volcanic ash

1. Get the HYSPLIT code

2. Install the BGS software to produce the plots

3. Download HYSPLIT driving data.

4. Calculate emission rates.

5. Edit the configure script and enter data regarding the emission rates,
etc. If from a VONA then extract the relevant data, see below

6. Run HYSPLIT, convert results to netcdf, and change metadata and
scaling of concentrations for a combined (i.e. all the ash fields summed
into a single field) ash concentration

7. Run the ash-model plotting code to get the total column ash plots.

8. If a real eruption, prepare to be contacted by the UKMO dispersion
team.

2. Installation
===============

Requirements: (i) HYSPLIT code, (ii) BGS plotting code, (iii) simple
code to provide emission rates (see below)

The simplest is to copy the “clean” HYSPLIT code on mummra. This is
located in::

	/home/lecrrb/hysplit_master_test/hysplit_tracer_test/testing_example/

You will also need the ash-model-plotting software (N.B. I haven’t
tested this on i``mummra`` - I do this from my home space on my desktop)

https://github.com/BritishGeologicalSurvey/ash-model-plotting

The simple emission code is ``source.f`` and is located in the same
directory as the HYSPLIT code mentioned above.

3. Download driving data for HYSPLIT.
=====================================

This can be obtained from

`https://ftp.ncep.noaa.gov/data/nccf/com/hysplit/prod/ <https://ftp.ncep.noaa.gov/data/nccf/com/hysplit/prod/>`__

a sample bash script could be

.. code-block::

	cyear=`date --date='yesterday' -u '+%Y'\`
	cmonth=`date --date='yesterday' -u '+%m'\`
	cday=`date --date='yesterday' -u '+%d'\`
	wget --no-use-server-timestamps https://ftp.ncep.noaa.gov/data/nccf/com/hysplit/prod/hysplit.${cyear}{cmonth}{cday}/hysplit.t18z.gfsf

this will download yesterday’s 18Z driving data.

The timing of the eruption start will determine which driving data to
use. 

.. note ::

	There is usually about 4 hours latency in availability between the
	HYSPLIT cycles (6Z, 12Z, 18Z, 00Z). For example the 00Z cycle arrives
	online ~ 04Z. So if you wanted to start an eruption at 14Z then you
	would probably have to use the 06Z cycle.

4. Calculate emission rates.
============================

To run HYSPLIT for volcanic ash you need to know the height of the
plume. The height of the plume comes from either the VONA or by
guessing.

4.1 VONA Volcanic Observatory Notice for Aviation
-------------------------------------------------

To do the HYSPLIT simulation you need to have the following information
from the VONA:

**Volcano Location**: lat/lon, degrees (VONA usually give a name and a
volcano catalogue number: search the `Smithsonian Volcano
Database <https://volcano.si.edu/>`__ for the exact location of the
volcano)

**Plume height** (assume this number is in km, above sea level). **The
VONA will state if ash is being released - if it is just gas, then there
is less urgency.** We haven’t provided gas dispersion plots in the last
few eruptions on the peninsula.

VONAs are sent to ``volcano@ncas.ac.uk``

Example of VONA (this from an exercise) - my notes in red. **This was an
exercise**. The words “Exercise” will feature heavily in the VONA and
email subject if it is an exercise.

.. code-block::

	(1) VOLCANO OBSERVATORY NOTICE FOR AVIATION — VONA
	(2) Issued:
	20250211/1504Z
	(3) Volcano:
	Langjökull (371080)

.. note:: 

	look this up in the Smithsonian volcano database to get the latitude and longitude
	of the volcano

.. code-block::

	(4) Current aviation colour code:
	RED
	(5) Previous aviation colour code:
	red

.. note:: 
	This is the worst case - means an eruption is happening.

.. code-block::

	(6) Source:
	Icelandic Meteorological Office
	(7) Notice number:
	2025-598
	(8) Volcano location:
	N6451 W01947

.. note:: 
	These are some Icelandic ref grid locations - use the location from the
	catalogue (see above).

.. code-block::

	(9) Area:
	Western Volcanic Zone
	(10) Summit elevation:
	1067 M
	(11) Volcanic activity summary:
	The radar network has detected a volcanic plum over Langjökull from
	15:40 UTC. The plume height is assessed to be about 8 km asl. Ash and
	volcanic gases are expected to be produced and transported to the west.
	Seismic tremor continues to increase locally, and Langjökull volcano
	continues to erupt.

.. note:: 

	This contains the time of the eruption, 15:40. This will be used in the 
	HYSPLIT simulations.

.. code-block::

	(12) Volcanic cloud height:
	Ash plume height is 8 km asl.

.. note::

	This is the plume height you’ll need to use in ``source.f`` (see below).
	Assume this is height about sea level.

.. code-block::

	(13) Other volcanic cloud information:
	Plume height is 8 km asl and expected to be transported to the west.

4.2. Guessing
-------------

If no information on the height of the plume is available, then you have
to make an educated guess. Or run a set of scenarios - see below.

4.3 Get the emission rates from the plume height
------------------------------------------------

We use the Sparks-Mastin type formula
:math:`Q\  = \ AB{H^{4.5}}^{}`\ where *H* is the plume height, *B* is an
empirically-derived constant (55) and *A* is set to 0.03, which allows
most of the ash to have been removed from the plume apart from very
close to the source.

Edit ``source.f`` and change the first line of the program to the plume
height specified in the VONA

e.g. ``rdepth_base=8.0`` for plume height = 8 km in the example above.

compile and run source.f to produce the emission rates for the particle
size distributions. In the HYSPLIT directory above the plume height is
set at 5 km. This will give something like

.. code-block::

	66411.2211
	556193.964
	2133460.37
	5545336.56
	xxxxxxxxxxxxxxxxxxxxxx
	101978.483
	854069.779
	3276058.62
	8515202.74
	xxxxxxxxxxxxxxxxxxxxxx
	41336.4102
	346192.428
	1327932.11
	3451590.

**The first four are the ones you will need to use**, these give the
emission rates corresponding to the given plume height (in this example
``r_depth=5``) for the mass distribution in the plume.

The mass distribution is 0.008 for ash species 1, 0.067 for ash species
2, 0.257 for ash species 3 and 0.668 for ash species 4. The four ash
species have different diameters - full details can be found in the
HYSPLIT configuration file (see below) but **you don’t need to know or
change these**.

Note that the Sparks-Mastin equation shown here gives emission rates in
kg/hr, we need kg/s for HYSPLIT, so source.f makes this conversion.

If you want/have time to do “scenarios”, the second set of numbers gives
estimated emission rates +10% and the third set of numbers gives
estimated emission rates - 10%.

5. Edit the HYSPLIT configuration file
======================================

N.B. the HYSPLIT user guide can be found at

https://www.arl.noaa.gov/documents/reports/hysplit_user_guide.pdf

-although this is mainly aimed at users of the HYSPLIT GUI, it contains
all the information about the contents of the configuration and setup
files.

You will need

``CONTROL.VOLCICE_auto`` - the configuration file

and

``SETUP.VOLCICE_auto`` - the setup file

**To run a HYSPLIT example, you can use the ``CONTROL.VOLCICE_auto`` and
``SETUP.VOLCICE_auto`` exactly as they are in the HYSPLIT directory**. This
will work for a 28/2/25 case. However you will eventually need to run
for a different date, and the following shows what you need to do to
change this.

``SETUP.VOLCICE_auto`` should not need changing - it contains information on
the number of particles used, whether we use heights above sea level (we
do) and a few other things. Should be no need to change this file, but
**it needs to be present** in any run.

Change the ``CONTROL.VOLCICE_auto`` file at the following points: This is
the configure file, line by line: (comments in red text)

.. code-block::

	YYS MMS DDS 12 00 # CHANGE! start of eruption

.. note:: 

	Change to the start time of the eruption. YYS = year etc. Here 12 00 = 12:00 UTC eruption.

.. code-block::

	2

.. note:: 

	don’t change this! it tells HYSPLIT that there are ”two” sources - one
	at the surface, and one at 5000 m - HYSPLIT knows to interpret this as a
	vertical line source. The sources are in the following lines. 

.. code-block::

	S_LAT S_LON 0.0 # CHANGE! location

.. note:: 

	change to latitude and longitude of volcano. Always have 0.0 as the last
	item in this line 

.. code-block::

	S_LAT S_LON 5000 # CHANGE! location and height of plume AGL

.. note:: 

	change to latitude and longitude of volcano. Last item is plume height
	(in m), here 5 km. 

.. code-block::

	36 # CHANGE! hours to run for

.. note::

	 change if you like but see below for outputting options, otherwise leave 

.. code-block::

	0 # vertical motion option. 0=use met model field
	40000.0 # model top, m
	1 # number of gfs files to use, listed below, 2 lines per gfs file
	../driving_data_example/

.. note:: 

	you might want to change, but this must be where you store the HYSPLIT
	met driving data. Can be a full path, does not have to be a relative
	path, 

.. code-block::

	hysplit.tZZz.gfsf

.. note:: 

	change ZZ to the UTC hour of the HYSPLIT file, e.g. ``hysplit.t18z.gfsf`` 

.. code-block::

	4 # number of ash species, listed below, ash1 etc.

.. note:: 

	we have four ash species, that have different emission rates,
	effectively 

.. code-block::

	ash1
	66411.2211 # CHANGE! use values from source.f

.. note:: 

	the first number from the output from source.f above 

.. code-block::

	36 # CHANGE! hours to run for. same as above.

.. note:: 
	have this the same as the run time above.

.. code-block::

	YYS MMS DDS 12 00 # CHANGE! start time, same as above

.. note:: 

	must be the same as the eruption start time, above, same definitions for
	YYS etc. As before we have 12:00Z eruption start, 

.. code-block::

	ash2
	556193.964 # CHANGE! use values from source.f

.. note:: 

	the second number from the output from source.f above 

.. code-block::

	36 # CHANGE! hours to run for. same as above.

.. note:: 

	have this the same as the run time above. 

.. code-block::

	YYS MMS DDS 12 00 # CHANGE! start time, same as above

.. note:: 

	must be the same as the eruption start time, above, same definitions for
	YYS etc 

.. code-block::

	ash3
	2133460.37 # CHANGE! use values from source.f

.. note:: 

	the third number from the output from source.f above 

.. code-block::

	36 CHANGE! hours to run for. same as above.

.. note:: 

	have this the same as the run time above. 

.. code-block::

	YYS MMS DDS 12 00 # CHANGE! start time, same as above

.. note:: 

	must be the same as the eruption start time, above, same definitions for
	YYS etc 

.. code-block::

	ash4
	5545336.56 # CHANGE! use values from source.f

.. note:: 

	the fourth number from the output from source.f above 

.. code-block::

	36 CHANGE! hours to run for. same as above.

.. note:: 

	have this the same as the run time above. 

.. code-block::

	YYS MMS DDS 12 00 # CHANGE! start time, same as above

.. note:: 

	must be the same as the eruption start time, above, same definitions for
	YYS etc 

.. code-block::

	1
	S_LAT S_LON # grimsvotn is at centre of domain.

.. note:: 

	change this to the lat and lon of the the volcano - this is for the
	graphics 

.. code-block::

	0.2 0.2

.. note:: 

	this is the resolution in degrees - should be OK for most purposes 

.. code-block::

	90.0 360.0

.. note:: 

	this is the extent of the domain - should be OK for most purposes 

.. code-block::

	./
	cdumpVOLCICE

.. note:: 

	this is the name of the output particle file that we will need. 
	i.e. current directory, named cdumpVOLCICE

.. code-block::

	9 # number of conc levels to store data in, listed in next line
	0 1524 3048 4572 6096 7620 10668 15240 30000 # conc is stored on \\n
	these levels in m

.. note:: 

	these are the levels at which concentration is calculated. Should be OK
	for most purposes. They look odd but they are at flight levels (so they
	look better when expressed in feet). 

.. code-block::

	YYS MMS DDS 12 00 # CHANGE! start of output. keep to either \\n
	00,06,12,18

.. note:: 

	this is the first time the model will produce output. Met Office
	Requires output to be at 6 hourly, at 0Z, 6Z, 12Z, 18Z. So make sure
	this entry ends in either 06 00, 12 00, 18 00 or 00 00. Has to be equal
	to either the start time or after the start time but at one of the
	defined times above, E.g. if the eruption starts at 13:45, you would use
	18 00. Eruption = 00:20, use 06 00. Eruption = 19:45, use 00 00, etc. 

.. code-block::

	YYE MME DDE 00 00 # CHANGE! end of output. keep to either 00,06,12,18

.. note:: 

	similar to the above entry. Based on the length of your run, make this
	to be either 00 00, 06 00, 12 00, 18 00 - this is the last time that the
	model will produce output for. So, before or equal to the end time of
	the run. 

.. note:: 

	No need to change any of the following. 

.. code-block::

	-1 6 00 # averaging. every 6 hours, 1 hour average

.. note:: 

	This is agreed with the MO. don’t change! 

.. code-block::

	4 # number of pollutants depositing DON'T CHANGE ANY OF THE FOLLOWING

.. note:: 

	don’t change!! this gives a total of four ash species - and defines the
	particle size distribution, density for the four different ash species. 

.. code-block::

	0.6 2.5 1.0 #PARTICLE:DIAMETER (um), DENSITY (g/cc), SHAPE
	0 0.0 0.0 0.0 0.0 #DEP VEL (m/s), MW (g/Mole), SFC REACT. RATIO, DIFFUSIVITY RATIO, HENRY'S CONSTANT
	0.0 8.0E-05 8.0E-05 #WET REMOVAL: HENRY'S (Molar/atm), IN-CLOUD (1/s), BELOW-CLOUD (1/s)
	0 #RADIOACTIVE DECAY HALF-LIFE (days)
	0.0 #POLLUTANT RESUSPENSION (1/m)
	2.0 2.5 1.0 #PARTICLE:DIAMETER (um), DENSITY (g/cc), SHAPE
	0 0.0 0.0 0.0 0.0 #DEP VEL (m/s), MW (g/Mole), SFC REACT. RATIO, DIFFUSIVITY RATIO, HENRY'S CONSTANT
	0.0 8.0E-05 8.0E-05 #WET REMOVAL: HENRY'S (Molar/atm), IN-CLOUD (1/s), BELOW-CLOUD (1/s)
	0 #RADIOACTIVE DECAY HALF-LIFE (days)
	0.0 #POLLUTANT RESUSPENSION (1/m)
	6.0 2.5 1.0 #PARTICLE:DIAMETER (um), DENSITY (g/cc), SHAPE
	0 0.0 0.0 0.0 0.0 #DEP VEL (m/s), MW (g/Mole), SFC REACT. RATIO, DIFFUSIVITY RATIO, HENRY'S CONSTANT
	0.0 8.0E-05 8.0E-05 #WET REMOVAL: HENRY'S (Molar/atm), IN-CLOUD (1/s), BELOW-CLOUD (1/s)
	0 #RADIOACTIVE DECAY HALF-LIFE (days)
	0.0 #POLLUTANT RESUSPENSION (1/m)
	20.0 2.5 1.0 #PARTICLE:DIAMETER (um), DENSITY (g/cc), SHAPE
	0 0.0 0.0 0.0 0.0 #DEP VEL (m/s), MW (g/Mole), SFC REACT. RATIO, DIFFUSIVITY RATIO, HENRY'S CONSTANT

6. Run the HYSPLIT model
========================

You do this from the HYSPLIT directory that you copied on mummra.

To run:

.. code-block::

	../exec/hycs_std VOLCICE_auto

I usually run HYSPLIT in serial mode, but it is possible to run in
parallel - will speed things up, although this type of run doesn’t take
too long (10 mins or so).

.. note::

	To run using multiple processors, use e.g. mpiexec -np 10
	../exec/hycm_std VOLCICE_auto 

Output should look something like:

.. code-block::

	HYSPLIT - Initialization VOLCICE_auto
	HYSPLIT version: hysplit.v5.2.2
	Last Changed Date: 2022-06-22
	NOTICE main: using namelist file - SETUP.VOLCICE_auto
	Calculation Started ... please be patient
	Percent complete: 2.8
	Percent complete: 5.6
	Percent complete: 8.3
	Percent complete: 11.1
	Percent complete: 13.9
	Percent complete: 16.7
	Percent complete: 19.4
	…
	Percent complete: 91.7
	Percent complete: 94.4
	Percent complete: 97.2
	Percent complete: 100.0
	Complete Hysplit

The output that we will use is called ``cdumpVOLCICE``. Check that this has
been produced.

7. Produce a netcdf file for the plotting software
==================================================

There are several stages to this - we need to convert the HYSPLIT output
to netcdf, then we have to make several changes to the file itself.
Total column ash requires the sum of the four species of ash that we
use, so we need to combine these into one single field with a recognised
long_name. Also, HYSPLIT concentrations are in kg/m3, we need g/m3. You
can do this with a combination of cdo and ncks. There is a strange
1-hour offset which needs to be corrected also.

First use the HYSPLIT command

.. code-block:: 

	../exec/conc2cdf -icdumpVOLCICE -oTEST.nc

to get ``TEST.nc``

Then we use other packages to manipulate the netcdf file: primarily the
``ncks`` package but also ``cdo.``

.. code-block::

	ncap2 -s 'time=time+1' TEST.nc TESTout.nc

there is an offset of a day: no idea why! but correct it.

.. code-block::

	mv -f TESTout.nc TEST.nc

housekeeping

.. code-block::

	\\rm -f cdumpsum.nc cdumpsum1.nc

housekeeping

.. code-block::

	ncap2 -s 'ashsum=(ash1+ash2+ash3+ash4)' TEST.nc cdumpsum.nc

we want all the ash species together. ``cdumpsum.nc`` is now our working
file.

.. code-block::

	ncatted -a long_name,ashsum,o,c,Ashsum cdumpsum.nc

gives the combined ash field a name, needed by the plotting package

.. code-block::

	cdo -mulc,1000 cdumpsum.nc cdumpsum1.nc

convert to g/m3.

There is probably a more elegant way of doing this but the above works
and **every stage must be completed.**

8. Produce plots
================

The plotting package developed by BGS is used for this. It produces
plots with a projection (polar cylindrical), colour scale and contour
interval that have been developed by BGS, UKMO, and NCAS. It produces
total column ash - there is no other way to do this other than use the
package. Also, it produces concentration plots at various levels - these
may be useful to the MO, but it is **the total column ash plots that are
the key output**.

Having installed and activated the ``ash-model-plotting`` package, go into
the ash-model-plotting directory, and copy ``cdumpsum1.nc`` (from the
HYSPLIT directory) to this plotting directory.

First create a new directory where the results will be stored - e.g.
``test_output``.

The command to use is as follows - this gives a nice geographical extent
and includes most of Europe. To change the bounding box, change the
degree fields (``-40 40 20 85``)

.. code-block::

	python ash_model_plotting/plot_ash_model_results.py cdumpsum1.nc
	--model_type hysplit --limits -40 40 20 85 --output_dir test_output

(all one line) N.B. you may get lots of warnings, about units and other
things - these can be ignored.

This will put the plots in the ``test_output`` directory. You will have
directories of concentrations on various levels, 01524, 03048, 04572
etc. and **the important ones** **are the total column ash plots**,
``Total_Column_Mass_15762\_\*.png`` (ignore the ``15762`` bit - a package
artefact, I don’t know what it refers to). You will also get some
deposition plots - these have never been asked for but if needed, here
they are.

9. In the event of real eruption
================================

If an eruption happens,

(1) a VONA will arrive at the volcano@ncas.ac.uk email address hopefully
giving plume height - if not, have to guess! If it’s a major eruption,
assume say a 15 km plume height. If you have time you could do scenarios
(see comments above re. source.f). VONAs are issued by IMO.

(2) ``volcano@ncas.ac.uk`` may be contacted by the team at the UKMO, asking
if we are producing dispersion plots. This might not happen immediately
as they will be very busy. If you are able, then run HYSPLIT and produce
the plots as above.

The MO will provide a Google folder where the results can be put. Again,
this might be delayed due to busy schedules. Upload all the total column
ash plots to the folder they provide.

Things to tell the MO:

- This HYSPLIT run used the exact same particle size distributions as we
  have used in exercises. The HYSPLIT runs use the same Sparks-Mastin
  emission rate equation as NCAS has used in the exercises.

- The plots use BGS plotting software as in the exercises.

- Stress that this is exactly the same setup that is used in the
  exercises.This applies to number of particles released, model
  resolution, averaging time, etc. If pressed just say that the exact
  same configuration has been used except for the plume height.

- You will need to tell them the HYSPLIT-GFS cycle that we used (i.e.
  yesterday 18Z, etc.)
