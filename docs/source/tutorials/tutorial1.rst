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

   which requests an interactive job (`salloc`), with 1 GPU (`-G1`), a job name (`-J`) of "composite-text", and 10 CPU cores (`-c`).



4) **Create personal directories to work within**:

- Confirm that the ``$USER`` environment variable is set to your username:

  .. code-block:: bash

     echo $USER

  which should return your username.


- Create a "development" directory:

  .. code-block:: bash

     mkdir -p /explore/nobackup/people/$USER/development


5) **Clone the necessart GitHub repositories**

.. code-block:: bash

   cd /explore/nobackup/people/$USER/development  # Navigate to development folder
   git clone https://github.com/nasa-nccs-hpda/ethiopia-lcluc-tensorflow # Clone the Ethiopia LCLUC repository
   git clone https://github.com/nasa-nccs-hpda/vhr-composite # Clone the VHR Composite tool repository


6) **Load necessary module(s)**

.. code-block:: bash

   module load singularity


7) **Build the container environment using Singularity**

   Containers package software and their dependencies into isolated environments to ensure consistent behavior across systems. On ADAPT, it is recommended to use the high-speed local scratch space (`/lscratch`) when building containers.

   - First, create your personal directory in `/lscratch` (if it doesn't already exist). This storage is local to the compute node and temporary—files may be deleted after your job ends:

     .. code-block:: bash

        mkdir -p /lscratch/$USER

   - Then, build the Ethiopia LCLUC TensorFlow container into a writable sandbox directory using Singularity:

     .. code-block:: bash

        singularity build --sandbox /lscratch/$USER/container/ethiopia-lcluc-tensorflow docker://nasanccs/ethiopia-lcluc-tensorflow:latest

   This command pulls the latest container image from Docker Hub and builds it as a sandbox under your `lscratch` space.


Bonus Tips
----------

- If you are using a PIV card to connect, on your local system, save your login keychain to avoid repeatedly entering your pin by entering the following and answering the subsequent prompt:

  .. code-block:: bash

     ssh-add -s /usr/lib/ssh-keychain.dylib

- If you plan to open any graphical applications while logged on to the ADAPT server (e.g., QGIS), make sure you have an X11 server installed on your local system (such as `XQuartz <https://www.xquartz.org/>`_ for Mac or `MobaXterm <https://mobaxterm.mobatek.net/>`_ for Windows) and enable trusted X11 forwarding when logging on to the server:

  .. code-block:: bash

     ssh -Y user@adaptlogin.nasa.gov
     ssh -Y gpulogin1

- To keep processes running after disconnecting from the server (e.g., long-running jobs or scripts), use ``screen`` to create detachable terminal sessions:

  .. code-block:: bash

     screen	# Start a new screen session
     screen -d	# Detach screen session: Ends current ssh session but keeps processes running

  Reconnect later with:

  .. code-block:: bash

     screen -r          # Reattach session

  If ``screen`` is not installed, you can add it with:

  .. code-block:: bash

     sudo apt install screen    # On Debian/Ubuntu systems

  Additional tips:
  - Use ``screen -S session_name`` to name your session.
  - List existing sessions with ``screen -ls``.
  - Kill a session with ``screen -X -S session_name quit``.
 
 Full screen documentation can be found `here <https://www.gnu.org/software/screen/manual/screen.txt/>`_.
