

Run Directory Configuration
===========================

All GCHP run directories have default simulation-specific run-time settings that are set when you create a run directory. 
You will likely want to change these settings. 
This page goes over how to do this.

.. contents:: Table of contents
    :depth: 4

---------------------------------------------------------------------------------------------------

Configuration files
-------------------

GCHP is controlled using a set of configuration files that are included in the GCHP run directory. Files include:

1. :file:`CAP.rc`
2. :file:`ExtData.rc`
3. :file:`GCHP.rc`
4. :file:`input.geos`
5. :file:`HEMCO_Config.rc`
6. :file:`HEMCO_Diagn.rc`
7. :file:`input.nml`
8. :file:`HISTORY.rc`
9. :file:`logging.yml`

Several run-time settings must be set consistently across multiple files. 
Inconsistencies may result in your program crashing or yielding unexpected results. 
To avoid mistakes and make run configuration easier, bash shell script :file:`runConfig.sh` is included in all run directories to set the most commonly changed config file settings from one location. 
Sourcing this script will update multiple config files to use values specified in file.

Sourcing :file:`runConfig.sh` is done automatically prior to running GCHP if using any of the example run scripts, or you can do it at the command line. 
Information about what settings are changed and in what files are standard output of the script. 
To source the script, type the following:

.. code-block:: console

   $ source runConfig.sh

You may also use it in silent mode if you wish to update files but not display settings on the screen:

.. code-block:: console

   $ source runConfig.sh --silent

While using :file:`runConfig.sh` to configure common settings makes run configure much simpler, it comes with a major caveat. 
If you manually edit a config file setting that is also set in :file:`runConfig.sh` then your manual update will be overrided via string replacement. 
Please get very familiar with the options in :file:`runConfig.sh` and be conscientious about not updating the same setting elsewhere.

You generally will not need to know more about the GCHP configuration files beyond what is listed on this page. 
However, for a comprehensive description of all configuration files used by GCHP see the Configuration Files section of this user manual.

---------------------------------------------------------------------------------------------------

Common options
--------------

Compute configuration
^^^^^^^^^^^^^^^^^^^^^

Set number of nodes and cores
"""""""""""""""""""""""""""""
To change the number of nodes and cores for your run you must update settings in two places: (1) :file:`runConfig.sh`, and (2) your run script. 
The :file:`runConfig.sh` file contains detailed instructions on how to set resource parameter options and what they mean. 
Look for the Compute Resources section in the script. 
Update your resource request in your run script to match the resources set in :file:`runConfig.sh`.

It is important to be smart about your resource allocation. 
To do this it is useful to understand how GCHP works with respect to distribution of nodes and cores across the grid. 
At least one unique core is assigned to each face on the cubed sphere, resulting in a constraint of at least six cores to run GCHP. 
The same number of cores must be assigned to each face, resulting in another constraint of total number of cores being a multiple of six. 
Communication between the cores occurs only during transport processes.

While any number of cores is valid as long as it is a multiple of six (although there is an upper limit per resolution), you will typically start to see negative effects due to excessive communication if a core is handling less than around one hundred grid cells or a cluster of grid cells that are not approximately square. 
You can determine how many grid cells are handled per core by analyzing your grid resolution and resource allocation. 
For example, if running at C24 with six cores each face is handled by one core (6 faces / 6 cores) and contains 576 cells (24x24). 
Each core therefore processes 576 cells. Since each core handles one face, each core communicates with four other cores (four surrounding faces). Maximizing squareness of grid cells per core is done automatically within :file:`runConfig.sh` if variable :samp:`NXNY_AUTO` is set to :samp:`ON`.

Further discussion about domain decomposition is in :file:`runConfig.sh` section :literal:`Domain Decomposition`.

Change domain stack size
""""""""""""""""""""""""

For runs at very high resolution or small number of processors you may run into a domains stack size error. 
This is caused by exceeding the domains stack size memory limit set at run-time and the error will be apparent from the message in your log file. 
If this occurs you can increase the domains stack size in file :file:`input.nml`. The default is set to 20000000.

---------------------------------------------------------------------------------------------------

Basic run settings
^^^^^^^^^^^^^^^^^^

Set cubed-sphere grid resolution
""""""""""""""""""""""""""""""""
GCHP uses a cubed sphere grid rather than the traditional lat-lon grid used in GEOS-Chem Classic. 
While regular lat-lon grids are typically designated as ΔLat ⨉ ΔLon (e.g. 4⨉5), cubed sphere grids are designated by the side-length of the cube. 
In GCHP we specify this as CX (e.g. C24 or C180). 
The simple rule of thumb for determining the roughly equivalent lat-lon resolution for a given cubed sphere resolution is to divide the side length by 90. 
Using this rule you can quickly match C24 with about 4x5, C90 with 1 degree, C360 with quarter degree, and so on.

To change your grid resolution in the run directory edit the :literal:`CS_RES` integer parameter in :file:`runConfig.sh` section :literal:`Internal Cubed Sphere Resolution` to the cube side length you wish to use. 
To use a uniform global grid resolution make sure that :literal:`STRETCH_GRID` is set to :literal:`OFF`.

Set stretching parameters
"""""""""""""""""""""""""

GCHP has the capability to run with a stretched grid, meaning one portion of the globe is stretched to fine resolution. 
Set stretched grid parameter in :file:`runConfig.sh` section Internal Cubed Sphere Resolution. 
See instructions in that section of the file.

Turn on/off model components
""""""""""""""""""""""""""""

