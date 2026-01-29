# flyTomo.cfg Tutorial

This document provides a detailed guide to configuring the `flytomo.cfg` file for the flyTomo pipeline. Each section corresponds to a processing step or a set of related parameters.

---

## `[pipeline]`

This section defines the sequence of processing steps that flyTomo will execute.

- **1 = motioncor2**: Motion correction for raw movie frames.
- **2 = ctf**: CTF estimation for the motion-corrected micrographs.
- **3 = doseweighting**: Applies dose weighting to the micrographs.
- **4 = tsalignment**: Tilt-series alignment.
- **5 = reconstruction**: Reconstructs the tomogram.

---

## `[cryoem.para]`

This section contains fundamental parameters related to your microscope and imaging setup. **It is critical to set these values correctly for your dataset.**

- **CS**: Spherical aberration of the microscope, in millimeters (mm).
  - *Example*: `2.7`
- **tilt_axis_angle**: The angle of the tilt-axis in degrees. This value is crucial for accurate reconstruction.
  - *Example*: `-95.5`
- **kV**: The high tension (accelerating voltage) of the microscope, in kilovolts (kV).
  - *Example*: `300`
- **Pixel_size**: The pixel size of the original movie frames *before* any binning, in Angstroms (Å).
  - **Common Pitfall**: Ensure this is the pixel size of the raw detector image, not a binned image.
  - *Example*: `1.907`

---

## `[acceleration.options]`

These settings control performance and resource usage.

- **tmpdir**: A directory for temporary files. Using a RAM disk like `/dev/shm` can significantly speed up I/O operations and reduce I/O for numerous small files.
  - *Recommended*: `/dev/shm` if available and has enough space.
- **tmpdir_enable**: `1` to enable using the `tmpdir`, `0` to disable.
- **compress_movies_enable**: `1` to enable movie compression, `0` to disable. This can save disk space but adds computational overhead. Recommended for movies in MRC format; not essential for other formats.
- **Threads**: The maximum number of CPU threads to use for parallel processing tasks.
  - *Recommended*: Set to the number of available CPU cores / 2, or slightly less to leave resources for other system tasks.

---

## `[autoprocessing.options]`

Options for automated data quality control and pre-processing.

- **auto_detect_dark_image_threshold**: A threshold to automatically detect unusually dark images, which may indicate a blocked beam or other issues.
  - *Usage*: A value between 0 and 1. A common starting value is `0.75`.
- **auto_delete_dark_image_enable**: `1` to automatically delete images flagged as dark, `0` to keep them.
- **auto_estimate_pretilt_angle**: `1` to enable automatic estimation of the pre-tilt of the lamella.
- **auto_correct_pretilt_enable**: `1` to apply the estimated pre-tilt correction.
  - **Recommendation**: Enable this (`1`) for FIB-milled lamella samples. For purified samples, it's often better to disable this (`0`) as there is no lamella pre-tilt.
- **pretilt**: Manually set the pre-tilt angle in degrees if automatic estimation is disabled.

---

## `[motioncor.programpara]`

Parameters for the MotionCor2 program.

- **Environment**: A shell command to set up the environment for MotionCor2 (e.g., loading required modules or sourcing a script).
  - **Tips: Design Philosophy**: Each program's `Environment` can be set independently. This allows flyTomo to isolate different software environments at runtime, preventing conflicts between programs that have different dependencies.
- **Program**: The absolute path to the MotionCor2 executable.
- **Iter**: Maximum number of iterations for alignment.
- **Patch**: The number of patches to divide the frame into for local motion estimation (Y X).
  - *Recommended*: `3 3` or `5 5`. More patches can help with highly variable motion but increases computation time.
- **bft**: B-Factor for weighting frames, in A^2.
  - *Recommended*: `100`.
- **Tol**: Tolerance for motion trajectory fitting.
  - *Recommended*: `0.5`.
- **ftbin**: Binning factor for the frames before motion correction.
  - **Usage**: Set to `2` if the movies were collected in super-resolution mode. Otherwise, set to `1`.
- **Group**: Group frames for alignment. `1 3` means grouping every 3 frames starting from frame 1.
- **FmRef**: Use a specific frame as the reference for alignment.
- **Throw**: Discard the first N frames.
- **Gain**: Path to the gain reference file for the detector.
- **SplitSum**: `1` to save even and odd frame summed stack file separately (useful for some post-processing), `0` to save a single summed stack file.
- **FmIntFile**: Path to a text file required when using EER format data. This file specifies the number of frames for each tilt.

## `[motioncor.runtimepara]`

Runtime settings for MotionCor2.

- **GPUlist**: A comma-separated list of GPU device IDs to use.frame, which can be useful for Relion 5 programs. `0` to disable
- **save_avg_file**: `1` to save the average (non-dose-weighted) stack, which be useful for Relion 5 programs.
- **save_tmp_file**: `1` to keep temporary files, `0` to delete them. Useful for debugging.

