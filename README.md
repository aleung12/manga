# manga
Kinematic modeling of disk galaxies in MaNGA via parallel-tempered MCMC

SDSS-IV MaNGA (Mapping Nearby Galaxies at the Apache Point Observatory) is a large 
IFU survey with a goal of observing ~10,000 galaxies by the survey's end. MaNGA 
produces spatially resolved kinematic maps describing the motions of stars and gas 
in each galaxy, but these maps include the effects of beam smearing due to the 
intrinsic velocity gradient in the signal incident on individual IFU fibers, which 
inflates the measured velocity dispersion.

The primary application of this code is to recover intrinsic velocity and dispersion 
fields for disk galaxies observed by MaNGA. The code assumes a general 
nonaxisymmetric velocity model that permits noncircular motions (Spekkens & Sellwood 
2007, ApJ, 664, 204); velocity dispersion is initialized as an exponential profile 
declining with radius and fitted nonparametrically.

Basic descriptions of files included in this repository: 

[1] ```make_slurm.py``` (if running on Texas Advanced Computing Center)
 - generates a slurm file to run ```mcmc_tacc.py``` for a MaNGA galaxy and submits 
   job to queue on TACC

[2] ```mcmc_tacc.py```
 - implements a parallel-tempered Markov Chain Monte Carlo (MCMC) method (e.g., 
   Vousden 2016, MNRAS, 455, 1919) to determine the most probable intrinsic model 
   given the observed kinematic data for a MaNGA disk galaxy
 - to initiate a run from command line, execute, e.g.: 
 
   ```
   python mcmc_tacc.py 8604-12701 None 10 1
   ```
    - the first argument after the file name is the MaNGA designation for the galaxy
    - the second argument specifies starting from scratch (as opposed to continuing 
      a run that had been stopped, picking up from the last completed iteration)
    - the third argument specifies the number of MCMC samplers or "walkers" that will 
      explore the parameter space in parallel for each component (the example 
      specifies 10 walkers each for stars and gas, so the code will spawn 20 
      processes; a node on TACC has 20 cores)
    - fourth argument specifies an intrinsic dispersion (in km/s); this effectively 
      enforces a minimum value of measurement error associated with each pixel
      in the calculation of &chi;<sup>2</sup>, preventing the code from overfitting 
      regions where the reported measurement error is questionably small

[3] ```simdisk.py```
 - generates initial kinematic models for galactic disks, projected onto the plane of 
   the sky given the spatial orientation of the galaxy in question (position angle of 
   the projected major axis and inclination angle of the disk from the NASA–Sloan 
   Atlas are obtained photometrically and used as initial guesses for kinematic 
   fitting via MCMC)
   
[4] ```dirs.py```
 - specifies paths to four local directories:
 
    - ```datapath``` is the base directory for MaNGA MAPS files; the data file for 
      each galaxy contains 2D fields of velocity and its dispersion for stars and gas 
      (subdirectory structure from SDSS server assumed preserved)
    - ```psfpath``` is the base directory for MaNGA LOGCUBE files, which contain 
      reconstructed PSFs (point spread functions) corresponding to each observation 
      (subdirectory structure from SDSS server assumed preserved)
    - ```modeldir``` is a directory to which initial kinematic models and elements of 
      initial disk geometry are written out in FITS format
    - ```testdir``` is the base directory to which MCMC samples and results are saved 
      (subdirectories for galaxies will be generated by the code)
