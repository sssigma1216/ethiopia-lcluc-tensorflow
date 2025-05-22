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

1. **Navigate to your development directory:**

::

    cd /explore/nobackup/people/$USER/development/

2. **Create your output directory:**

This is an example directory name—feel free to use a name that fits your study area and experiment versioning:

::

    mkdir -p sample_data/ethiopia/cnn_landcover_composite/ethiopia-v8

3. **Edit the configuration file:**

Open the configuration file using a text editor, such as `nano`:

::

    nano /explore/nobackup/people/$USER/development/ethiopia-lcluc-tensorflow/projects/composite/configs/dev/composite_ethiopia_epoch1.yaml

Update the following fields:

- **Set the start and end year:**

  ::

      start_year: 2009
      end_year: 2015

- **Set the output directory:**

  ::

      output_dir: '/explore/nobackup/people/$USER/development/sample_data/ethiopia/cnn_landcover_composite/ethiopia-v8'

.. note::

   If you're unfamiliar with using `nano`:

   - Use **arrow keys** to move around.
   - Type or delete to make changes.
   - You can paste with **Shift+Insert** or **right-click** (depending on your terminal).
   - Press `Ctrl+O`, then `Enter` to save.
   - Press `Ctrl+X` to exit.
   - To search, press `Ctrl+W` and type a keyword like `output_dir`.

Step 1: Build Footprints
------------------------

Run the following command to generate the initial footprint layer:

::

    singularity exec --env PYTHONPATH="/explore/nobackup/people/$USER/development/vhr-composite:/explore/nobackup/people/$USER/development/ethiopia-lcluc-tensorflow" --nv -B $NOBACKUP,/lscratch,/explore/nobackup/people,/explore/nobackup/projects,/panfs/ccds02/nobackup/projects /lscratch/$USER/container/tensorflow-caney python /explore/nobackup/people/$USER/development/ethiopia-lcluc-tensorflow/ethiopia_lcluc_tensorflow/view/landcover_composite_pipeline_cli.py -c /explore/nobackup/people/$USER/development/ethiopia-lcluc-tensorflow/projects/composite/configs/dev/composite_ethiopia_epoch1.yaml -s build_footprints

.. note::

   Breakdown of the command:

   - `--env PYTHONPATH=...`: Sets the Python import paths.
   - `--nv`: Enables GPU access inside the container.
   - `-B ...`: Binds required filesystem paths.
   - `-c`: Path to your configuration file.
   - `-s build_footprints`: Runs the footprint-building stage.

Snippet of expected output:

::

    INFO; Found 2144 tifs to process.
    INFO; Created 2,144 records
    INFO; Saved .../Amhara_M1BS_griddedToa.gpkg
    INFO; Took 4.39 min.

Step 2: Extract Metadata
------------------------

Run the following command to extract metadata from the footprints:

::

    singularity exec --env PYTHONPATH="/explore/nobackup/people/$USER/development/vhr-composite:/explore/nobackup/people/$USER/development/ethiopia-lcluc-tensorflow" --nv -B $NOBACKUP,/lscratch,/explore/nobackup/people,/explore/nobackup/projects,/panfs/ccds02/nobackup/projects /lscratch/$USER/container/tensorflow-caney python /explore/nobackup/people/$USER/development/ethiopia-lcluc-tensorflow/ethiopia_lcluc_tensorflow/view/landcover_composite_pipeline_cli.py -c /explore/nobackup/people/$USER/development/ethiopia-lcluc-tensorflow/projects/composite/configs/dev/composite_ethiopia_epoch1.yaml -s extract_metadata

.. note::

   The `-s extract_metadata` flag runs the metadata extraction stage.

Snippet of expected output:

::

    INFO; Created 39,367 records
    INFO; Saved .../Amhara_M1BS_griddedToa_metadata.gpkg
    INFO; Took 0.05 min.

Step 3: Build Composite
-----------------------

.. note::

   This step shows how to run compositing for a **single tile list**, which is useful for verifying your setup interactively.  
   If you want to composite **all tile lists**, see the batch mode section below.

Before running the composite step, you'll need a tile list. These are located in:

::

    /explore/nobackup/people/$USER/development/ethiopia-lcluc-tensorflow/projects/composite/configs/tile_lists/

To test your setup, use:

