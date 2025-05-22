Getting Started
===============

This tutorial walks you through logging into the NASA ADAPT system, accessing the GPU cluster, setting up your working environment, and cloning the necessary repositories.

1. Log into the NASA ADAPT Server
---------------------------------

.. code-block:: bash

   ssh user@adaptlogin.nasa.gov

2. Connect to the PRISM GPU Cluster
-----------------------------------

.. code-block:: bash

   ssh gpulogin1

3. Request Computational Resources (Slurm)
------------------------------------------

To run compute jobs, request an interactive Slurm session. For example:

.. code-block:: bash

   salloc -G1 -J composite-text -c 10

This requests:

- An interactive job (`salloc`)
- 1 GPU (`-G1`)
- A job name of `"composite-text"` (`-J`)
- 10 CPU cores (`-c 10`)

.. seealso::
   NCCS Slurm Guide: https://www.nccs.nasa.gov/nccs-users/instructional/adapt-instructional/slurm

4. Create Personal Development Directory
----------------------------------------

Ensure the environment variable `$USER` is set correctly:

.. code-block:: bash

   echo $USER

This should return your username.

Now create your development directory:

.. code-block:: bash

   mkdir -p /explore/nobackup/people/$USER/development

5. Clone Required GitHub Repositories
-------------------------------------

Navigate to your development directory and clone the repositories:

.. code-block:: bash

   cd /explore/nobackup/people/$USER/development
   git clone https://github.com/nasa-nccs-hpda/ethiopia-lcluc-tensorflow
   git clone https://github.com/nasa-nccs-hpda/vhr-composite

6. Load Required Module(s)
--------------------------

To use Singularity, first load the appropriate module:

.. code-block:: bash

   module load singularity

.. note::

   Singularity is a tool used to run containers—self-contained environments that package software and dependencies. 
   This helps ensure your code runs consistently across different systems. If you're new to Singularity or want 
   to learn more, see the `official Singularity documentation <https://sylabs.io/guides/>`_.

7. Build the Container Environment (Singularity)
------------------------------------------------

Containers allow for reproducible software environments. On ADAPT, it's best to build containers in local scratch space (`/lscratch`), which is high-speed but **temporary** storage available only for the duration of a job.

.. note::
   Files in `/lscratch` may be deleted when your job ends.

.. warning::
   You **must** be on a GPU node *and* have an active Slurm job (e.g., requested via ``salloc``) to access ``/lscratch``.  
   Attempting to access or create directories in ``/lscratch`` **without** this may result in a ``Permission denied`` error.

- First, create your personal container directory structure in `/lscratch`:

  .. code-block:: bash

     mkdir -p /lscratch/$USER/container

- Then build the **Ethiopia LCLUC TensorFlow** container into a writable sandbox:

  .. code-block:: bash

     singularity build --sandbox /lscratch/$USER/container/ethiopia-lcluc-tensorflow docker://nasanccs/ethiopia-lcluc-tensorflow:latest

- Also build the **TensorFlow Caney** container:

  .. code-block:: bash

     singularity build --sandbox /lscratch/$USER/container/tensorflow-caney docker://nasanccs/tensorflow-caney:latest

These commands pull the latest container images from Docker Hub and build them into writable sandboxes under your personal scratch space.

Bonus Tips
----------

Using a PIV Card
~~~~~~~~~~~~~~~~

If you're using a PIV card to log in, you can cache your login key to avoid re-entering your PIN repeatedly:

.. code-block:: bash

   ssh-add -s /usr/lib/ssh-keychain.dylib

Running Graphical Applications (X11 Forwarding)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

To launch GUI tools like QGIS from ADAPT, ensure X11 forwarding is enabled and an X11 server is running locally.

- Example login with trusted forwarding:

  .. code-block:: bash

     ssh -Y user@adaptlogin.nasa.gov
     ssh -Y gpulogin1

- Local X11 servers:

  - macOS: `XQuartz <https://www.xquartz.org/>`_
  - Windows: `MobaXterm <https://mobaxterm.mobatek.net/>`_

Using `screen` to Detach Terminal Sessions
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

To keep processes running after disconnecting from SSH, use `screen` to start a detachable terminal session:

- Start a new session:

  .. code-block:: bash

     screen

- Detach the session (e.g., when you're done or want to close SSH):

  .. code-block:: bash

     screen -d

  .. note::
     This ends the SSH session **but keeps the process running** in the background.

- Reattach later:

  .. code-block:: bash

     screen -r

Advanced screen commands:

- Name your session:

  .. code-block:: bash

     screen -S mysession

- List all active sessions:

  .. code-block:: bash

     screen -ls

- Kill a session:

  .. code-block:: bash

     screen -X -S mysession quit

.. seealso::
   Full GNU Screen manual: https://www.gnu.org/software/screen/manual/screen.txt/

