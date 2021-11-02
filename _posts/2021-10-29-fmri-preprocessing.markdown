---
layout: post
title:  "fMRI Preprocessing"
date:   2021-10-29 21:11:12 -0400
categories: fmri preprocessing
---
Many steps are available to clean functional MRI data prior to analysis. All are technically optional. Some are highly recommended. And there are many tools and methods to accomplish the same thing. This table is designed to help figure out what has already been done (and shouldn't be done twice), what hasn't, and how to design pipelines to get you from acquisition to actual analyses.

<table id="pre-proc-fmri">
    <tr class="top-title">
        <th style="width: 25%;">
            <a href="https://fmriprep.org/en/latest/workflows.html">fMRIPrep</a><br />
            v20.2.0
        </th>
        <th style="width: 25%;">
            <a href="https://dx.doi.org/10.1016/j.neuroimage.2013.04.127">HCP Original</a><br />
            v?
        </th>
        <th style="width: 25%;">
            <a href="https://github.com/DCAN-Labs/abcd-hcp-pipeline">ABCD-DCAN</a><br />
            v0.0.4?
        </th>
        <th style="width: 25%;">
            <a href="https://fsl.fmrib.ox.ac.uk/fsl/fslwiki/FEAT">FSL Feat</a><br />
            v6.00
        </th>
    </tr>

    <tr class="step-title">
        <td colspan="4"><p>MR gradient nonlinearity induced distortions</p></td>
    </tr>
    <tr class="step-items">
        <td></td>
        <td>gradient_nonlin_unwarp</td>
        <td></td>
        <td></td>
    </tr>
    <tr class="step-notes">
        <td colspan="4">
            <p>The bore of Siemens magnets have greater distortion than GE
                magnets, but none are completely consistent. The difference in
                magnetic field between the center and the periphery can be
                corrected with a gradient coefficient file from the scanner.
                This is less important with GE scanners, particular with
                centered participant placement.
            </p>
        </td>
    </tr>

    <tr class="step-title">
        <td colspan="4"><p>Motion Correction</p></td>
    </tr>
    <tr class="step-items">
        <td>mcflirt (6 DOF)</td>
        <td>6 DOF FLIRT to single-band or slice 0</td>
        <td></td>
        <td></td>
    </tr>
    <tr class="step-notes">
        <td colspan="4">
            <p>All fMRI acquisitions contain multiple volumes over time,
               each volume called a "TR" and usually taking around 2 seconds.
               Because the participant may move slightly over time, each volume
               is registered to the same space as a reference. The reference
               could be the first volume, the middle volume, the average of all
               volumes, or even the T1w space. These are all reasonable choices
               because the translation between them is linear. Good registrations
               to template spaces, like MNI152, would be nonlinear and overkill
               for motion correction.
            </p>
        </td>
    </tr>

    <tr class="step-title">
        <td colspan="4"><p>Slice timing correction</p></td>
    </tr>
    <tr class="step-items">
        <td>turned off for multi-band with --ignore-slicetiming<br />
            normally, uses AFNI's 3dTShift
        </td>
        <td></td>
        <td></td>
        <td></td>
    </tr>
    <tr class="step-notes">
        <td colspan="4">
            <p>
            </p>
        </td>
    </tr>

    <tr class="step-title">
        <td colspan="4"><p>Susceptibility Distortion Correction (SDC)</p></td>
    </tr>
    <tr class="step-items">
        <td>was PEPOLAR, is now SDCFlows TOPUP</td>
        <td>FSL topup*<br/>
            * SDC transform combined with BBR-based transform to other spaces for one-step transforms.
        </td>
        <td></td>
        <td></td>
    </tr>
    <tr class="step-notes">
        <td colspan="4">
            <p>
            </p>
        </td>
    </tr>

    <tr class="step-title">
        <td colspan="4"><p>B1 intensity bias removal</p></td>
    </tr>
    <tr class="step-items">
        <td></td>
        <td>approximate from structural image</td>
        <td></td>
        <td></td>
    </tr>
    <tr class="step-notes">
        <td colspan="4">
            <p>
            </p>
        </td>
    </tr>

    <tr class="step-title">
        <td colspan="4"><p>Brain-mask BOLD frames</p></td>
    </tr>
    <tr class="step-items">
        <td>init_bold_reference_wf</td>
        <td>use PostFreeSurfer mask</td>
        <td></td>
        <td></td>
    </tr>
    <tr class="step-notes">
        <td colspan="4">
            <p>
            </p>
        </td>
    </tr>

    <tr class="step-title">
        <td colspan="4"><p>Normalization</p></td>
    </tr>
    <tr class="step-items">
        <td></td>
        <td>normalize to mean of 10,000</td>
        <td></td>
        <td></td>
    </tr>
    <tr class="step-notes">
        <td colspan="4">
            <p>
            </p>
        </td>
    </tr>

    <tr class="step-title">
        <td colspan="4"><p>Smoothing</p></td>
    </tr>
    <tr class="step-items">
        <td>x</td>
        <td>x</td>
        <td>x</td>
        <td>optional</td>
    </tr>
    <tr class="step-notes">
        <td colspan="4">
            <p>Smoothing is often used in volumetric studies to boost signal
               without boosting noise. In cortical surface studies, the same 
               effect is obtained by averaging parcels without sacrificing
               spatial resolution in the same way.
            </p>
        </td>
    </tr>

    <tr class="step-title">
        <th colspan="4"><p>Final image</p></th>
    </tr>
    <tr class="step-items">
        <td></td>
        <td>files/task-*/task-*_nonlin_norm.wdir/<br/>
        </td>
        <td></td>
        <td></td>
    </tr>

</table>

During later analyses, see [fMRIPrep's take](https://fmriprep.org/en/stable/outputs.html#confounds)
and [the 2007 Behzadi CompCor paper](https://doi.org/10.1016/j.neuroimage.2007.04.042) regarding confounds.

Additions, subtractions, and any corrections are appreciated.
[Email Mike](mailto:mikeschmidt@schmidtgracen.com) or `git clone https://github.com/mfschmidt/mfschmidt.github.io.git`, edit, and submit a pull request.