You can toggle all primary GEOS-Chem components, including type of mixing, from within :file:`runConfig.sh`. 
The settings in that file will update :file:`input.geos` and :file:`HEMCO_Config.rc` automatically. 
Look for section :literal:`Turn Components On/Off`. 
Please note that you can turn on/off all base emissions from :file:`runConfig.sh` but you cannot toggle HEMCO extensions or individual base emissions. For this you must manually edit :file:`HEMCO_Config.rc`.

Change model timestep
"""""""""""""""""""""

Model timesteps, both chemistry and dynamic, are configured within :file:`runConfig.sh`. 
They are set to match GEOS-Chem Classic default values for low resolutions for comparison purposes but can be updated, with caution. 
Timesteps are automatically reduced for high resolution runs. 
Read the documentation in :file:`runConfig.sh` section :literal:`Timesteps` for setting them.

Set simulation start and end dates
""""""""""""""""""""""""""""""""""

Set simulation start and end in :file:`runConfig.sh` section :literal:`Simulation Start, End, Duration, # runs`.
Read the comments in the file for a complete description of the options. 
Typically a "CAP" runtime error indicates a problem with start, end, and duration settings. 
If you encounter an error with the words "CAP" near it then double-check that these settings make sense.

---------------------------------------------------------------------------------------------------

Inputs
^^^^^^

Change initial restart file
"""""""""""""""""""""""""""

All GCHP run directories come with symbolic links to initial restart files for commonly used cubed sphere resolutions. 
The appropriate restart file is automatically chosen based on the cubed sphere resolution you set in :file:`runConfig.sh`.

You may overwrite the default restart file with your own by specifying the restart filename in :file:`runConfig.sh` section :literal:`Initial Restart File`. 
Beware that it is your responsibility to make sure it is the proper grid resolution.

Unlike GEOS-Chem Classic, HEMCO restart files are not used in GCHP. 
HEMCO restart variables may be included in the initial species restart file, or they may be excluded and HEMCO will start with default values. 
GCHP initial restart files that come with the run directories do not include HEMCO restart variables, but all output restart files do.

Turn on/off emissions inventories
"""""""""""""""""""""""""""""""""

Because file I/O impacts GCHP performance it is a good idea to turn off file read of emissions that you do not need. 
You can turn individual emissions inventories on or off the same way you would in GEOS-Chem Classic, by setting the inventories to true or false at the top of configuration file :file:`HEMCO_Config.rc`. 
All emissions that are turned off in this way will be ignored when GCHP uses :file:`ExtData.rc` to read files, thereby speeding up the model.

For emissions that do not have an on/off toggle at the top of the file, you can prevent GCHP from reading them by commenting them out in :file:`HEMCO_Config.rc`.
No updates to :file:`ExtData.rc` would be necessary. 
If you alternatively comment out the emissions in :file:`ExtData.rc` but not :file:`HEMCO_Config.rc` then GCHP will fail with an error when looking for the file information.

