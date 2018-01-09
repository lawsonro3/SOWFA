# Overview
The files in this repository comprise the current version of SOWFA (Simulator for
Offshore Wind Farm Applications), created at the National Renewable Energy
Laboratory (NREL).  The files are based on the OpenFOAM software and are
either new files or modifications of files in the OpenFOAM source code
distribution. Please see the included OpenFOAM readme file
("README.OpenFOAM") and the GPL licence information ("COPYING"). Access
to and use of SOWPA imposes obligations on the user, as set forth in the
NWTC Design Codes DATA USE DISCLAIMER AGREEMENT that can be found at
<https://nwtc.nrel.gov/disclaimer>.

# Solvers and Codes Included
## Solvers
 * ABLSolver - A  solver, primarily for performing large-eddy simulation
   (LES), for computing atmospheric boundary layer turbulent flow with
   the ability to specify surface roughness, stability, and wind speed
   and direction. It must be used with hexahedral meshes (like those
   created by OpenFOAM's blockMesh utility) in order for it to perform
   the planar averaging it does.
 * ABLTerrainSolver - Basically the same as ABLSolver, but the planar
   averaging done in ABLSolver is replaced with time averaging since
   planar averaging does not make sense for terrain.
 * windPlantSolver.<X> - A specialized version of ABLTerrainSolver for
   performing LES of wind plant flow.  It includes the ability to
   include actuator line turbine models with local grid refinement
   around the turbine. The <X> corresponds to the turbine type and is
   "ALM", "ADM", or "ALMAdvanced".
 * pisoFoamTurbine.<X> - OpenFOAM's standard pisoFoam solver, but with the
   actuator turbine models included.  This is useful for simple cases, or
   for cases where atmospheric effects are not important, like turbines
   in wind tunnels.
 * turbineTestHarness.<X> - A simple code coupled to the actuator line model
   that allows you to quickly test a actuator line model setup.  It mimics
   pisoFoam, but the governing equations are not actually solved, the
   velocity field is constant through the domain, but can change as a
   function of time.  Therefore, no axial induction is computed.  But, it
   lets one quickly test out a new actuator line model setup to see if
   things like pitch control gains are appropriate.

> NOTE:  These solvers have been primarily tested for flow over flat terrain.
We have tested the non-flat terrain capabilities less, but will do so in the
future.  It is important to remember, though, that you may couple the
actuator line models with any standard OpenFOAM solver, such as pisoFoam.



## Utilities
 * setFieldsABL - A utility to initialize the flow field for performing
   atmospheric boundary layer LES.  With the utility, you can specify
   an initial mean profile and perturbations that accelerate the
   development of turbulence will be superimposed.  You may also
   specify the initial temperature profile and location and strength
   of the capping inversion.
   
##  Libraries
1. finiteVolume - Contains custom boundary conditions for:
 * surface shear stress - Schumann model
 * surface temperature flux / heating - Specify a surface cooling/heating
 rate and the appropriate temperature flux is calculated
 * surface velocity - For use with surface shear stress model, which
 requires no wall-parallel velocity, but that velocity   is required for
 specification of the gradient for the SGS model, and setting it to
 no-slip is not appropriate for rough walls
 * inflow velocity - an inflow condition that applies a log-law with
 fluctuations and drives flow to a certain speed at a specified location
 * inflow temperature - an inflow temperature condition that attempts to
 recreate a typical ABL potential temperature profile
 * time varying mapped fixed value with organized random perturbations.
 Useful for taking inflow from a mesoscale weather model and applying
 temperature perturbations to create resolved-scale turbulence.
2. incompressible LES models:
     * a modified version of OpenFOAM standard Smagorinsky model with Pr_t
   sensitization to atmospheric stability.
     * Deardorff-Lilly one-equation model.
     * Kosovic nonlinear backscatter anisotropy one-equation model.
     * a modified version of OpenFOAM standard dynamic Lagrangian model of
       Meneveau et al. but  that writes out the Cs field.  Also, contains a
       modified version of that same model that clips the Cs field.
3. turbineModelsStandard - Contains the actuator line/disk turbine
   models similar to that outlined by Sorensen and Shen (2002).
4. turbineModelsFASTv8 - Actuator line model with coupling to FAST 8
   (see http://wind.nrel.gov/designcodes/simulators/fast/) meant for
   coupling with the windPlantSolver.ALMAdvancedFASTv8 solver.
5. postProcessing - Function objects to simulate the sampling patterns
   of scanning lidar.  Useful for simulating lidar to understand its
   capabilities and limitations.
6. fileFormats - Adds a structured VTK file format that cuts down
   on file size by greatly eliminating the write out of x,y,z points.
7. sampling - Adds a sampling set that defines an annulus.  Useful
   for sampling annuli in rotor plane to see blade-local flow.

# Installation
A. The included codes work only with the OpenFOAM CFD Toolbox.  OpenFOAM has
not been distributed with the SOWFA package.  Please visit www.openfoam.com
to download and install OpenFOAM.  This release of SOWFA is known to work
with up to OpenFOAM-2.4.x.  Making SOWFA work with versions 3.0 and higher
may come in the near future.


# Downloading and Compiling
1.  Make sure that you have OpenFOAM version `OpenFOAM-2.4.x` downloaded and
    compiled on your system.
2.  Use [Git](https://git-scm.com/) to clone SOWFA from the [https://github.com/NREL/SOWFA](SOWFA GitHub
    repository). We recommend that you clone the SOWFA repository into your
    OpenFOAM `$WM_PROJECT_USER_DIR` directory. For OpenFOAM version
    `OpenFOAM-2.4.x` this would be typically be
    `~/user-name/OpenFOAM/user-name-2.4.x`. The Git command to clone SOWFA is:
    ```
    [/home/OpenFOAM/user-name-2.4.x]$ git clone https://github.com/NREL/SOWFA.git
    ```
3.  Move into the `SOWFA` directory and execute `Allwclean` and `Allwmake`:
```
[/home/OpenFOAM/user-name-2.4.x]$ cd SOWFA
[/home/OpenFOAM/user-name-2.4.x/SOWFA]$ ./Allwclean
[/home/OpenFOAM/user-name-2.4.x/SOWFA]$ ./Allwmake
```
4.  Make sure that no error messages appeared and that all libraries
    and applications are listed as "up to date."

# Tutorials
Tutorial example cases are provide for each solver in the `SOWFA/exampleCases` folder. The available tutorials are
as follows:
1. Example cases for computing flat-terrain laterally periodic precursor
  atmospheric large-eddy simulations under the full range of stability.
 * `example.ABL.flatTerrain.neutral`
 * `example.ABL.flatTerrain.unstable`
 * `example.ABL.flatTerrain.stable`
 * Uses: `ABLSolver`

2. Example cases that use the actuator turbine models using the
  windPlantSolver.<X> solvers.  These cases assume that a precursor
  case has been run to generate inflow velocity and temperature
  data.  Because the inflow data is large, we did not include it here.
 * `example.ALM`
 * `example.ALMAdvanced`
 * `example.ADM`
 * Uses:
     * `windPlantSolver.ALM`
     * `windPlantSolver.ADM`
     * `windPlantSolver.ALMAdvanced`
3. A case with the setup of the NREL Unsteady Aerodynamics Experiment
  Phase VI, in the NASA Ames 80'x120' wind tunnel.
 * `example.UAE_PhaseVI.ALMAdvanced`
 * Uses: `pisoFoamTurbine.ALMAdvanced`

5. An example of using mesoscale influence to drive the atmospheric
 boundary layer LES.  This case is driven by WRF model output
 for the DOE/Sandia Scaled Wind Farm Technology (SWiFT) site in
 Lubbock, Texas.  The terrain is flat with laterally periodic
 boundaries in this example.  The case has enough WRF output
 contained in the "forcing" directory to simulate two diurnal
 cycles starting November 11, 2013 at 00:00:00 UTC.
 * `example.mesoscaleInfluence.SWiFTsiteLubbock.11Nov2013Diurnal`

## Running Tutorials
* To run a tutorial, change to that tutorial directory and run the
`runscript.preprocess` script to set up the mesh, etc.  Then run the
`runscript.solve.1` script to run the solver.
* These are basic tutorials meant to familiarize the user with the
general file structure of a case and the various input files.  They
can be run on a small amount of processors with coarse meshes, but if
that is done, they will generate poor results.
