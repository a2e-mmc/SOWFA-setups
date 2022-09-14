#!/bin/bash
#SBATCH --job-name=saveSlices
#SBATCH --output foam3slices
#SBATCH --ntasks=36 
#SBATCH --time=4:00:00
#SBATCH --account=car

source $HOME/.bash_profile

# ******************************************** USER INPUT ******************************************** #
module load paraview/5.6.0
vars=(U)
terrain=(80)
znormal=()
xnormal=()
ynormal=(0)
# **************************************************************************************************** #

for v in ${vars[@]}; do
    # save terrain slices
    for t in ${terrain[@]}; do
        pvbatch --mesa ~/utilities/saveNormalSlice.py -var $v -slice terrain -normal $t -comp Z -lb -4 -ub 4
        pvbatch --mesa ~/utilities/saveNormalSlice.py -var $v -slice terrain -normal $t -comp Mag -lb 2 -ub 18
    done
    # save zNormal slices
    for z in ${znormal[@]}; do
        pvbatch --mesa ~/utilities/saveNormalSlice.py -var $v -slice zNormal -normal $z -comp Z -lb -4 -ub 4
        pvbatch --mesa ~/utilities/saveNormalSlice.py -var $v -slice zNormal -normal $z -comp Mag -lb 2 -ub 18
    done
    # save xNormal slices
    for x in ${xnormal[@]}; do
        pvbatch --mesa ~/utilities/saveNormalSlice.py -var $v -slice xNormal -normal $x -comp Z -lb -4 -ub 4
        pvbatch --mesa ~/utilities/saveNormalSlice.py -var $v -slice xNormal -normal $x -comp Mag -lb 2 -ub 18
    done
    # save yNormal slices
    for y in ${ynormal[@]}; do
        pvbatch --mesa ~/utilities/saveNormalSlice.py -var $v -slice yNormal -normal $y -comp Z -lb -4 -ub 4
        pvbatch --mesa ~/utilities/saveNormalSlice.py -var $v -slice yNormal -normal $y -comp Mag -lb 2 -ub 18
    done
done

# **************************************************************************************************** #
