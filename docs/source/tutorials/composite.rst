Running the Landcover Composite Pipeline
========================================

This tutorial walks you through the full landcover compositing pipeline. After generating predictions across your study area, this pipeline helps you turn those results into composite landcover layers through three main stages:

1. **Build Footprints**
2. **Extract Metadata**
3. **Build Composite**

.. note::

   If you have not already, please complete the steps outlined in the :doc:`Getting Started guide <quickstart>`.

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

    singularity exec \
      --env PYTHONPATH="/explore/nobackup/people/$USER/development/vhr-composite:/explore/nobackup/people/$USER/development/ethiopia-lcluc-tensorflow" \  # Set environment path
      --nv \  # Enable GPU support
      -B $NOBACKUP,/lscratch,/explore/nobackup/people,/explore/nobackup/projects,/panfs/ccds02/nobackup/projects \  # Bind mounts
      /lscratch/$USER/container/tensorflow-caney \  # Container path
      python /explore/nobackup/people/$USER/development/ethiopia-lcluc-tensorflow/ethiopia_lcluc_tensorflow/view/landcover_composite_pipeline_cli.py \  # Script
      -c /explore/nobackup/people/$USER/development/config/composite_ethiopia_epoch1.yaml \  # Config
      -s build_footprints  # Step

Expected log snippet::

    INFO; Found 2144 tifs to process.
    INFO; Created 2,144 records
    INFO; Saved .../Amhara_M1BS_griddedToa.gpkg
    INFO; Took 4.39 min.

Step 2: Extract Metadata
------------------------

Run:

::

    singularity exec \
      --env PYTHONPATH="/explore/nobackup/people/$USER/development/vhr-composite:/explore/nobackup/people/$USER/development/ethiopia-lcluc-tensorflow" \  # Set environment path
      --nv \  # Enable GPU support
      -B $NOBACKUP,/lscratch,/explore/nobackup/people,/explore/nobackup/projects,/panfs/ccds02/nobackup/projects \  # Bind mounts
      /lscratch/$USER/container/tensorflow-caney \  # Container path
      python /explore/nobackup/people/$USER/development/ethiopia-lcluc-tensorflow/ethiopia_lcluc_tensorflow/view/landcover_composite_pipeline_cli.py \  # Script
      -c /explore/nobackup/people/$USER/development/config/composite_ethiopia_epoch1.yaml \  # Config
      -s extract_metadata  # Step

Expected log snippet::

    INFO; Created 39,367 records
    INFO; Saved .../Amhara_M1BS_griddedToa_metadata.gpkg
    INFO; Took 0.05 min.

Step 3: Build Composite
-----------------------

Before running the composite step, you'll need a tile list. Tile lists specify the tiles to be processed and are located in:

::

    /explore/nobackup/people/$USER/development/ethiopia-lcluc-tensorflow/projects/composite/configs/tile_lists/

For a quick test, you can use:

::

    test_tile_0.txt  # Contains two tiles to verify your setup

To run this test interactively (within an `salloc` session), use:

::

    singularity exec \
      --env PYTHONPATH="/explore/nobackup/people/$USER/development/vhr-composite:/explore/nobackup/people/$USER/development/ethiopia-lcluc-tensorflow" \  # Set environment path
      --nv \  # Enable GPU support
      -B $NOBACKUP,/lscratch,/explore/nobackup/people,/explore/nobackup/projects,/panfs/ccds02/nobackup/projects \  # Bind mounts
      /lscratch/$USER/container/tensorflow-caney \  # Container path
      python /explore/nobackup/people/$USER/development/ethiopia-lcluc-tensorflow/ethiopia_lcluc_tensorflow/view/landcover_composite_pipeline_cli.py \  # Script
      -c /explore/nobackup/people/$USER/development/config/composite_ethiopia_epoch1.yaml \  # Config
      -t /explore/nobackup/people/$USER/development/ethiopia-lcluc-tensorflow/projects/composite/configs/tile_lists/test_tile_0.txt \  # Tile list
      -s composite  # Step

Expected log snippet::

    INFO; Metadata includes 39367 strips.
    INFO; Reducing with multi-mode
    INFO; Writing warped to .tif: .../Amhara.M1BS.h26v42.2009.2015.mode.tif
    INFO; Writing class-pct and nobservations .tif
    INFO; Took 1.79 min.

Running in Batch Mode (Optional)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

To composite many tiles (e.g., all Amhara tiles), it’s better to use a batch approach.

