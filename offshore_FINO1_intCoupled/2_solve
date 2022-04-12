#!/bin/bash
#SBATCH --job-name=rOffshoreFino
#SBATCH --output foam2run_%j.log
#SBATCH --ntasks=360         # Change on setUp as well
#SBATCH --time=10-00
#SBATCH --account=car
#SBATCH --mail-user=regis.thedin@nrel.gov
#SBATCH --mail-type=ALL

source $HOME/.bash_profile
cores=$SLURM_NTASKS

echo "Working directory is" $SLURM_SUBMIT_DIR
echo "Job name is" $SLURM_JOB_NAME
echo "Submit time is" $(squeue -u $USER -o '%30j %20V' | grep -e $SLURM_JOB_NAME | awk '{print $2}')
echo "Starting OpenFOAM job at: " $(date)
echo "using" $cores "core(s)"

# ******************************************** USER INPUT ******************************************** #
OpenFOAM-6-gcc-dev              # OpenFOAM/SOWFA version. OpenFOAM-6-{gcc,intel}-{central,dev}
startTime=113200                # Start time
timeBdData=14400                 # Time executed further after endTime. Also saves averages
writeIntervalBdData=3600
solver=superDeliciousVanilla
initializer=setFieldsABL

cp constant/ABLProperties.upperdamping.lessRAM          constant/ABLProperties
cp system/sampling/vmasts_1kmGrid.bak                   system/sampling/vmasts_1kmGrid
cp system/sampling/vmasts_small3x3Grid.bak              system/sampling/vmasts_small3x3Grid
cp system/sampling/slicesFINO1_correlation.bak          system/sampling/slicesFINO1_correlation
# **************************************************************************************************** #

# **************************************** PERFORM SOME CHECKS *************************************** #
if [ ! -f foam1preprocess.log ];                                        then echo "Job killed (1)"; scancel $SLURM_JOBID; fi
if [ ! -f system/controlDict.$solver.startAt$startTime ];               then echo "Job killed (2)"; scancel $SLURM_JOBID; fi
if [ $initializer = setFieldsABL ] && [ ! -f system/setFieldsABLDict ]; then echo "Job killed (3)"; scancel $SLURM_JOBID; fi
if [ ! -f setUp ];                                                      then echo "Job killed (4)"; scancel $SLURM_JOBID; fi
if [ $(foamDictionary -entry "nCores" -value setUp) -ne $cores ];       then echo "Job killed (5)"; scancel $SLURM_JOBID; fi
# **************************************************************************************************** #

# ***************************** COPY APPROPRIATE FILES AND SET VARIABLES ***************************** #
cp system/controlDict.$solver.startAt$startTime        system/controlDict

endTime=$(foamDictionary -entry "endTime" -value system/controlDict.$solver.startAt$startTime)
latestTime=$(foamListTimes -processor -latestTime -withZero | tail -1)
# **************************************************************************************************** #

# Run decomposePar if domain not yet decomposed
if [ ! -d "processor0" ]; then
	decomposePar -cellDist -force > log.2.decomposePar 2>&1
fi

# Run the flow field initializer
if [ $initializer = setFieldsABL ] && [ ! -f log.2.$initializer ];  then
   srun -n $cores --cpu_bind=cores $initializer -parallel > log.2.$initializer 2>&1
fi

# Split run to get to developed-flow stage
if [ $latestTime -lt $endTime ]; then
    foamDictionary -entry "temporalAverages.enabled"    -set "false" -disableFunctionEntries system/sampling/temporalAverages
    foamDictionary -entry "boundaryData.enabled"        -set "false" -disableFunctionEntries system/sampling/boundaryData
    foamDictionary -entry "slicesFINO1_correlation.enabled"   -set "false" -disableFunctionEntries system/sampling/slicesFINO1_correlation
    foamDictionary -entry "boundaryData.enabled"        -set "false" -disableFunctionEntries system/sampling/boundaryData
    foamDictionary -entry "probeSurface.enabled"        -set "false" -disableFunctionEntries system/sampling/probeSurface
    foamDictionary -entry "vmasts_1kmGrid.enabled"      -set "false" -disableFunctionEntries system/sampling/vmasts_1kmGrid
    foamDictionary -entry "vmasts_small3x3Grid.enabled" -set "false" -disableFunctionEntries system/sampling/vmasts_small3x3Grid
    foamDictionary -entry "startTime"                   -set $latestTime -disableFunctionEntries system/controlDict
    srun -n $cores --cpu_bind=cores $solver -parallel > log.2.$solver.startAt$latestTime 2>&1
fi

# Split run to save boundaryData and averages
continueTime=$(( $latestTime > $endTime ? $latestTime : $endTime ))
foamDictionary -entry "writeInterval"               -set $writeIntervalBdData -disableFunctionEntries system/controlDict
foamDictionary -entry "temporalAverages.timeStart"  -set $endTime -disableFunctionEntries system/sampling/temporalAverages
foamDictionary -entry "temporalAverages.enabled"    -set "true" -disableFunctionEntries system/sampling/temporalAverages
foamDictionary -entry "slicesFINO1_correlation.enabled"   -set "true" -disableFunctionEntries system/sampling/slicesFINO1_correlation
foamDictionary -entry "boundaryData.enabled"        -set "true" -disableFunctionEntries system/sampling/boundaryData
foamDictionary -entry "probeSurface.enabled"        -set "true" -disableFunctionEntries system/sampling/probeSurface
foamDictionary -entry "vmasts_1kmGrid.enabled"      -set "true" -disableFunctionEntries system/sampling/vmasts_1kmGrid
foamDictionary -entry "vmasts_small3x3Grid.enabled" -set "true" -disableFunctionEntries system/sampling/vmasts_small3x3Grid
foamDictionary -entry "startTime"            -set $continueTime -disableFunctionEntries system/controlDict
foamDictionary -entry "endTime" -set $(echo $endTime+$timeBdData  | bc) -disableFunctionEntries system/controlDict
srun -n $cores --cpu_bind=cores $solver -parallel > log.2.$solver.saveBdData.startAt$continueTime 2>&1

echo "Ending OpenFOAM job at: " $(date)

cp foam2run_${SLURM_JOBID}.log foam2run_startAt$continueTime

# **************************************************************************************************** #
