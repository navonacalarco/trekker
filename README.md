__Description__. This repo provides a record of the process I followed to generate "parallel transport tractography" on one example SPINS participant, using `trekker` software [[manual]](https://dmritrekker.github.io/manual/trekker.html) [[citation]](https://www.ismrm.org/19/program_files/O60.htm)

__Note__. I am running on Mac 10.15.3. Trekker does not have a stand-alone executable for Mac, so I installed [linux-noah](https://github.com/linux-noah/noah) and then was provided with a trekker executiable from Dr. Aydogan via [GitHub](https://github.com/dmritrekker/trekker/raw/master/binaries/trekker_linux_x64_v0.5).

-----

Step 1: Copy DWI.nii and FSL-created bvec and bval file from `/archive/data`
```
mkdir trekker; cd trekker
cp /archive/data/SPINS/data/nii/sub-EXAMPLE/*dwi.nii.gz .
cp /archive/data/SPINS/data/nii/sub-EXAMPLE/*bv* .
```

Step 2: Make a mask with MRTrix
```
dwi2mask -fslgrad bvecs bvals sub-EXAMPLE_ses-02_dwi.nii.gz sub-EXAMPLE_mask.nii.gz 
mrview sub-EXAMPLE_ses-02_dwi.nii.gz -roi.load sub-EXAMPLE_mask.nii.gz #not great
```

Step 3: Estimate response function for spherical deconvolution with MRTrix
```
dwi2response tournier sub-EXAMPLE_ses-02_dwi.nii.gz responseFunction.txt -fslgrad bvecs bvals
shview responseFunction.txt
```

Step 4: Perform constrained spherical deconvolution with MRTrix
```
dwi2fod csd sub-EXAMPLE_ses-02_dwi.nii.gz responseFunction.txt sub-EXAMPLE_FOD.nii.gz -mask sub-EXAMPLE_mask.nii.gz -fslgrad bvecs bvals
mrview sub-EXAMPLE_ses-02_dwi.nii.gz -odf.load_sh sub-EXAMPLE_FOD.nii.gz
```

Step 5: Compute whole-brain streamlines tractography with MRTrix (iFOD2, probabilistic), for comparative purposes
```
tckgen sub-EXAMPLE_FOD.nii.gz sub-EXAMPLE_tractography_MRTrixStreamlines_50000.tck -seed_image sub-EXAMPLE_mask.nii.gz -mask sub-EXAMPLE_mask.nii.gz -select 50000
mrview sub-EXAMPLE_ses-02_dwi.nii.gz -tractography.load sub-EXAMPLE_tractography_MRTrixStreamlines_50000.tck
```

Step 5: Compute 'parallel transport tractography' with Trekker (enabled by noah)
```
noah trekker_linux_x64_v0.5 \
  -fod sub-EXAMPLE_FOD.nii.gz \
  -seed_image sub-EXAMPLE_mask.nii.gz \
  -seed_count 1000 \
  -output sub-EXAMPLE_tractography_trekkerPTT.vtk
```
