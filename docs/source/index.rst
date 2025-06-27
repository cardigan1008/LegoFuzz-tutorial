Welcome to documentation for LegoFuzz OOPSLA'25 Artifact!
===================================

Introduction
---------------

This is the documentation for the LegoFuzz OOPSLA'25 Artifact. It includes the source code of our tool, as well as all the necessary scripts and data required to reproduce the experimental results and support the claims made in our paper.

The main claims supported by this artifact are: 

1. (Sec 5.2) In both GCC and LLVM, many of the bugs are related to loop transformations and peephole optimizations, which is consistent with findings from prior empirical studies. 
2. (Sec 5.3) ...indicating that iterative synthesis plays a crucial role in generating more complex programs that engage a wider range of compiler features.
3. (Sec 5.3) This high efficiency indicates that program generation is not a bottleneck of LegoFuzz.
4. (Sec 5.4) ...indicating that a larger code database provides more diverse features

The experimental results are presented in Section 5, along with Tables 1–5 and Figures 9, 11–14.

All these claims and results are *fully reproduced* in :doc:`/evaluation`.

For *reusability*, the core of this artifact is the LLM-based compiler testing tool LegoFuzz. :doc:`/legofuzz`` provides a quick start guide and complete instructions for using both the two-stage generation and fuzzing components.

Hardware Dependencies
---------------

The full evaluation of this artifact is **resource-intensive**. 

We recommend to run the full evaluation on a machine with at least **32 cores, 64GB memory, and 100GB disk space** for a reasonable evaluation time (< 5 hours).

For LegoFuzz, **no GPU is required**. A GPU is only needed if you wish to regenerate the baselines (Fuzz4All and WhiteFox) from scratch.

However, we provide **cached results**, so GPU usage is optional.

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