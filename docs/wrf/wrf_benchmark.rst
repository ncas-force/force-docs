RF configuration for testing performance
#########################################

1. Namelist files
=================

A simple WRF configuration to test speed-up on various machines and compilers can be found on mummra in

.. code-block::

     /home/lecrrb/wrf/master/WRF_big_test/test/em_real   

.. note:: 

        You should be able to run WRF with ``wrf_bdy*`` and ``wrfinput_*`` files, butif you need to recreate the WPS ``met_em*`` files the relevant directory is ``/home/lecrrb/wrf/master/WPS_big_test``

This has 2 domains, 500x500 gridpoints in each domain, 6 hour simulation, 1 output file per domain per hour. All other namelist options as per the NWR UK runs.

2. Reference simulation timing
==============================

For reference, this run was completed on ``liono`` with 64 processors. 

**Wallclock time**: 176 minutes


**Output times** (average): 

Domain 1: 10.5s

Domain 2: 10.7s

.. note::
     If no other method is available the total wallclock time can be estimated from the difference between the write time of ``namelist.output`` and ``rsl.out.0000`` . 

To get average output times for different domains you can use

.. code-block::

     grep "Timing for Writing" rsl.out.0000 | grep "2:" | awk '{sum += $(NF-2); count++} END {if (count > 0) print "Average elapsed seconds:", sum / count}'

This is for domain 2, change ``2:`` to ``1:`` for domain 1.