Tile lists for Amhara are named:

::

    amhara_tiles_*.txt

and are stored in:

::

    /explore/nobackup/people/$USER/development/ethiopia-lcluc-tensorflow/projects/composite/configs/tile_lists/

To run them in parallel using Slurm:

1. **Create a scripts directory and navigate into it:**

::

    mkdir -p /explore/nobackup/people/$USER/development/scripts
    cd /explore/nobackup/people/$USER/development/scripts

2. **Create and open a new script file:**

::

    touch run_composites.sh
    nano run_composites.sh

3. **Paste the following into nano** (use **right-click** or **Shift+Insert** to paste):

.. code-block:: bash

   #!/bin/bash

   # Ensure log directory exists
   mkdir -p /explore/nobackup/people/$USER/development/logs

   # Submit each tile list as a batch job
   for tile_file in /explore/nobackup/people/$USER/development/ethiopia-lcluc-tensorflow/projects/composite/configs/tile_lists/amhara_tiles_*.txt; do
       sbatch --mem-per-cpu=32G --gres=gpu:1 -c10 -t05-00:00:00 \
       --output=/explore/nobackup/people/$USER/development/logs/run_tiles_%j.out \
       --error=/explore/nobackup/people/$USER/development/logs/run_tiles_%j.err \
       -J composite \
       --wrap="singularity exec --env PYTHONPATH='/explore/nobackup/people/$USER/development/vhr-composite:/explore/nobackup/people/$USER/development/ethiopia-lcluc-tensorflow' \
       --nv -B \$NOBACKUP,/lscratch,/explore/nobackup/people,/explore/nobackup/projects,/panfs/ccds02/nobackup/projects \
       /explore/nobackup/people/$USER/development/tensorflow-caney \
       python /explore/nobackup/people/$USER/development/ethiopia-lcluc-tensorflow/ethiopia_lcluc_tensorflow/view/landcover_composite_pipeline_cli.py \
       -c /explore/nobackup/people/$USER/development/config/composite_ethiopia_epoch1.yaml \
       -t \${tile_file} -s composite"
   done

4. **Save and exit nano:**

- Press **Ctrl+O** to save, then **Enter** to confirm.
- Press **Ctrl+X** to exit.

5. **Make the script executable and run it:**

::

    chmod +x run_composites.sh
    ./run_composites.sh

Monitoring Job Status
~~~~~~~~~~~~~~~~~~~~~

Check the status of your jobs with:

::

    squeue -u $USER

Output and error logs will be stored in:

::

    /explore/nobackup/people/$USER/development/logs/

Example successful `.out` log snippet::

    Running for tile list: .../amhara_tiles_0.txt
    INFO; Output logs sent to: .../2009.2015/2009.2015.log
    INFO; Created output dir: .../2009.2015
    INFO; Reading in metadata .../Amhara_M1BS_griddedToa_metadata.gpkg
    INFO; Created 39,367 records
    INFO; Saved updated metadata file
    INFO; ************************************************************
    INFO; * Initializing compositing *
    INFO; ************************************************************
    INFO; Loaded grid: Amhara_grid.gpkg
    INFO; Metadata includes 39367 strips.
    INFO; Strips remaining after filters: 16025.
    INFO; Reading tiles from: amhara_tiles_0.txt
    INFO; Tiles provided in amhara_tiles_0: 100

Verifying Outputs
-----------------

After running the compositing step—whether interactively or using batch mode—you can check the output directory specified in your config file to verify success.

For example, if your `output_dir` is set to:

::

    /explore/nobackup/people/$USER/development/sample_data/ethiopia/cnn_landcover_composite/ethiopia-new

you should expect to see:

1. A `.gpkg` file for each tile list that was processed:

::

    Amhara-otcb.v11-qaTest1-Amhara.M1BS-amhara_tiles_0.gpkg

2. A subfolder named after the year range you configured:

::

    /explore/.../2009.2015/

Inside that folder, for each tile (e.g., `h31v12`), look for these output `.tif` files:

::

    Amhara.M1BS.h31v12.2009.2015.class-pct-temp.tif
    Amhara.M1BS.h31v12.2009.2015.class-pct.tif
    Amhara.M1BS.h31v12.2009.2015.mode-temp.tif
    Amhara.M1BS.h31v12.2009.2015.mode.tif
    Amhara.M1BS.h31v12.2009.2015.nobservations-temp.tif

These files indicate the compositing ran successfully and produced the expected classification and observation layers.
