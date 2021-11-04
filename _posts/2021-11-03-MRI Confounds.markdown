---
layout: post
title:  "Confounds to clean fMRI"
date:   2021-11-03 21:11:12 -0400
---
# Using Confounds to Clean fMRI Data

## Means

BOLD Signal in gray matter is interpreted as delayed evidence of neural activity.
But there is no interpretation of BOLD signal in white matter or CSF. So signal
there is more likely to be artifact, scanner noise, physiological confound, etc.
By regressing these nuisance signals out in a GLM, we are likely to remove the noise we
don't care about and leave behind cleaner signal we do.

    csf, white_matter, csf_wm

Global signal can be regressed to normalize the data.

    global_signal

## aCompCors

Anatomical component corrections.

    a_comp_cor_*

## tCompCors

Temporal component corrections.

    tcompcor
    t_comp_cor_*

## DVARS

Volume-to-volume root mean squared change in BOLD signal averaged over all voxels.

    dvars, std_dvars

## Motion Signal

The volume-to-volume motion detected during motion correction is saved as vectors
in the confounds files. The spatial motion has already been reversed by transforming
each volume to match. But the deeper effects of movement on things like spin-history
within the BOLD signal remain. Regressing out signal related to this motion can be
accomplished by including these vectors as nuisance variables in the GLM.

    trans_x, trans_y, trans_z, rot_x, rot_y, rot_z
    framewise_displacement, rmsd

## Motion Outliers

Motion censoring (or "scrubbing") ([Power, et al. 2011](https://dx.doi.org/10.1016%2Fj.neuroimage.2011.10.018)) is using a vector
of 0's, with 1's representing TRs where high motion occurred, in your GLM to
soak up the variance unique to that one TR, removing its effect from your data.
fMRIPrep adds these vectors to the end of the confounds.tsv file.
[Carp, 2011](https://doi.org/10.1016/j.neuroimage.2011.12.061) suggests doing this before bandpass filtering and
[Power, 2012](https://dx.doi.org/10.1016%2Fj.neuroimage.2012.03.017) describes how that may depend on the nature of the motion.

    motion_outlier*

## Also

Band-pass filtering, usually from 0.008-0.01Hz on the low end to 0.08-1.0Hz
on the high end, is generally accepted to remove non-BOLD signals.
[Carp, 2011](https://doi.org/10.1016/j.neuroimage.2011.12.061) filtered 0.009-0.08Hz, then blurred with 6mm FWHM kernel.

[Satterthwaite](https://dx.doi.org/10.1016%2Fj.neuroimage.2012.08.052)
excluded subjects with gross motion > 0.55mm mean relative displacement, leaving only those with <=0.20mm.
They took six motion confound parameters and condensed them into RMS (root mean square) displacement,
then the entire timeseries was condensed into a scalar MRD (mean relative displacement).
