# MRS_UHP
Processing scripts for magnetic resonance spectroscopy data acquired on UHP scanner (post-upgrade 2024) \
Created by Meaghan Perdue, May 2024

This repository contains processing code for semi-laser (sLASER) MRS data. Start-to-finish processing uses Osprey <https://schorschinho.github.io/osprey/> via Matlab.

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
MRS raw data (P-files, end in .7) and T1w anatomical data must be organized according to BIDS. \
MRS data acquisition from 2024- is tracked on the Lebel Lab OneDrive under Lab Management: <https://uofc-my.sharepoint.com/:x:/r/personal/clebel_ucalgary_ca/Documents/Lebel%20Lab/Lab%20Management/SPEC%20Tracking/MRS_data_tracking.xlsx?d=wd314a7e09f824aec820441ae346d1d08&csf=1&web=1&e=98f7Za> \
Scripts called 'mrs_to_bids.sh' are available under study-specific folders in /Volumes/catherine_team/MRI_Data/bids_conversion to pull and organize the P-files. \
The info in the tracking spreadsheet can be used to create a subject list for running the mrs_to_bids.sh scripts. This requires subject ID, session ID, study code, scan ID, and series numbers for each MRS voxel.

## Processing scripts
Osprey job files for processing are saved in the 'src' directory and named by study. Job files are currently available for the following studies: \
    PDP \
    PEACH (separate job files for each voxel: ACC, Thal, PWM) 

Things to update in the job files: 
1. List of metabolite inputs (P-files)
2. List of T1w anatomical inputs (must match order of p-file list)
3. Output directory (these will be overwritten with each run, so name them with detail)
** Note that the Osprey job file assumes the T1w image is named sub-NNN_ses-NNN_T1w.nii.gz file, may need to edit the job file to specify which to use if multiple are available (pure/not, multiple runs), or if it is saved as .nii instead of .nii.gz

This info can be entered either to loop over data in BIDS format or by listing full paths to P-files and T1w files. I recommend the full paths to avoid errors with missing data, etc. 

To run an Osprey job file via Matlab command line:
``` MRSCont = RunOspreyJob('path/to/job/file') ```

To run the job via Matlab from the terminal (without the GUI):
``` 
cd /Applications/MATLAB_R*.app
./matlab -nodesktop
MRSCont = RunOspreyJob('path/to/job/file)
```
** Osprey jobs are set up to run in batches. Group data outputs (compiled stat files,etc.) will be overwritten if a new batch is run. Rename the 'output' directory in the job file to write results to a new folder **

## Study-Specific information
### PDP
The 3y and 4y PDP protocols include 1 MRS acquisition with voxel location in the Anterior Cingulate Cortex (ACC). This is an sLASER sequence which provides measures of tNAA, tCr, tCho, Glx and mI (NAA/NAAG and Glu/Gln may be separable, will need to check data quality and reliablity). 
sLASER acquisition details: TE=30ms, TR=2000ms, voxel size = 20 x 20 x 15 mm, 96 averages \
To run loop using BIDS format, edit contents of 'sublist' in PDP_job.m line 205 to include the subjects you wish to process.
Sessions to be processed are listed in the loop below the subject list setup and currently includes ses-03y and ses-04y. The contents of 'seslist' can be changed to specify or add new sessions. 

### PEACH
As of spring 2024, the PEACH protocol includes 3 MRS voxels: Anterior Cingulate Cortex (ACC), left Thalamus (Thal), and left Parietal White Matter (PWM) using sLASER. \
sLASER acquisition details: TE=30ms, TR=2000ms, voxel size = 20 x 20 x 15 mm, 96 averages \
There are separate job files for each voxel

### Bioenergetics (BIOE)
Pilot study protocol includes 3 MRS voxels: midline Anterior Cingulate Cortex (ACC), left Thalamus (Thal), and midline Posterior Cingulate Cortex/Precuneus (PCC) \
short TE sLASER acquisition details ACC and Thal: TE=30ms, TR=2000ms, voxel size = 20 x 20 x 15 mm, 96 averages \
short TE sLASER acquisition details PCC: TE=30ms, TR=2000ms, voxel size = 30 x 30 x30 mm, 96 averages \
long TE sLASER acquisition details PCC (for Lactate): TE=288ms, TR=2100ms, voxel size = 30 x 30 x 30 mm, 96 averages \

## Outputs and Quality Check
Osprey outputs for each study will be saved to the study BIDS directory under derivatives/osprey. \ 
For QC, view summarized html outputs under "Reports", check the **Averaged Spectra** and **Model Results** for spectral artifacts, check **Coregistration & Segmentation** for voxel location and tissue segmentation accuracy. Individual PDF outputs from each processing step can be opened from "Figures" for detailed inspection.

**Osprey QuantifyResults outputs:**  include A_TissCorrWaterScaled... outputs with tissue correction implemented according to the Harris method. Note that the .tsv files do *not* include columns with subject/session IDs so this info needs to be added based on the *order of subjects enetered in the processing scripts* (create a .tsv or .csv file with the subject/session IDs and use a join function to create these)

**Quality Metrics** including SNR & linewidth of the Creatine peak and linewidth of the water peak are output to QM_processed_spectra.tsv

derivatives/FilesSPM outputs contain intermediate SPM files from coregistration/segmentation, these outputs can be deleted after MRS processing. \
Osprey also outputs segmented T1 images (GM, WM and CSF, labelled c1..., c2..., c3...) in the folder with the original T1w scan (subject/session/anat folder in BIDS). These can be removed after MRS processing.

