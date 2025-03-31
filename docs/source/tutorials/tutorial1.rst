Getting Started
===============

1) **Log on to the NASA ADAPT server**

.. code-block:: bash

   ssh user@adaptlogin.nasa.gov

2) **Connect to PRISM GPU Cluster**

.. code-block:: bash

   ssh gpulogin1

3) **Request computational resources with Slurm** (see `NCCS guide to Slurm on ADAPT <https://www.nccs.nasa.gov/nccs-users/instructional/adapt-instructional/slurm>`_).  
   For example, to request a quick interactive Slurm job:

.. code-block:: bash

   salloc -G1 -J composite-text -c 10

   which requests an interactive job (salloc), with 1 GPU (-G), a job name (-J) of "composite-text", and 10 CPU cores (-c). 

4) **Create personal directories to work within**:

- Confirm that the ``$USER`` environment variable is set to your username:

  .. code-block:: bash

     echo $USER

  which should return your username.

- Create a "development" directory:

  .. code-block:: bash

     mkdir -p /explore/nobackup/people/$USER/development

5) **Clone the GitHub repository**

.. code-block:: bash

   cd /explore/nobackup/people/$USER/development  # Navigate to development folder
   git clone https://github.com/nasa-nccs-hpda/senegal-lcluc-tensorflow

6) **Load necessary module(s)**

.. code-block:: bash

   module load singularity

Bonus Tips
----------

- If you are using a PIV card to connect, on your local system, save your login keychain to avoid repeatedly entering your pin by entering the following and answering the subsequent prompt:

  .. code-block:: bash

     ssh-add -s /usr/lib/ssh-keychain.dylib

- If you plan to open any graphical applications while logged on to the ADAPT server (e.g., QGIS), make sure you have an X11 server installed on your local system (such as `XQuartz <https://www.xquartz.org/>`_ for Mac or `MobaXterm <https://mobaxterm.mobatek.net/>`_ for Windows) and enable trusted X11 forwarding when logging on to the server:

  .. code-block:: bash

     ssh -Y user@adaptlogin.nasa.gov
     ssh -Y gpulogin1
