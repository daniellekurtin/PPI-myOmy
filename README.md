# PPI: My, oh my
This is a step-by-step guide for how to run a psychophysiological interaction (PPI) using FSL to determine changes in functional connectivity (FC). 

## Background on the study and data: 
This study is investigating the interactions among the parameters of transcranial direct current stimulation (tDCS) and the effects of stimulation on behavior, network activity, and FC. The parameters of stimulation include brain state (either task or rest state), stimulation polarity (anodal, cathodal, or sham stimulation), and WM structure. 
This data consists of two participant groups: traumatic brain injury (TBI) patients and healthy controls (HC). TBI patients are viewed as a model for impaired white matter (WM) structure, since TBI results in axonal injury, thus creating damaged WM tracts in the brain. 

This analysis focuses on changes in FC due to stimulation; specifically, it will us a PPI to determine the changes in FC, and how the parameters of stimulation (brain state, stimulation polarity, and WM structure) influence these changes. 

## Step 1: Where do I begin?  
*Creating the masks*  
Your first step is to determine your regions of interest (ROI) that you'll use as seed regions to interrogate the FC. Since I am interested in changes in FC between nodes in two networks, the salience network (SN) and default mode network (DMN), I use the following four ROIs created from the MNI152 2mm standard template:   

1. SN nodes  
a. dorsal anterior cingulate cortex (dACC)  
b. right inferior frontal gyrus (rIFG_F8)  
2. DMN nodes  
a. posterior cingulate cortex/precuneus (PCCprecu)  
b. ventral medial prefrontal cortex (vmPFC)  

We can see them stored in their own folder as nifti files:  
    
      [dk818@login-4 PPI]$ dk818cd ROIs/  
      [dk818@login-4 PPI]$ ls  
      dACC.nii.gz  PCCprecu.nii.gz  rIFG_F8.nii.gz  vmPFC.nii.gz

## Step 2: Where am I?
*Changing masks to participant space*  
Next, we'll want to change the masks from standard space to participant, or native, space. Make sure the mask and nifti files are in the same dimensions; I put everything in 2mm voxels. Using the same command, you can shift the mask to the participant space, where the participant's clean nifti is your reference, then binarize it to get a clean mask.  
            
       flirt -in dACC.nii.gz -ref /apps/fsl/5.0.10/fsl/data/standard/MNI152_T1_2mm_brain.nii.gz -applyxfm -usesqform -out dACC.nii.gz
       flirt -in dACC.nii.gz -ref CREST1_CRTac1.nii.gz -applyxfm -usesqform -out CREST1_CRTac1_dACC.nii.gz
       CREST1_CRTac1_dACC.nii.gz -bin bin_CREST1_CRTac1.nii.gz
Now you have a mask in the participant's space. Take a look at it and see how your new mask fits vs if you used the standard mask in participant space:  
  
        fslview_depreciated CREST1_CRTac1.nii.gz
 This is particularly easy to see in TBI patients, as their physiology is usually worse than HC. 
 
 ## Step 3: I know where I am. What's going on here?  
 *Extracting timecourses for your ROI*  
 This step allows us to determine the activity over time for the ROI in each participant. You'll first need to navigate out of the clean nifti folder, and into where the first-level FEATs are stored for each participant. Then run fslmeants. 
 
    fslmeants -i CREST1_CRTac1.nii.gz -o CREST1_CRTac1_vmPFC_timecourse.txt -m CREST1_CRTac1_vmPFC.nii.gz

The timecourse.txt is what we'll use in our next step.  

## Step 4: I see what's going here. . . 
*Using the FSL GUI to evaluates changes in FC between your ROI and the whole brain, using the timecourse from the previous step*