---

## `[ctf.programpara]`

Parameters for the GCTF program for CTF estimation.

- **Environment**: Shell command to set up the GCTF environment.
- **Program**: Absolute path to the GCTF executable.
- **ac**: Amplitude contrast.
  - *Recommended*: `0.1`.
- **defL` / `defH` / `defS`**: Defocus search range: Low (Angstroms), High (Angstroms), and Step (Angstroms).
- **astm**: Maximum astigmatism to search for (in Angstroms).
- **bfac**: B-Factor for CTF fitting.
- **resL` / `resH`**: Resolution search range: Low (Å) and High (Å).
- **boxsize**: Box size for CTF estimation from the power spectrum.
- **do_EPA**: `1` to enable Estimated Phase Aberration correction.

## `[ctf.runtimepara]`

- **GPUlist**: Comma-separated list of GPU device IDs for GCTF.

---

## `[doseweighting.programpara]`

Parameters for dose weighting.

- **dose_per_tilt**: The electron dose for each tilt image, in e-/Å².
  - **Critical**: This value must be accurate.
- **pre_dose**: The dose accumulated on the sample before the start of the tilt series acquisition (e.g., for focusing).

## `[doseweighting.runtimepara]`

- **doseweighting_enable**: `1` to enable this step, which generates a dose-weighted stack (`dw.st`). `0` to skip.
- **Threads**: Number of CPU threads to use.

---

## `[tsalignment.programpara]`

Parameters for tilt-series alignment, primarily using AreTomo.

- **fiducial_size**: The size of gold fiducial markers in nanometers (nm), if used.
- **TargetNumberOfBeads**: The desired number of fiducials to track.
- **patch_size` / `patch_overlap_percentage`**: Parameters for patch tracking alignment.
- **Environment.batchruntomo` / `Program.batchruntomo`**: Environment and path for IMOD's `batchruntomo`.
- **Environment.aretomo` / `Program.aretomo`**: Environment and path for AreTomo.
- **Thickness**: The thickness of the final reconstructed tomogram in pixels (Bin 1).
- **SampleThickness**: An estimate of the sample thickness in pixels, used for marker-free alignment.
- **sample_thickness_detection_enable**:
  - `1`: Enable automatic sample thickness detection.
  - `2`: Enable detection with an alternative (sd) method.
  - `0`: Disable.
  - **Recommended**: `1` or `2` for FIB-milled lamella samples; `0` for purified samples.
  - **Rationale**: When ice contamination is present, the actual sample thickness may deviate significantly from the user-provided `SampleThickness`. 

- **sample_thickness_detection_threshold**: Threshold for detecting ice contamination based on thickness variation. If the automatically detected sample thickness (`Sample_thicknesss`) differs from the user-provided `SampleThickness` by more than this ratio (`(Sample_thicknesss - SampleThickness) / SampleThickness > threshold`), the tomogram is considered potential ice contamination.

## `[tsalignment.runtimepara]`

- **align_method**: The alignment method to use.
  - `fiducials`: Gold bead-based alignment.
  - `marker_free`: Alignment without fiducials.
  - `patch_tracking`: Patch-based alignment.
  - **Common Pitfall**: Ensure this matches your data type. Using `fiducials` on marker-free data will fail.
- **GPUlist**: GPU devices for alignment.
- **Threads**: CPU threads for alignment.

---

## `[tomogramprocess.programpara]`

Parameters for various post-reconstruction processing steps like deconvolution, segmentation, and particle picking.

- **Environment/Program paths**: Set up environments and executables for `novaCTF`, `Isonet`, `membraneseg`, and `pytom-tm`.
- **pytomtm...**: Parameters for template matching with `pytom`.
- **defocus_step**: Defocus step size in nm for novaCTF.
- **bin**: Binning factor for the reconstruction. This value is passed to novaCTF.
- **need_erase_goldbeads**: `1` to erase gold beads from the tomogram.
- **zshift**: Apply a Z-shift to the final tomogram.
- **tilt_angle_offset**: Apply an offset to the tilt angles.
- **Isonet.ModelFile**: Path to the pre-trained Isonet model for denoising.
- **Membraneseg.ModelFile**: Path to the pre-trained model for membrane segmentation.
- **NovaCTF.parameterfile**: Path to the parameter file for novaCTF.

## `[tomogramprocess.runtimepara]`

- **GPUlist**: GPUs for these processing steps.
- **Threads**: CPU threads.

---

## `[sta.programpara]` & `[sta.runtimepara]`

Parameters for sub-tomogram averaging (STA) using Dynamo.

- **Environment.dynamo` / `Program.dynamo`**: Environment and path for Dynamo.
- **GPUlist` / `Threads`**: Resource allocation for STA.
