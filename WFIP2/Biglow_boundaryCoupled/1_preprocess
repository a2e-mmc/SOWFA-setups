#!/bin/bash
#SBATCH --job-name=sBdCoupling_biglow
#SBATCH --output foam1preprocess.log
#SBATCH --ntasks=16
#SBATCH --time=1:00:00
#SBATCH --account=mmc

source $HOME/.bash_profile
cores=$SLURM_NTASKS

echo "Working directory is" $SLURM_SUBMIT_DIR
echo "Job name is" $SLURM_JOB_NAME
echo "Submit time is" $(squeue -u $USER -o '%30j %20V' | grep -e $SLURM_JOB_NAME | awk '{print $2}')
echo "Starting OpenFOAM job at: " $(date)
echo "using" $cores "core(s)"

# ******************************************** USER INPUT ******************************************** #
OpenFOAM-6-gcc-dev        # OpenFOAM/SOWFA version. OpenFOAM-6-{gcc,intel}-{central,dev}
startTime=0               # Start time
stabState=neutral         # neutral, stable, unstable
terrainSTL=wfip_xm15015to15735_ym15015to13815_ds30.0_SRTMblendToWRF_3N3S3E3W_noShift_setBlockMeshzMinTo379.stl
temperatureBC=unif   # T{stableThroughout,strongInv,unif}  Only used if initializer != precursor
initializer=none     # precursor, setFieldsABL, none. If precursor, set precursorDir, mapTime
    precursorDir=~/OpenFOAM/rthedin-6/run/precursor/neutral.W.5at80.5dTInv.0p05z
    mapTime=20000
    # Change diffusivity on dynamicMeshDict depending on geometry (no quadratic for bump)


cp setUp.neutral.1block.2300mHigh  setUp
cp system/blockMeshDict.unifRes    system/blockMeshDict

# Placeholder for many-level forces
#cp constant/ABLProperties.wrf       constant/ABLProperties
cp constant/ABLProperties.noSources  constant/ABLProperties

# **************************************************************************************************** #

# **************************************** PERFORM SOME CHECKS *************************************** #
if [ ! -f system/controlDict.moveDynamicMesh ];                  then echo "Job killed (1)"; scancel $SLURM_JOBID; fi
if [ ! -f 0.original/qwall.$stabState ];                         then echo "Job killed (3)"; scancel $SLURM_JOBID; fi
if [ ! -f 0.original/pointDisplacement ];                        then echo "Job killed (4)"; scancel $SLURM_JOBID; fi
if [ ! -f constant/dynamicMeshDict ];                            then echo "Job killed (5)"; scancel $SLURM_JOBID; fi
if [ ! -f constant/triSurface/$terrainSTL ] ;                    then echo "Job killed (6)"; scancel $SLURM_JOBID; fi
if [ $initializer != precursor ] && [ $(foamDictionary -entry "nCores" -value setUp.$stabState) -ne $cores ]; 
                                                                 then echo "Job killed (7)"; scancel $SLURM_JOBID; fi

if [ $initializer = precursor ] && [ ! -d $precursorDir/postProcessing/boundaryData ];  then echo "Job killed (8)"; scancel $SLURM_JOBID; fi
if [ $initializer = precursor ] && [ ! -f $precursorDir/constant/givenSourceU ];        then echo "Job killed (9)"; scancel $SLURM_JOBID; fi
if [ $initializer = precursor ] && [ ! -d $precursorDir/$startTime ];                   then echo "Job killed (10)";scancel $SLURM_JOBID; fi
if [ $initializer = setFieldsABL ] && [ ! -f system/setFieldsABLDict.Ugeo.$temperatureBC ]; then echo "Job killed (11)"; scancel $SLURM_JOBID; fi
# **************************************************************************************************** #

# ***************************** COPY APPROPRIATE FILES AND SET VARIABLES ***************************** #
rm -f problemFaces
cd constant/triSurface  &&  ln -sf $terrainSTL    terrain.stl  &&  cd ../..
cp system/controlDict.moveDynamicMesh                  system/controlDict
cp system/fvSolution.mesh                              system/fvSolution
cp system/fvSchemes.mesh                               system/fvSchemes
rm -rf constant/polyMesh
rm -rf constant/boundaryData
rm -rf $startTime $mapTime
# **************************************************************************************************** #

