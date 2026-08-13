grasshopper
===========

<img src="figures/icon.png" width="100" height="auto" alt="icon">

This application is based on the Geant4 development toolkit, which allows to program complex particle
tracking (e.g. gamma, electrons, protons, etc) and particle-matter interaction MC simulations.
Grasshopper is a simple Geant4 application, where all the geometries and even generators parameters are 
defined in a GDML file, with the purpose of setting up quick and simple simulations.  The
goal is to allow users with no C++ and Geant4 knowledge *quickly* set up and run simulations.

<span style="color: red;">**Discussion Forum**</span>: you can join grasshopper's WhatsApp community using this [link here](https://chat.whatsapp.com/LCp8bb2xUJY3eZ1OngeuRs?mode=gi_t).

Author:  		Areg Danagoulian

Creation time:  11/2015  
Last update:    continuous

For copyright and licensing see files COPYRIGHT and LICENSE.

To install
==

This will require Make and CMake.  If you're on the engaging cluster, CMake is part of a module that's not loaded by default, so make sure you load that (and it's dependency, gcc) first.
```bash
module load gcc
module load cmake
```

### Using CMake
First of all, if you haven't used CMake, you need to use it thrice here, so here's a brief summary of how it works.
In the year 2000, computer programmers had invented the computer program but hadn't yet invented computer programs that do multiple things.
So instead of shipping software with a script to install the software like modern Python libraries, they shipped it with a bunch of folders full of files that you install in three separate steps using two separate build tools.

First, you create a "build" directory in which you'll dump all of the data that needs to be passed between the build tools.
```bash
mkdir myprogram-build
```
Then you run CMake to configure the source code into the build directory, and pass most of your compilation options using the `-D` flag.
One of the more important compilation options is `CMAKE_INSTALL_PREFIX` which is the folder where you want to put the built binaries and libraries.
If you leave it blank it'll put them somewhere high up in your file system where they're hidden away and presumably on your paths,
but then you'll need to use `sudo` when you install.  If you don't have super-user rights, you'll have to specify a different directory to which you have write access.
The locations of dependencies are also often compilation options.
```bash
cd myprogram-build
cmake -D OPTION=ON -D ANOTHER_OPTION=OFF -D CMAKE_INSTALL_PREFIX=~/myprogram-install ~/myprogram-src
```
Then you run regular Make to build the binaries and libraries.
```bash
make
```
Then you run regular Make in "install" mode to move the binaries and libraries from wherever it put them into the correct folder.
```bash
make install
```

If either of the Make steps gets interrupted or you make a mistake and have to start over, make sure to call Make in "clean" mode to make it clean up after its mistakes.
```bash
make clean
```

### Installing Xerces

First, you need Apache Xerces, an XML parser, which is a prerequisite for Geant4 with GDML, which is a prerequisite for Grasshopper.
This one doesn't require any CMake arguments other than maybe an install directory.

```bash
mkdir ~/xerces-build
cd ~/xerces-build
cmake -D CMAKE_INSTALL_PREFIX=~/xerces-install ~/xerces-build
make
make install
```

After it's installed, if you changed the install directory, you need to add the install's library subdirectory to your library path.
```bash
export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:~/xerces-install/lib64
```
Note that sometimes the library subdirectory is `lib/` and sometimes it's `lib64/`.
I don't know what determines when it's which; you just have to look inside the installation and check the name of the folder that contains the shared object files.

### Installing Geant4

Now you need to install Geant4.  For this, you'll need Expat.  IDK what Expat is, but most systems have it already installed so you probably don't have to worry about installing it.
But if you're on the Engaging cluster you need to load it as a module, and it's hidden.  So first you have to search for it with
```bash
module --show_hidden spider expat
```
and then you have to `module load` one of the available versions that comes up.
If you have issues, an alternative approach is to configure Geant4 to use its internal copy of Expat with `-D GEANT4_USE_SYSTEM_EXPAT=OFF`

Then you can move on to Geant4 itself.  Download the source, then configure it with CMake, then build it with Make, then install it with Make.
There are many CMake options for Geant4, but the important ones are:
* `GEANT4_USE_GDML` which Grasshopper requires be set to `ON`
* `GEANT4_INSTALL_DATA` which must be set to `ON` but is off by default for some reason
* `GEANT4_BUILD_MULTITHREADED` which you presumably want `ON`, especially if you're building on a cluster
* `CMAKE_PREFIX_PATH` which must point to Xerces if Xerces wasn't installed in the default place

The build step takes a while here, so you might want to move it into a Slurm partition if you're on a shared computing cluster.

```bash
mkdir ~/geant4-build
cd ~/geant4-build
cmake -D GEANT4_USE_GDML=ON -D GEANT4_INSTALL_DATA=ON -D GEANT4_BUILD_MULTITHREADED=ON -D CMAKE_PREFIX_PATH=~/xerces-install -D CMAKE_INSTALL_PREFIX=~/geant4-install ~/geant4-src
sbatch --time=2:00:00 --wrap="make && make install"
```

After it's installed, if you changed the install directory, you need to add the install's library subdirectory to your library path.
```bash
export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:~/geant4-install/lib64
```
Note that sometimes the library subdirectory is `lib/` and sometimes it's `lib64/`.
I don't know what determines when it's which; you just have to look inside the installation and check the name of the folder that contains the shared object files.

### Installing Grasshopper

Finally, Grasshopper.  Download the source, then configure it with CMake, then build it with Make, then install it with Make.
The only CMake option you need is `DGeant4_DIR` which should point to the `lib/cmake/Geant4/` subfolder of the Geant4 installation,
if Geant4 wasn't installed in the default place.

```bash
mkdir ~/grasshopper-build
cd ~/grasshopper-build
cmake -D Geant4_DIR=~/geant4-install/lib/cmake/Geant4 -D CMAKE_INSTALL_PREFIX=~/grasshopper-install ~/grasshopper-src
make
make install
```

After it's installed, you need to manually add the Grasshopper executable to your path.  For example:
```bash
export PATH=$PATH:~/grasshopper-install/bin
```

Once this is done, you can remove all of the build directories, plus all of the source code directories if you don't plan to make changes.


Python GUI frontend
==
A point-and-click interface for Grasshopper is provided in the `gui/` directory.  It covers the full workflow — defining the simulation, generating the GDML file, running the executable, and inspecting the output — without any command-line or XML editing.

```bash
pip install -r gui/requirements.txt   # matplotlib, numpy; vtk optional
python3 gui/grasshopper_gui.py
```

See [`gui/README.md`](gui/README.md) for full installation instructions and a description of each tab.

Visualization
==
If you would like to have grasshopper automatically open the visualization `.wrl` file, do this before running:   
`export G4VRMLFILE_VIEWER=paraview`  
Here we use paraview, and the syntax corresponds to bash. If you are using a different software for rendering the .wrl files, change [paraview] accordingly.


Tutorial
==
The best way to learn how to use grasshopper is by using the tutorial on the project wiki, [here](https://github.com/ustajan/grasshopper/wiki).  You can also get there by going to Wiki link above.

Input
==
 * input.gdml -- the geometry definition markup language file.  This file defines
   * the geometries of the objects and detectors, as well as their materials and the positioning
   * the particle type, energy, and direction
   * various computational optimization options
   * output formats
 * input\_spectrum.txt -- this is currently a fixed file name.  The user can define the energy of the particle.
   If the energy is set negative, the code defaults to reading the energy spectrum from "input\_spectrum.txt" and samples
   the energy from that spectrum.
   
Output
==
The code will generate three files:

 * output.root -- The code will always generate a root file, as specified in the input arguments.
 * output.dat  -- depending on the settings in the gdml file, it can also generate a simple ascii file, output.dat,  which can then be read into python/matlab/etc.
 * g4_00.wrl.  This is the VRML visualization file.  The settings for this file's content are defined in the gdml file.
   The wrl file can be loaded in paraview (http://www.paraview.org/download/), thus allowing for a rendering of the
   geometries and some particle tracks.

The root output structure (which is identical to the ascii output) can be somewhat complex. The output of the simulation can be
thought of as a table -- where every line is an entry corresponding to a **particular set of information**
within an event. Below is a set of entries in the ASCII output from a simulation of a 40meV neutron beam undergoing capture inside two 3He ionization chamber (numbered 5 and 37):

![alt text](https://github.com/ustajan/grasshopper/blob/master/documentation/ascii_long.png?raw=true)

By enabling the BriefOutput flag in the gdml, it's is possible to get the shorter version of the above:

![alt text](https://github.com/ustajan/grasshopper/blob/master/documentation/ascii_short.png?raw=true)


There are three type of entries:
 * IsSurfaceHitTrack.  These entries are registered when a track enters the detector volume.  To filter out these events, simply search for all entries where IsSurfaceHitTrack==1
 * regular tracks.  At the end of an event all the tracks inside the detector can be recorded in the form of individual entries.  All entries for which IsSurfaceHitTrack!=1 && IsEdepositedTotalEntry!=1 are regular track entries.  These entries are useful for debugging and diagnostic purposes, as well as for general studies of the interaction physics of the detector.
 * IsEdepositedTotalEntry.  At the end of an event all the relevant tracks' deposited energies is summed into E\_deposited variable, and a dedicated entry is made in the tree with this information.  By filtering IsEdepositedTotalEntry==1, all the deposition energy entries can be selected.  This would be useful for studying the detector response function.  These entries have the additional feature of including a concatenation of all the processes which contributed to the deposited energy.  For example, for EventID=0 the energy deposition was achieved via the main track (EventGenerator), a compton scatter and a photoelectric effect (see the last line above).

The variables in the tree/table are as following:
 * E\_beam -- the energy of the particle produced by the event generator
 * E\_incident    -- only for IsSurfaceHitTrack==1 entries this is the energy of the particle at the detector entrance.  Good way to study the flux of the particles along a surface.
 * E\_deposited   -- the deposited energy in the detector
 * x\_incident    -- for IsSurfaceHitTrack==1 or SaveTrackInfo==1 the hit position on the detector, in mm
 * y\_incident    --  same
 * z\_incident    --  same
 * theta          --  same (radians)
 * Time           --  the (flight) time from the inception of the event
 * EventID        -- this is the # of the event in the simulation history
 * TrackID        -- for a particular event, many tracks may be produced.  They will all have the same EventID and different TrackIDs
 * ParticleID     -- for a particular track, this is the ParticleID, based on LLNL Particle Data Group definitions.
 * ParticleName   -- the actual name (which is more useful than the ParticleID)
 * CreatorProcessName         -- the name of the process which created this track
 * IsEdepositedTotalEntry     -- the flag for total deposited energy entries
 * IsSurfaceHitTrack          -- the flag for surface hit entries
 * detector#       -- the number of the detector.  The code assumes that all detector volumes are named **det_phys#**, where **#** is the detector number.  The code trunkates the **det_phys** and thus extracts the detector number.

The CreatorProcessName for IsEdepositedTotalEntry==1 entries contains a concatenation of all the processes which contributed to the deposited energy.  This is a good way to study the various effecst which, for example, contribute to the photopeak and the Compton continuum in a gamma detector.

Finally, it is also possible to request a brief output, in the form of EventID, Energy, ParticleName, and CreatorProcessName.  To do this set the BriefOutputOn flag to 1, in the gdml input.

The output can be modified to be only limited for just one or two of these entry types.  The gdml file allows to do this using the SaveSurfaceHitTrack, SaveTrackInfo, SaveEdepositedTotalEntry variables.  For example, for a simulation where the flux is to be determined, only SaveSurfaceHitTrack needs to be set.  For studying the energy deposition distribution
for a particular type of detector the SaveEdepositedTotalEntry needs to be set.  The use of these variables can significantly simplify the analysis of the MC output.



General status
==
In its current form, the code has been tested with <12MeV photons, neutrons, gammas, and electrons.  Very general checks indicate that most processes are being simulated correctly.


To do
==
Below is a prioritized list of future tasks.

* ~~Write the python/javascript front end to the gdml?~~ — done, see `gui/`
* General code improvements