Another option to skip file read for certain files is to replace the file path in :file:`ExtData.rc` with :literal:`/dev/null`. 
However, if you want to turn these inputs back on at a later time you should preserve the original path by commenting out the original line.

Add new emissions files
"""""""""""""""""""""""

There are two steps for adding new emissions inventories to GCHP:

1. Add the inventory information to :file:`HEMCO_Config.rc`.
2. Add the inventory information to :file:`ExtData.rc`.
3. To add information to :file:`HEMCO_Config.rc`, follow the same rules as you would for adding a new emission inventory to GEOS-Chem Classic. 
   Note that not all information in :file:`HEMCO_Config.rc` is used by GCHP. 
   This is because HEMCO is only used by GCHP to handle emissions after they are read, e.g. scaling and applying hierarchy. 
   All functions related to HEMCO file read are skipped. 
   This means that you could put garbage for the file path and units in :file:`HEMCO_Config.rc` without running into problems with GCHP, as long as the syntax is what HEMCO expects. 
   However, we recommend that you fill in :file:`HEMCO_Config.rc` in the same way you would for GEOS-Chem Classic for consistency and also to avoid potential format check errors.

Staying consistent with the information that you put into :file:`HEMCO_Config.rc`, add the inventory information to :file:`ExtData.rc` following the guidelines listed at the top of the file and using existing inventories as examples. 
You can ignore all entries in :file:`HEMCO_Config.rc` that are copies of another entry since putting these in :file:`ExtData.rc` would result in reading the same variable in the same file twice. 
HEMCO interprets the copied variables, denoted by having dashes in the :file:`HEMCO_Config.rc` entry, separate from file read.

A few common errors encountered when adding new input emissions files to GCHP are:

1. Your input file contains integer values. 
   Beware that the MAPL I/O component in GCHP does not read or write integers. 
   If your data contains integers then you should reprocess the file to contain floating point values instead.
2. Your data latitude and longitude dimensions are in the wrong order. 
   Lat must always come before lon in your inputs arrays, a requirement true for both GCHP and GEOS-Chem Classic. 
3. Your 3D input data are mapped to the wrong levels in GEOS-Chem (silent error). 
   If you read in 3D data and assign the resulting import to a GEOS-Chem state variable such as :literal:`State_Chm` or :literal:`State_Met`, then you must flip the vertical axis during the assignment. 
   See files :file:`Includes_Before_Run.H` and setting :literal:`State_Chm%Species` in :file:`Chem_GridCompMod.F90` for examples.
4. You have a typo in either :file:`HEMCO_Config.rc` or :file:`ExtData.rc`. Error in :file:`HEMCO_Config.rc` typically result in the model crashing right away. 
   Errors in :file:`ExtData.rc` typically result in a problem later on during ExtData read. 
   Always try running with all debug prints on in your first test with new emissions. 
   See separate section on this page about turning on debug prints, or check the debug print section of :file:`runConfig.sh` for instructions. 
   Another useful strategy is to find config file entries for similar input files and compare them against the entry for your new file. 
   Directly comparing the file metadata may also lead to insights into the problem.

---------------------------------------------------------------------------------------------------

Outputs
^^^^^^^

Output diagnostics data on a lat-lon grid
"""""""""""""""""""""""""""""""""""""""""

See documentation in the :file:`HISTORY.rc` config file for instructions on how to output diagnostic collection on lat-lon grids. If outputting on a lat-lon grid you may also output regional data instead of global.

Output restart files at regular or irregular frequency
""""""""""""""""""""""""""""""""""""""""""""""""""""""

The MAPL component in GCHP has the option to output restart files (also called checkpoint files) prior to run end. 
The frequency of restart file write may be at regular time intervals (regular frequency) or at specific programmed times (irregular frequency). 
These periodic output restart files contain the date and time in their filenames.

Enabling this feature is a good idea if you plan on doing a long simulation and you are not splitting your run into multiple jobs. 
If the run crashes unexpectedly then you can restart mid-run rather than start over from the beginning.

