# chromatic-seeing

The purpose of this package is to study the wavelength dependence of atmospheric seeing predicted in two ways:  (1) numerical computations of analytic formulae that are valid for infinite exposure time, and (2) simulations corresponding to finite exposure time. In both cases, the predicted PSF is based on a von Karman phase power spectrum, which is modified from the Kolmogorov spectrum to take into account the non-infinite outer scale — i.e., it limits the power at large scales. The simulation, which is implemented in GalSim, models the 3-dimensional turbulence in the atmosphere as a series of 2-dimensional phase screens at different altitudes, through which the LSST pupil can be projected, and the instantaneous PSF derived via Fourier optics. The screens independently drift with the wind in each layer, and the net PSF for a finite exposure time is accumulated in steps. For more details on GalSim, see https://github.com/GalSim-developers/GalSim

Original use of these programs was to compare the GalSim simulations generated by generate_simulation_psf.py to the theoretical PSFs generated by generate_theory_psf.py. Specifically, comparison was made between the full width at half maximum (FWHM) of the theoretical profiles and averaged profiles derived from the GalSim-generated images. Details can be found in the presentation linked below. 

Link to presentation
https://docs.google.com/presentation/d/1dzB7xHbTjP1Q4Crbw64JSBjJHxzfnBSHlbU9miCT6WY/edit?usp=sharing

The study shows a definite departure from the Kolmogorov prediction for the wavelength dependence of the PSF size (FWHM proportional to <a href="http://www.codecogs.com/eqnedit.php?latex=\lambda^{-1/5}" target="_blank"><img src="http://latex.codecogs.com/gif.latex?\lambda^{-1/5}" title="\lambda^{-1/5}" /></a>) that depends on the outer scale.  

This analysis and many of the graphs found in the presentation were done using modifications of psf_image_analysis.py run on simulations generated with the two other scripts, given relevant input parameters. For example, running psf_image_analysis.py on the provided sample images in the sample_images folder and theoretical profiles in the theory_PSFs folder produces the plot FWHM vs. Wavelength.png
___________________________________________________________________________________________________________________________
___________________________________________________________________________________________________________________________

Each script is described briefly below:




generate_theory_psf.py generates a thoeretically calculated PSF profile for the von Karman spectrum at a given outer scale, wavelength, Fried parameter (specified at 500nm), and telescope diameter. The PSF profile is output as a .txt file containing a list of the (arbitrarily renormalized) expected intensity of the azimuthally symmetric PSF at points along a radius, with the first entry on the list corresponding to the center. By default, the samples are taken in .005" increments up to a maximum of 1/2 of the theoretical FWHM of a PSF for the Kolmogorov model at 300nm. Alternatively, a user-provided list of points can be sampled. Defualt parameters are recorded in the dict 'args_def' and can be changed at command line, 
e.g.

generate_theory_psf.py --r0_500 0.35  --lam 400.0 --diam 7.5  --L0 20.0  --alphas [0.0,.005,.05,.1]

Many example theoretical PSF text files generated with this script can be found in the theory_PSFs folder. The name of the file records all of the parameters used to create that profile, with the point sampling done with the default setting. 
(NOTE: this default sampling of points matches the analysis done in psf_image_analysis.py, so if using the two programs in tandem with custom sample points, the theory_alphas variable in psf_image_analysis.py will need to be changed accordingly)

___________________________________________________________________________________________________________________________

generate_simulation_psf.py is a modification of a GalSim demo that generates simulated PSF images and incorperates many user-tunable parameters, including outer scale. These images are notably NOT explicitly generated using the theoretical expressions for expected intensity that are used in generate_theory_psf.py; rather, these images are created in GalSim which more directly simulates a specific turbulence profile being evolved in time for some specified image exposure time--see https://github.com/GalSim-developers/GalSim for details. In particular, one might expect images generated with short exposure times in GalSim to differ significantly from the idealized long-exposure limit PSF profiles predicted by the theoretical expressions in generate_theory_psf.py. Just as in generate_theory_psf.py, the default parameters are recorded in the dict 'args_def' at the top of the program and each can be changed at command line. The PSF is saved as a .fits file with the title recording all of the non-default parameters that the user specfied to create the image. Many example simulated PSFs generated with this script can be found in the sample_images folder. You may need to download a program such as that found at http://ds9.si.edu/site/Download.html in order to directly view the images.

___________________________________________________________________________________________________________________________

psf_image_analysis.py uses the files generated by generate_theory_psf.py and generate_simulation_psf.py to analyze the respective PSFs, compare the FWHM of each generation method, and produce plots (e.g. FWHM vs. Wavelength.png). The script utilizes the file naming convention used in the generation of the files in the other scripts to keep track of relevant input parameters. 

In order to extract a FWHM from the 'noisy' GalSim-generated images which can be meaningfully compared to the smooth, azimuthally symmetric theoretical profiles, the script has a method generate_monte_radial_function which takes in the simulated image data and a list of sample points which extend radially from some user-specified center point (I used first moments). It then performs a monte-carlo averaging of the PSF intensity around circles at the specified sample-point radius in order to convert the 'noisy' 2d PSF image into azimuthally averaged radial PSF. This conversion takes some time, so the porgram automatically maintains a saved_radial_profiles.txt file which stores the radial profiles after calculating them and on later runs checks this file before performing new calculations. 

The intended worflow when using these three programs in tandem is as follows: 

1) identify relevant parameter ranges for outer scale and wavelength
2) run generate_theory_psf.py for each pair of these parameters to create a theory psf file for each
3) run generate_simulation_psf.py for each pair of parameters, taking care to match the diameter and Fried parameter used for the theoretical PSFs. There are still many more parameters for the simulation (e.g. exposure time) that may affect the comparison with theory. 
4) among all of the PSFs simulated thus far, specify which range of parameters you wish to analyze by updating the ranged_args dict in the main of psf_image_analysis.py, and specifying any individual non-default parameter that is the same accross all simulations by altering the relevant value in the args dict. The program will then generate or retrieve a radial profile for each simulation of the given parameters, which can then be compared and plotted against the corresponding theoretical prediction


_________________________________________________________________________________________________________________________________________
_________________________________________________________________________________________________________________________________________

This work was conducted by Kyle Gulshen, in collaboration with Pat Burchat and Josh Meyers, at Stanford University in 2016/2017. Work was supported by the Office of the Vice Provost for Undergraduate Education at Stanford and by the National Science Foundation. 
