# PRESTO

http://www.cv.nrao.edu/~sransom/presto/

PRESTO is a large suite of pulsar search and analysis software developed
primarily by Scott Ransom mostly from scratch, and released under the GPL (v2).
It was primarily designed to efficiently search for binary millisecond pulsars
from long observations of globular clusters (although it has since been used in
several surveys with short integrations and to process a lot of X-ray data as
well).  It is written primarily in ANSI C, with many of the recent routines in
Python.  According to Steve Eikenberry, PRESTO stands for: PulsaR Exploration
and Search TOolkit!

**PRESTO has discovered over 1000 pulsars, including approximately 400 recycled
and/or binary pulsars!**

## New in Version 4.0:
 * This is a **major release** since it involves big changes to the Python
   portions of the codebase:
   * Python v3.7 or newer is now required.
   * A long-standing memory issue was fixed with Anaconda Python (running
     `python tests/test_presto_python.py` will tell you if you have that issue
     or not).
   * Swig v4 is used to generate the Python wrappers of the PRESTO C library.
   * Big thanks to **Shami Chatterjee** and **Bradley Meyers** who helped me get
     to the bottom of this!
 * There is a [FAQ](https://github.com/scottransom/presto/blob/master/FAQ.md)
   with lots of information!
 * PRESTO has a dockerfile that allows it to build on Docker Hub automatically.
   Thanks to **Nick Swainston** for this!  (more testing and improvements would
   be welcome)
 * `simple_zapbirds.py` makes it much easier to manually zap interference from
   simple searches (no need for copying ".inf" files and running both
   `makezaplist.py` and `zapbirds`).
 * `realfft` and `zapbirds` can now be called on many files at once on the
   command line. This benefits HPC systems which often don't like many programs
   running serially on many small files.
 * A new python interface to the internal `prepfold` folding code
   (`simplefold`), as well as wrappers of fast `C` implementations of $\chi^2$
   and $Z^2_N$ (thanks to **Matteo Bachetti**).
 * Many bug fixes and minor improvements, including one that would cause
   segfaults with very large dispersion sweeps in `prepdata` and `prepsubband`,
   and a problem with `prepfold` significance calculations.

## In Version 3.0.1:
 * This is a minor release which fixes several issues and adds some minor
   improvements:
   * Fix of long-standing `rfifind` bug that could cause the program to hang if
     channels had zero variance
   * Multiple Python3-related bug fixes
   * Added `-debug` flag to `prepfold` to allow debugging of TEMPO calls to make
     polycos
   * `DDplan.py` can now read observation parameters from filterbank or PSRFITS
     input files. And you can write a `dedisp_*.py` dedispersion script, based
     on the plan, using the `-w` option
   * The `rednoise` program now writes a corresponding *_red.inf file
   * Update of the Tutorial document, including a new slide on red noise

## In Version 3.0:
 * This major release of PRESTO includes a massive restructuring of python code
   and capabilities. Things should work with Python versions 2.7 and Python 3.6
   and 3.7 at least. The installation of the python code has changed and has
   become more "pythonic" so that `PYTHONPATH` is not needed, and all of the
   various modules are now under a top-level "presto" module. For example, to
   use the psr_utils module you would now do:
   
   `import presto.psr_utils as pu`
   
   rather than

   `import psr_utils as pu`

   All of these changes will likely lead to code breakage and bugs!

   Please check your code and processing carefully and post issues (and
   hopefully pull requests) if you find them.

   The installation instructions have been updated in the INSTALL file.

   Huge thanks thanks go to **Gijs Molenaar, Matteo Bachetti, and
   Paul Ray** for the work that they have done helping with this!

 * There is also a new `examplescripts` directory where you will find some
   example code to do a lot of important things, like
   * Fully dedispersing an observation: `dedisp.py`
   * Fully searching a dedispersed observation: `full_analysis.py`
   * Sifting the results of a full search: `ACCEL_sift.py`
   * Searching short chunks of a long time series: `short_analysis_simple.py`
   * Making a really nice P-Pdot plane: `ppdot_plane_plot.py`
   * and a few others.

## Status of Version 2.2:
 * Version 2.2 was the last version of PRESTO to work with the old-style python
   interface which requires Python v2.7 or earlier and is "installed" in-place
   and used via having `$PRESTO/lib/python` in your `PYTHONPATH`. There will
   probably be occasional bug fixes for v2.2 in the `v2.2maint` branch of
   PRESTO. You can get it using:

   `git checkout -b v2.2maint origin/v2.2maint`

   and then installing as per the INSTALL file.

## Improvements in Version 2.1:
 * `accelsearch` now has a "jerk" search capability (thanks to (then) UVA
   undergrad **Bridget Andersen** for help with this!).  This makes searches
   take a *lot* longer, but definitely improves sensitivity when the observation
   duration is 5-15% of the duration of the orbital period.  Typically `-wmax`
   should be set to 3-5x `-zmax` (and you probably never need to set `-zmax` to
   anything larger than 300).
 * Ability to ignore bad channels on the command line (-ignorechan) (see
   `rfifind_stats.py` and `weights_to_ignorechan.py`)

## About PRESTO:
PRESTO is written with portability, ease-of-use, and memory efficiency in mind,
it can currently handle raw data from the following pulsar machines or formats:

 * PSRFITS search-format data (as from GUPPI at the GBT, PUPPI and the Mock
   Spectrometers at Arecibo, and much new and archived data from Parkes)
 * 1-, 2-, 4-, 8-, and 32-bit (float) filterbank format from SIGPROC
 * A time series composed of single precision (i.e. 4-byte) floating point data
   (with a text ".inf" file describing it)
 * Photon arrival times (or events) in ASCII or double-precision binary formats

Notice that the following formats which *used* to be supported are not:

 * Wideband Arecibo Pulsar Processor (WAPP) at Arecibo
 * The Parkes and Jodrell Bank 1-bit filterbank formats
 * SPIGOT at the GBT
 * Berkeley-Caltech Pulsar Machine (BCPM) at the GBT

If you need to process them, you can either checkout the "classic" branch of
PRESTO (see below), which is not being actively developed. Or you can use DSPSR
to convert those formats into SIGPROC filterbank or (even better) PSRFITS search
format. You can grab DSPSR [here](http://dspsr.sourceforge.net).  If you
*really* need to get one of these machines working in modern PRESTO, let me know
and we can probably make it happen.

The software is composed of numerous routines designed to handle three main
areas of pulsar analysis:

1. Data Preparation: Interference detection (`rfifind`) and removal (`zapbirds`)
   , de-dispersion (`prepdata`, `prepsubband`, and `mpiprepsubband`),
   barycentering (via TEMPO).
2. Searching: Fourier-domain acceleration (`accelsearch`), single-pulse
   (`single_pulse_search.py`), and phase-modulation or sideband searches
   (`search_bin`).
3. Folding: Candidate optimization (`prepfold`) and Time-of-Arrival (TOA)
   generation (`get_TOAs.py`).
4. Misc: Data exploration (`readfile`, `exploredat`, `explorefft`),
   de-dispersion planning (`DDplan.py`), date conversion (`mjd2cal`, `cal2mjd`),
   tons of python pulsar/astro libraries, average pulse creation, flux density
   estimation, and more...
5. Post Single Pulse Searching Tools: Grouping algorithm (`rrattrap.py`),
   Production and of single pulse diagnostic plots (`make_spd.py`,
   `plot_spd.py`, and `waterfaller.py`).

Many additional utilities are provided for various tasks that are often required
when working with pulsar data such as time conversions, Fourier transforms, time
series and FFT exploration, byte-swapping, etc.

**References**: The Fourier-Domain acceleration search technique that PRESTO
uses in the routine `accelsearch` is described in [Ransom, Eikenberry, and
Middleditch
(2002)](https://ui.adsabs.harvard.edu/abs/2002AJ....124.1788R/abstract), the
"jerk" search capability is described in [Andersen & Ransom
(2018)](https://ui.adsabs.harvard.edu/abs/2018ApJ...863L..13A/abstract), and the
phase-modulation search technique used by `search_bin` is described in [Ransom,
Cordes, and Eikenberry
(2003)](https://ui.adsabs.harvard.edu/abs/2003ApJ...589..911R/abstract).  Some
other basic information about PRESTO can be found in my
[thesis](http://www.cv.nrao.edu/~sransom/ransom_thesis_2001.pdf).

**Support/Docs**:  I may eventually get around to finishing the documentation
for PRESTO (or not), but until then you should know that each routine returns
its basic usage when you call it with no arguments. I am also willing to provide
limited support via email (see below).  And make sure to check out the `FAQ.md`!

**Tutorial**: There is a tutorial in the "docs" directory which walks you
through all the main steps of finding pulsars using PRESTO.

## Getting it: 
The PRESTO source code is released under the GPL and can be browsed or gotten
from here in many different ways (including zipped or tar'd or via git).  If you
are too lazy to read how to get it but have git on your system do:

    git clone git://github.com/scottransom/presto.git

To update it on a regular basis do

    cd $PRESTO
    git pull

and then re-make things in $PRESTO/src.

For more detailed installation instructions, see `INSTALL.md`.

If you don't want to mess with git (which means that you will need to re-install
a tarball whenever there are updates) you can get it from the "Download Source"
link on the github page.

If you want the "classic" branch, do the following:

    git clone git://github.com/scottransom/presto.git
    cd presto
    git checkout -b classic origin/classic

then build as per the (old) INSTALL file.

### Development:

If you plan to tweak the code, I highly suggest that you use git and clone the
directory (or fork it using an account on github).  And if you want to
contribute your changes back, please give me a "pull request"!

**Code contributions and/or patches to fix bugs are most welcome!**

### Final Thoughts:
Please let me know if you decide to use PRESTO for any "real" searches,
especially if you find pulsars with it!

And if you find anything with it, it would be great if you would cite either my
thesis or whichever of the three papers listed above is appropriate.

Also note that many people are now citing software using the ASCL. [PRESTO is
there as well](https://www.ascl.net/1107.017).

Thanks!

### Acknowledgements:
Big thanks go to Steve Eikenberry for his help developing the algorithms, Dunc
Lorimer and David Kaplan for help with (retired) code to process BCPM, SCAMP,
and Spigot data, among other things, Jason Hessels and Patrick Lazarus for many
contributions to the Python routines, and (alphabetical): Bridget Andersen, Anne
Archibald, Cees Bassa, Matteo Bachetti, Slavko Bogdanov, Fernando Camilo, Shami
Chatterjee, Kathryn Crowter, Paul Demorest, Paulo Freire, Nate Garver-Daniels,
Chen Karako, Mike Keith, Maggie Livingstone, Ryan Lynch, Erik Madsen, Bradley
Meyers, Gijs Molenaar, Timothy Olszanski, Chitrang Patel, Paul Ray, Alessandro
Ridolfi, Paul Scholz, Maciej Serylak, Ingrid Stairs, Kevin Stovall, Nick
Swainston, and Joeri van Leeuwen for many comments, suggestions and patches!

Scott Ransom <sransom@nrao.edu>



-------------For IAR----------------


If we want presto inside our ipta docker we need presto 2.2 (forpython 2)so...


git clone git://github.com/scottransom/presto.git

cd presto

git checkout -b v2.2maint origin/v2.2maint

Just in case make sure you are in the v2.2maint branch, take a look at README.md , src/polycos.c to check iar codes...

Note, to use presto tools with our location and codes, we need to edit the following files from...

TEMPO:

obsys.dat

tempo2/T2runtime/observatory/Tempo2:

aliases  observatories.dat

PRESTO: 

telescope_to_tempocode() in misc_utils.c

make_polycos() in polycos.c

get_telescope_name() in sigproc_fb.c


SIGPROC: 

and aliases.c


This data will be used by the .fil header file, from the .iar file

Telescope ID,

IAR-A1: 19, "IAR1", "A1", "m"

IAR-A2: 20, "IAR2", "A2", "o",

IAR-ROACH-A1: 21, "IAR1R", "R1", "r"

IAR-ROACH-A2: 22, "IAR2R", "R2", "s",

DSA-3: 24, "DSA3", "D3", "p",

CLTC: 25, "CLTC", "CL", "q",


Machine ID,

RTL_Filterbank: 23

IAR_ROACH_v1: 24

IAR_SNAP_v1: 25

#Now build PRESTO, addto path: $export PRESTO=/home/ggancio/presto

cd $PRESTO/src

make makewisdom

make prep

make

#For some reason it does not find slalib in the first run

make

make clean

cd $PRESTO/python

make

make clean


