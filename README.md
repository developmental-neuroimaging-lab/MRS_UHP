# MRS_UHP
Processing scripts for magnetic resonance spectroscopy data acquired on UHP scanner (post-upgrade 2024) \
Created by Meaghan Perdue, May 2024

This repository contains processing code for semi-laser (sLASER) and HERMES MRS data. Start-to-finish processing uses Osprey <https://schorschinho.github.io/osprey/> via Matlab.

## Software Requirements 

**Matlab** 2017a or newer with GUI Layout Toolbox and Widget Toolbox \
**Osprey:** <https://github.com/schorschinho/osprey> \
**SPM12:**  <https://www.fil.ion.ucl.ac.uk/spm/docs/installation/> \
See Osprey documentation for additional dependencies.

** IMPORTANT ** As of May 2024, there is a bug in Osprey generating HTML output, if . To correct bug, add the following to osprey/plot/OspreyHTMLReport.m, lines 1015 and 1257: \
``` isfield(MRSCont.seg,'img') && ``` \
e.g., line 1015: \
``` if MRSCont.flags.didSeg && isfield(MRSCont.seg,'img') && isfield(MRSCont.seg.img, 'vol_Tha_CoM') % HBCD thalamus overlap ```

## Data Organization
MRS raw data (P-files, end in .7) and T1w anatomical data must be organized according to BIDS.

## Processing scripts
Osprey job files for processing are saved in the 'src' directory and named by study. Job files are currently available for the following studies: \
PDP
** Note that the Osprey job file assumes the T1w image is named sub-NNN_ses-NNN_T1w.nii.gz file, may need to edit the job file to specify which to use if multiple are available (pure/not, multiple runs), or if it is saved as .nii instead of .nii.gz

To run an Osprey job file via Matlab command line:
``` MRSCont = RunOspreyJob('path/to/job/file) ```

To run the job via Matlab from the terminal (without the GUI):
``` 
cd /Applications/MATLAB_R*.app
./matlab -nodesktop
MRSCont = RunOspreyJob('path/to/job/file)
```
** Osprey jobs are set up to run in batches. Group data outputs (compiled stat files,etc.) will be overwritten if a new batch is run. Rename the 'output' directory in the job file to write results to a new folder **

## Study-Specific information
### PDP
The 3y and 4y PDP protocols include 1 MRS acquisition with voxel location in the Anterior Cingulate Cortex (ACC). This is an sLASER sequence (no spectral editing) which provides measures of tNAA, tCr, tCho, Glx and mI (NAA/NAAG and Glu/Gln may be separable, will need to check data quality and reliablity). 
sLASER acquisition details: TE=30ms, TR=2000ms, voxel size = 20 x 20 x 15 mm, 96 averages \
Edit contents of 'sublist' in PDP_job.m line 205 to include the subjects you wish to process.
Sessions to be processed are listed in the loop below the subject list setup and currently includes ses-03y and ses-04y. The contents of 'seslist' can be changed to specify or add new sessions. 

## Outputs and Quality Check
Osprey outputs for each study will be saved to the study BIDS directory under derivatives/osprey. \ 
For QC, view summarized html outputs under "Reports", check the **Averaged Spectra** and **Model Results** for spectral artifacts, check **Coregistration & Segmentation** for voxel location and tissue segmentation accuracy. Individual PDF outputs from each processing step can be opened from "Figures" for detailed inspection.

**Quality Metrics** including SNR & linewidth of the Creatine peak and linewidth of the water peak are output to QM_processed_spectra.tsv

derivatives/FilesSPM outputs contain intermediate SPM files from coregistration/segmentation, these outputs can be deleted after MRS processing. 