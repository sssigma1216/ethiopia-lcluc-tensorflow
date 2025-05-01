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

Use Slurm to request resources. For example, to start a quick interactive job:

.. code-block:: bash

   salloc -G1 -J composite-text -c 10

This requests:

- An interactive job (`salloc`)
- One GPU (`-G1`)
- A job name `composite-text` (`-J`)
- Ten CPU cores (`-c 10`)

.. seealso::
   NCCS Slurm Guide: https://www.nccs.nasa.gov/nccs-users/instructional/adapt-instructional/slurm

4. Create Personal Development Directory
----------------------------------------

Confirm that the ``$USER`` variable is correctly set:

.. code-block:: bash

   echo $USER

Then create a working directory:

.. code-block:: bash

   mkdir -p /explore/nobackup/people/$USER/development

5. Clone Required GitHub Repositories
-------------------------------------

.. code-block:: bash

   cd /explore/nobackup/people/$USER/development
   git clone https://github.com/nasa-nccs-hpda/ethiopia-lcluc-tensorflow
   git clone https://github.com/nasa-nccs-hpda/vhr-composite

6. Load Required Module(s)
--------------------------

.. code-block:: bash

   module load singularity

7. Build the Container Environment (Singularity)
------------------------------------------------

ADAPT supports Singularity containers. It's recommended to build in local scratch space (`/lscratch`) for faster performance.

- Create your scratch directory:

  .. code-block:: bash

     mkdir -p /lscratch/$USER

- Build the container:

  .. code-block:: bash

     singularity build --sandbox /lscratch/$USER/container/ethiopia-lcluc-tensorflow docker://nasanccs/ethiopia-lcluc-tensorflow:latest

This builds a writable sandbox version of the latest container image from Docker Hub.

Bonus Tips
==========

Using a PIV Card
----------------

If you connect using a PIV card, you can cache your login key:

.. code-block:: bash

   ssh-add -s /usr/lib/ssh-keychain.dylib

Running Graphical Apps (X11 Forwarding)
---------------------------------------

Ensure you have X11 forwarding enabled and an X server installed:

.. code-block:: bash

   ssh -Y user@adaptlogin.nasa.gov
   ssh -Y gpulogin1

X11 Servers:

- macOS: `XQuartz <https://www.xquartz.org/>`_
- Windows: `MobaXterm <https://mobaxterm.mobatek.net/>`_

Using `screen` to Detach Sessions
---------------------------------

Start a new session:

.. code-block:: bash

   screen

Detach without exiting:

.. code-block:: bash

   screen -d

Reattach later:

.. code-block:: bash

   screen -r

Advanced usage:

- Name your session: `screen -S mysession`
- List sessions: `screen -ls`
- Kill a session: `screen -X -S mysession quit`

.. seealso::
   Full documentation: https://www.gnu.org/software/screen/manual/screen.txt/
