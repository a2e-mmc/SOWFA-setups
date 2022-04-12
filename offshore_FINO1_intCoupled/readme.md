# FINO1 SOWFA Internal Coupled

This case is the offshore FINO1 boundary coupled to be performed with SOWFA.

Pre-requisites:
    1. A mesoscale output file should exist. Make sure to give its path in the notebook.

Instructions:
    1. Execute the notebook to generate the SOWFA-ready input files
    2. The files will be inside `drivingData`. For RAM reasons, you may want to clip them to only the period of interest. That clipped file is used in the input files as `*lessRAM`. Adjust accordingly.
