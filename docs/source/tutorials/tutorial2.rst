Using QGIS on ADAPT
===================

QGIS (`GitHub repository <https://github.com/qgis/QGIS>`_) is an open-source Geographic Information System (GIS) that allows you to inspect and analyze geospatial data visually. This guide explains how to use QGIS on the NASA ADAPT system for visualization tasks.

.. note::
   QGIS is especially helpful for exploring geospatial inputs and outputs from LCLUC and similar remote sensing projects.

1. Enable Graphical Display with X11 Forwarding
-----------------------------------------------

To use the QGIS graphical interface on ADAPT:

- Ensure you have an X11 display server installed on your local machine:
  - `XQuartz <https://www.xquartz.org/>`_ for macOS
  - `MobaXterm <https://mobaxterm.mobatek.net/>`_ for Windows

- Enable trusted X11 forwarding when connecting to ADAPT:

  .. code-block:: bash

     ssh -XY user@adaptlogin.nasa.gov
     ssh -XY gpulogin1

.. warning::
   If you're only visualizing data (not running computations), **do not request resources with `salloc`**. Run QGIS in a separate session from any compute jobs.

2. Load the Default QGIS Module (Basic Usage)
---------------------------------------------

A default version of QGIS is available via environment modules:

.. code-block:: bash

   module load qgis
   qgis --version

This loads:

.. code-block:: text

   QGIS 3.18.3-Zürich 'Zürich' (exported)

You can open supported geospatial files directly:

.. code-block:: bash

   qgis /path/to/image.tif

3. Use an Updated QGIS Version via Conda (Recommended)
-------------------------------------------------------

A newer QGIS environment with enhanced plugins is available via a shared Conda environment file.

**Step-by-step:**

- Load Conda:

  .. code-block:: bash

     module load conda

- (Optional) Reset your Conda environments if needed:

  .. code-block:: bash

     mv ~/.conda $NOBACKUP
     cd /explore/nobackup/people/$USER
     rm -rf .conda
     ln -s $NOBACKUP/.conda ~/.conda

- Copy the custom environment YAML file to your development directory:

  .. code-block:: bash

     cp /explore/nobackup/people/mwooten3/shared/_conda/myqgis.yml /explore/nobackup/people/$USER/development/

- Remove any existing `myqgis` environment (if applicable):

  .. code-block:: bash

     conda env remove -n myqgis

- Create the new environment:

  .. code-block:: bash

     cd /explore/nobackup/people/$USER/development/
     conda env create -f myqgis.yml

- Activate the environment:

  .. code-block:: bash

     conda activate myqgis

- Confirm the version:

  .. code-block:: bash

     qgis --version

You should see:

.. code-block:: text

   QGIS 3.30.1-'s-Hertogenbosch 's-Hertogenbosch' (exported)

4. Open Files in QGIS
---------------------

To open imagery or vector layers:

.. code-block:: bash

   qgis /path/to/image.tif

.. note::
   Supported formats include `.tif`, `.shp`, `.geojson`, and others.

Tips & Troubleshooting
----------------------

- Avoid opening large datasets if not needed—QGIS can become slow.
- Use ``screen`` if running a long session and want to maintain it after disconnection.
- You may customize the QGIS interface by installing plugins from the Plugin Manager (e.g., semi-automatic classification, quick map services).