Update settings for checkpoint restart outputs in :file:`runConfig.sh` section :literal:`Output Restarts`. 
Instructions for configuring both regular and irregular frequency restart files are included in the file. 
Note that because months have an irregular number of days, outputting monthly checkpoint files requires using the irregular checkpoint option.

Turn on/off diagnostics
"""""""""""""""""""""""

To turn diagnostic collections on or off, comment ("#") collection names in the "COLLECTIONS" list at the top of file :file:`HISTORY.rc`. 
Collections cannot be turned on/off from :file:`runConfig.sh`.

Set diagnostic frequency, duration, and mode
""""""""""""""""""""""""""""""""""""""""""""

All diagnostic collections that come with the run directory have frequency and duration auto-set within :file:`runConfig.sh`. 
The file contains a list of time-averaged collections and instantaneous collections, and allows setting a frequency and duration to apply to all collections listed for each. Time-avraged collections also have a monthly mean option (see separate section on this page about monthly mean). 
To avoid auto-update of a certain collection, remove it from the list in :file:`runConfig.sh`, or set :literal:`AutUpdate_Diagnostics` to :literal:`OFF`. 
See section :literal:`Diagnostics` within :file:`runConfig.sh` for examples. 

Add a new diagnostics collection
""""""""""""""""""""""""""""""""

Adding a new diagnostics collection in GCHP is the same as for GEOS-Chem Classic netcdf diagnostics. 
You must add your collection to the collection list in :file:`HISTORY.rc` and then define it further down in the file. 
Any 2D or 3D arrays that are stored within GEOS-Chem objects :literal:`State_Met`, :literal:`State_Chm`, or :literal:`State_Diag`, may be included as fields in a collection. 
:literal:`State_Met` variables must be preceded by "Met\_", :literal:`State_Chm` variables must be preceded by "Chem\_", and :literal:`State_Diag` variables should not have a prefix. 
Collections may have a combination of 2D and 3D variables, but all 3D variables must have the same number of levels.
See the :file:`HISTORY.rc` file for examples.

Once implemented, you can either incorporate the new collection settings into :file:`runConfig.sh` for auto-update, or you can manually configure all settings in :file:`HISTORY.rc`.
See the :literal:`Diagnostics` section of :file:`runConfig.sh` for more information.

Generate monthly mean diagnostics
"""""""""""""""""""""""""""""""""

You can toggle monthly mean diagnostics on/off from within :file:`runConfig.sh` in the diagnostics configurations section. 
If you do this then ALL time-averaged diagnostic collections listed in :file:`runConfig.sh` will have monthly diagnostics enabled. 
As for collection frequency and duration, you can remove collections from the list in :file:`runConfig.sh` to prevent auto-update, or set :literal:`AutoUpdate_Diagnostics` to :literal:`OFF`, and then manually update settings for those collections in :file:`HISTORY.rc` prior to your run. 
This would allow you to have some time-averaged diagnostics be monthly mean and some not. 
If using monthly diagnostics, be sure to always start and end your run on the first day and time of the month.

---------------------------------------------------------------------------------------------------

Debugging
^^^^^^^^^

Enable maximum print output
"""""""""""""""""""""""""""

Besides compiling with :literal:`CMAKE_BUILD_TYPE=Debug`, there are a few settings you can configure to boost your chance of successful debugging.
All of them involve sending additional print statements to the log files.

1. Set Turn on debug printout? in input.geos to T to turn on extra GEOS-Chem print statements in the main log file.
2. Set the Verbose and Warnings settings in :file:`HEMCO_Config.rc` to maximum values of 3 to send the maximum number of prints to :file:`HEMCO.log`.
3. Set :literal:`CAP.EXTDATA` option :literal:`root_level` in :file:`logging.yml` to :literal:`DEBUG` to send root thread MAPL ExtData (input) prints to :file:`allPEs.log`.
4. Set :literal:`CAP.EXTDATA` option :literal:`level` in :file:`logging.yml` to :literal:`DEBUG` to send all thread MAPL ExtData (input) prints to :file:`allPEs.log`.

None of these options require recompiling. 
Be aware that all of them will slow down your simulation. 
Be sure to set them back to the default values after you are finished debugging.
