Running the Landcover Composite Pipeline
========================================

This tutorial walks you through the full landcover compositing pipeline. After generating predictions across your study area, this pipeline helps you turn those results into composite landcover layers through three main stages:

1. **Build Footprints**
2. **Extract Metadata**
3. **Build Composite**

Setup
-----

Before running the pipeline, you'll need to:

1. Navigate to your development directory:

::

    cd /explore/nobackup/people/$USER/development/

2. Create your output directory:

::

    mkdir -p sample_data/ethiopia/cnn_landcover_composite/ethiopia-v8

3. Edit the configuration file:

Path to config:

::

    /explore/nobackup/people/$USER/development/ethiopia-lcluc-tensorflow/projects/composite/configs/dev/composite_ethiopia_epoch1.yaml

Edit with:

::

    nano composite_ethiopia_epoch1.yaml

Key fields to update:

::

    start_year: 2009
    end_year: 2015
    output_dir: '/explore/nobackup/people/$USER/development/sample_data/ethiopia/cnn_landcover_composite/ethiopia-v8'

Step 1: Build Footprints
------------------------

Run:

::

    singularity exec --env PYTHONPATH="/explore/nobackup/people/$USER/development/vhr-composite:/explore/nobackup/people/$USER/development/ethiopia-lcluc-tensorflow" --nv \
      -B $NOBACKUP,/lscratch,/explore/nobackup/people,/explore/nobackup/projects,/panfs/ccds02/nobackup/projects \
      /lscratch/$USER/container/tensorflow-caney \
      python /explore/nobackup/people/$USER/development/ethiopia-lcluc-tensorflow/ethiopia_lcluc_tensorflow/view/landcover_composite_pipeline_cli.py \
      -c /explore/nobackup/people/$USER/development/config/composite_ethiopia_epoch1.yaml \
      -s build_footprints

Expected log snippet::

    INFO; Found 2144 tifs to process.
    INFO; Created 2,144 records
    INFO; Saved .../Amhara_M1BS_griddedToa.gpkg
    INFO; Took 4.39 min.

Step 2: Extract Metadata
------------------------

Run:

::

    singularity exec --env PYTHONPATH="/explore/nobackup/people/$USER/development/vhr-composite:/explore/nobackup/people/$USER/development/ethiopia-lcluc-tensorflow" --nv \
      -B $NOBACKUP,/lscratch,/explore/nobackup/people,/explore/nobackup/projects,/panfs/ccds02/nobackup/projects \
      /lscratch/$USER/container/tensorflow-caney \
      python /explore/nobackup/people/$USER/development/ethiopia-lcluc-tensorflow/ethiopia_lcluc_tensorflow/view/landcover_composite_pipeline_cli.py \
      -c /explore/nobackup/people/$USER/development/config/composite_ethiopia_epoch1.yaml \
      -s extract_metadata

Expected log snippet::

    INFO; Created 39,367 records
    INFO; Saved .../Amhara_M1BS_griddedToa_metadata.gpkg
    INFO; Took 0.05 min.

Step 3: Build Composite
-----------------------

Ensure you have a tile list file (e.g. ``test_1_tiles.txt``) then run:

::

    singularity exec --env PYTHONPATH="/explore/nobackup/people/$USER/development/vhr-composite:/explore/nobackup/people/$USER/development/ethiopia-lcluc-tensorflow" --nv \
      -B $NOBACKUP,/lscratch,/explore/nobackup/people,/explore/nobackup/projects,/panfs/ccds02/nobackup/projects \
      /lscratch/$USER/container/tensorflow-caney \
      python /explore/nobackup/people/$USER/development/ethiopia-lcluc-tensorflow/ethiopia_lcluc_tensorflow/view/landcover_composite_pipeline_cli.py \
      -c /explore/nobackup/people/$USER/development/config/composite_ethiopia_epoch1.yaml \
      -t /explore/nobackup/people/$USER/development/ethiopia-lcluc-tensorflow/projects/composite/configs/dev/test_1_tiles.txt \
      -s composite

Expected log snippet::

    INFO; Metadata includes 39367 strips.
    INFO; Reducing with multi-mode
    INFO; Writing warped to .tif: .../2009.2015/Amhara.M1BS.h26v42.2009.2015.mode.tif
    INFO; Writing class-pct and nobservations .tif
    INFO; Took 1.79 min.

Final Checks
------------

- All logs should be found in:

::

    .../cnn_landcover_composite/ethiopia-v8/2009.2015/*.log

- Output directory should contain:

  - ``*.gpkg`` files
  - ``*.tif`` files (mode, class-pct, nobservations)

Tips
----

- Use ``$USER`` in all paths to generalize for different users.
- Check ``.log`` files for troubleshooting.
- Always confirm outputs exist as expected after each step.