::

    test_tile_0.txt  # Contains two tiles to verify your setup

Run:

::

    singularity exec --env PYTHONPATH="/explore/nobackup/people/$USER/development/vhr-composite:/explore/nobackup/people/$USER/development/ethiopia-lcluc-tensorflow" --nv -B $NOBACKUP,/lscratch,/explore/nobackup/people,/explore/nobackup/projects,/panfs/ccds02/nobackup/projects /lscratch/$USER/container/tensorflow-caney python /explore/nobackup/people/$USER/development/ethiopia-lcluc-tensorflow/ethiopia_lcluc_tensorflow/view/landcover_composite_pipeline_cli.py -c /explore/nobackup/people/$USER/development/ethiopia-lcluc-tensorflow/projects/composite/configs/dev/composite_ethiopia_epoch1.yaml -t /explore/nobackup/people/$USER/development/ethiopia-lcluc-tensorflow/projects/composite/configs/tile_lists/test_tile_0.txt -s composite

Snippet of expected output:

::

    INFO; Metadata includes 39367 strips.
    INFO; Reducing with multi-mode
    INFO; Writing warped to .tif: .../Amhara.M1BS.h26v42.2009.2015.mode.tif
    INFO; Writing class-pct and nobservations .tif
    INFO; Took 1.79 min.

Running in Batch Mode (recommended for many tiles)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

When compositing a large number of tiles (e.g., for the entire Amhara region), it's more efficient to use **batch submission**. Instead of running each tile manually, this method submits one job per tile list to Slurm—allowing parallel execution.

1. **Create the batch script:**

Paste the following into `run_composite.sbatch`:

.. code-block:: bash

    #!/bin/bash

    TILE_LIST_DIR="/explore/nobackup/people/$USER/development/ethiopia-lcluc-tensorflow/projects/composite/configs/tile_lists"
    CONFIG="/explore/nobackup/people/$USER/development/ethiopia-lcluc-tensorflow/projects/composite/configs/dev/composite_ethiopia_epoch1.yaml"
    SCRIPT="/explore/nobackup/people/$USER/development/ethiopia-lcluc-tensorflow/ethiopia_lcluc_tensorflow/view/landcover_composite_pipeline_cli.py"
    CONTAINER="/lscratch/$USER/container/tensorflow-caney"
    PYTHONPATH="/explore/nobackup/people/$USER/development/vhr-composite:/explore/nobackup/people/$USER/development/ethiopia-lcluc-tensorflow"
    BIND_PATHS="$NOBACKUP,/lscratch,/explore/nobackup/people,/explore/nobackup/projects,/panfs/ccds02/nobackup/projects"

    # Ensure log directory exists
    mkdir -p /explore/nobackup/people/$USER/development/logs

    for TILE_LIST in "$TILE_LIST_DIR"/amhara_tiles_*.txt; do
      TILE_NAME=$(basename "$TILE_LIST" .txt)

      sbatch <<EOF
    #!/bin/bash
    #SBATCH --job-name=comp-${TILE_NAME}
    #SBATCH --output=/explore/nobackup/people/$USER/development/logs/${TILE_NAME}_%j.out
    #SBATCH --error=/explore/nobackup/people/$USER/development/logs/${TILE_NAME}_%j.err
    #SBATCH --time=24:00:00
    #SBATCH --gres=gpu:1
    #SBATCH --mem=32G
    #SBATCH --cpus-per-task=10

    module load singularity

    export PYTHONPATH="${PYTHONPATH}"

    singularity exec --env PYTHONPATH="${PYTHONPATH}" --nv -B ${BIND_PATHS} "${CONTAINER}" \
      python "${SCRIPT}" -c "${CONFIG}" -t "${TILE_LIST}" -s composite
    EOF

    done
    
2. **Submit the batch script:**

::

    sbatch run_composite.sbatch

3. **Check the status of your jobs:**

::

    squeue -u $USER

Output and error logs will appear in:

::

    /explore/nobackup/people/$USER/development/logs/
    
Snippet of an example of a sucessful `.out` log::
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

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
-----

After running the compositing step—whether interactively or using batch mode—you can check the output directory specified in your configuration file to verify success.

For example, if your `output_dir` is set to:

::

    /explore/nobackup/people/$USER/development/sample_data/ethiopia/cnn_landcover_composite/ethiopia-v8

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