refineMeshLocal()
{
    # Perform the levels refinement backwards
    for  ((i=$1; i>0; i--)); do
        topoSet -dict system/topoSetDict.level.$i -noFunctionObjects > log.1.topoSet.level.$i 2>&1
        refineHexMesh -overwrite -noFunctionObjects level.$i > log.1.refineHexMesh.level.$i 2>&1
        checkMesh -noFunctionObjects > log.1.checkMesh.level.$i 2>&1
    done
}


if [ $initializer = precursor ]; then
    startTime=$(( $mapTime > $startTime ? $mapTime : $startTime ))

    # Get information from the precursor
    cp $precursorDir/setUp                                 setUp
    rm -f setUp.*
    foamDictionary -entry "nCores" -set $cores setUp
    foamDictionary -entry "wallModelAverageType" -set "local" setUp

    # Copy the startTime solution dir from precursor 
    mkdir $startTime.fromPrec
    cp -f $precursorDir/$startTime/{U,T,k,kappat,nut,p_rgh,qwall,Rwall} $startTime.fromPrec
    cp -rf $startTime.fromPrec $startTime
    cp 0.original/pointDisplacement $startTime/

    # Change the wall model type to 'local' from precursor's 'planarAverage'
    foamDictionary -entry "boundaryField.lower.averageType" -set "local" $startTime/Rwall

    # Make the link to the boundary data
    ln -sf $precursorDir/postProcessing/boundaryData constant/boundaryData
    ln -sf $precursorDir/constant/givenSource* constant/

    # Copy the mesh from precursor
    cp -rf $precursorDir/constant/polyMesh constant/

    # Data from precursor has cyclic boundaries. Change that because of the nature of moveDynamicMesh
    changeDictionary -dict system/changeDictionaryDict.TVMIO -time $startTime -subDict dictionaryReplacement > log.1.changeDictionary.TVMIO 2>&1

else
    # Copy the "clean" .original initial fields to a working copy
    cp -rf $startTime.original $startTime
    cp     $startTime/qwall.$stabState   $startTime/qwall  &&  rm $startTime/qwall.*
    rm -f  $startTime/{T.*,U.*}

    # Build the mesh
    blockMesh -noFunctionObjects > log.1.blockMesh 2>&1

    # Create the stacked grid by means of refinements
    refineMeshLocal 3
fi

# Decompose the mesh
foamDictionary -entry "startTime" -set $startTime        system/controlDict
foamDictionary -entry "endTime" -set $(($startTime+101)) system/controlDict
decomposePar -cellDist -force -time $startTime > log.1.decomposePar 2>&1

# Make the mesh conform to the terrain boundary
srun -n $cores moveDynamicMesh -noFunctionObjects -parallel > log.1.moveDynamicMesh 2>&1

# Check the final mesh
srun -n $cores checkMesh -latestTime -noFunctionObjects -parallel > log.1.checkMesh.final 2>&1

# Extract volume information  over time from the moveDynamicMesh output
printf 'time \t min cell vol \t max cell vol \t total vol' > log.1.volumeMeshHistory
grep -e "Total volume" -e "^Time" log.1.moveDynamicMesh | awk 'NR%2 {print $4 "\t" $8 "\t" $12} !(NR%2) {printf $3 "\t"}' >> log.1.volumeMeshHistory 2>&1

# Make the last directory from moveDynamicMesh be the startTime dir
endTimeMesh=$(foamListTimes -processor -latestTime | tail -1)

# For some reason, recomposePar doesn't work with time 0. So we will keep the $endTimeMesh until recomposee
# Because of we run the solver to obtain boundary `points`, we need to get the field files
for (( c=0; c<$cores; c++)); do
    cp processor$c/$startTime/{cellDist,k,kappat,nut,p_rgh,qwall,Rwall,T,U} processor$c/$endTimeMesh
done 

echo "Ending OpenFOAM job at: " $(date)

# **************************************************************************************************** #
