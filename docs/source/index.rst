Welcome to documentation for LegoFuzz OOPSLA'25 Artifact!
===================================

Prerequisites
---------------

The full evaluation of this artifact is **resource-intensive**. 

We recommend to run the full evaluation on a machine with at least **32 cores, 32GB memory, and 100GB disk space** for a reasonable evaluation time (< 5 hours).

Getting Started Guide (Kick-the-tire)
---------------

First, download the artifact from the provided `Zenodo <https://doi.org/10.5281/zenodo.15544762>`_ link , then untar the artifact(~20min):

.. code-block:: console

  $ tar -xvf artifact_oopsla2025_legofuzz.tar.gz

Then, enter the docker container:


.. code-block:: console

  $ cd /path/to/the/artifact/
  $ docker load -i image-artifact-oopsla2025-legofuzz.tar  # takes ~10-15 min
  $ ./start_container.py

Then, execute the following command **in the container**:

.. code-block:: console

  $ /artifact/kick/kick.py

.. note::

   The expected exeuction time should be less than 30 mins.

If you see **Kick-the-tire passed!**, you are all set to go.

Contents
--------

.. toctree::

   evaluation
   legofuzz