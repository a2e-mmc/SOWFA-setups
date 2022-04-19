# FINO1 SOWFA Boundary-coupled

This case is the offshore FINO1 boundary coupled to be performed with SOWFA.

Pre-requisites:

1. Full-field WRF mesoscale solution including the locations where the boundaries are
2. Compiled `wrttoof` fortran tool (https://github.com/NREL/SOWFA-6/tree/dev/tools/WRFextraction)

Instructions:
    
1. Set the parameters at the top of each script and submit them in order
2. Adjust your conda environment in `2_wrfPreprocess`
