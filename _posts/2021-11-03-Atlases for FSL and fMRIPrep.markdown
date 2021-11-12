---
layout: post
title:  "Harvard Oxford Atlases"
date:   2021-11-03 21:18:12 -0400
---
# Harvard Oxford Atlases

FSL comes with a symmetrical Harvard-Oxford Atlas in MNI152NLin6Sym space, measuring 91x109x91 voxels.
fMRIPrep v20 uses a different MNI152NLin2009cAsym version of the MNI152 space for regsitering to a template.
So to use the Harvard-Oxford Atlas to extract ROIs from fMRIPrep output you need to find a matching
atlas in the correct space. The one used by fMRIPrep is 97x115x97 voxels and is available
from [templateflow](https://www.templateflow.org/browse). Commands below will do everything for you,
but they are written for linux. I hope they work on a Mac.

You can "install" these atlases into your FSL installation by copying them to FSL's data directory.
This is at $FSLDIR/data/ (For me, and by default, /usr/local/fsl/data/). You may need sudo or admin permissions.
If you run this, any missing files will raise an error, but won't hurt anything.

    S3_URL=https://templateflow.s3.amazonaws.com/tpl-MNI152NLin2009cAsym
    for RES in 1 2; do
        FILE=tpl-MNI152NLin2009cAsym_res-0${RES}_T1w.nii.gz
        wget ${S3_URL}/${FILE} -O ~/Downloads/${FILE}
        sudo cp ~/Downloads/${FILE} $FSLDIR/data/standard/MNI152NLin2009cAsym_T1_${RES}mm.nii.gz
        for CS in HOSPA HOCPA HOCPAL; do
            FILE=tpl-MNI152NLin2009cAsym_res-0${RES}_atlas-${CS}_probseg.nii.gz
            wget ${S3_URL}/${FILE} -O ~/Downloads/${FILE}
            sudo cp ~/Downloads/${FILE} $FSLDIR/data/atlases/HarvardOxford/HarvardOxford-${CS}-prob-${RES}mm.nii.gz
            for TH in 0 25 50; do
                FILE=tpl-MNI152NLin2009cAsym_res-0${RES}_atlas-${CS}_desc-th${TH}_dseg.nii.gz
                wget ${S3_URL}/${FILE} -O ~/Downloads/${FILE}
                sudo cp ~/Downloads/${FILE} \
                        $FSLDIR/data/atlases/HarvardOxford/HarvardOxford-${CS}-maxprob-thr${TH}-${RES}mm.nii.gz
            done
        done
    done

When you're done, your FSL /data/ subdirectory should include these:

    $FSLDIR/data/atlases/HarvardOxford/
        HarvardOxford-{cort,sub}-maxprob-thr{0,25,50}-{1,2}mm.nii.gz  # 12 of these integer atlases
        HarvardOxford-{cort,sub}-prob-{1,2}mm.nii.gz                  # 4 of these probabilistic atlases
        HarvardOxford-HOSPA-maxprob-thr{0,25,50}-{1,2}mm.nii.gz       # 6 new
        HarvardOxford-HOCPA-maxprob-thr{0,25,50}-{1,2}mm.nii.gz       # 6 new
        HarvardOxford-HOCPAL-maxprob-thr{0,25,50}-{1,2}mm.nii.gz      # 6 new
    $FSLDIR/data/standard/
        *                                                             # bunches of other templates
        MNI152NLin2009cAsym_T1_{1,2}mm.nii.gz                         # 2 new

Now you have access to these files in FSL and in any scripts you download from me.
But FSL does not know the labels for the integer values in each region. It gets these from separate label files.
Templateflow does not include the label files, so we have to find or build our own.
You can download a label file for the new 96-region HOCPAL atlas (L for lateral, meaning it has a Left* and Right*
ROI for each of the 48 ROIs in the symmetric version) from
[Enrico Glerian's github](https://github.com/eglerean/funpsy/raw/master/atlases/HarvardOxford/HarvardOxford-Cortical-Lateralized.xml)
and save it to FSL's atlases directory. 

    wget https://github.com/eglerean/funpsy/raw/master/atlases/HarvardOxford/HarvardOxford-Cortical-Lateralized.xml -O ~/Downloads/HarvardOxford-Cortical-Lateralized.xml
    sudo cp ~/Downloads/HarvardOxford-Cortical-Lateralized.xml $FSLDIR/data/atlases/HarvardOxford-Cortical-Lateralized.xml
    rm ~/Downloads/HarvardOxford-Cortical-Lateralized.xml

This file saves us some extra work (Thank you Enrico Glerian!), and should probably be part of the templateflow archive.
But it uses FSL naming conventions that would force us to overwrite the HarvardOxford subcortical atlas.
Both the old and new subcortical atlases, in different spaces, would have the same name.
I think it's best to have both atlases available in FSL, and with consistent naming conventions,
so we need to tweak his label file just a bit. We can use `sed` to fix the names.

    sudo sed -i 's/cortl/HOCPAL/g' ${FSLDIR}/data/atlases/HarvardOxford-Cortical-Lateralized.xml

The subcortical label file is perfectly fine for use with both the old and the new subcortical atlas,
but FSL doesn't know it. We should add the new atlases to the label file so FSL knows it can be used with them.
The following bash commands will rewrite the Subcortical file to include them. Only 2mm atlases
are included here because templateflow does not supply the 1mm probabilistic atlas that is required
for FSL to recognize the set of images.

    PRE_LIN=$(grep -n "</header>" $FSLDIR/data/atlases/HarvardOxford-Subcortical.xml)
    PRE_LIN=$(( ${PRE_LIN%%:*} - 1 ))
    TOT_LIN=$(wc -l $FSLDIR/data/atlases/HarvardOxford-Subcortical.xml)
    TOT_LIN=${TOT_LIN%%\ *}
    head -$(( $PRE_LIN )) $FSLDIR/data/atlases/HarvardOxford-Subcortical.xml > ./hos.xml
    echo "    <images>
          <imagefile>/HarvardOxford/HarvardOxford-HOSPA-prob-2mm</imagefile>
          <summaryimagefile>/HarvardOxford/HarvardOxford-HOSPA-maxprob-thr0-2mm</summaryimagefile>
          <summaryimagefile>/HarvardOxford/HarvardOxford-HOSPA-maxprob-thr25-2mm</summaryimagefile>
          <summaryimagefile>/HarvardOxford/HarvardOxford-HOSPA-maxprob-thr50-2mm</summaryimagefile>
        </images>" >> ./hos.xml
    tail -$(( $TOT_LIN - $PRE_LIN )) $FSLDIR/data/atlases/HarvardOxford-Subcortical.xml >> ./hos.xml
    sudo cp ./hos.xml $FSLDIR/data/atlases/HarvardOxford-Subcortical.xml
    rm ./hos.xml

Templateflow is also missing all cortical HOCPAL probabilistic atlases, so FSL will not recognize them, either.
But this sets things up as far as we can take them for now. Most importantly, scripts looking for atlases in
$FSLDIR/data/atlases/ will be able to find them and extract ROIs in the appropriate spaces.

This process was redone on all NYSPI jjm nodes with FSL on Wednesday, Nov 12, 2021,
so they all contain the MNI152NLin2009cAsym atlases.
