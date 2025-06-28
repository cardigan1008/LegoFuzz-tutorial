Step-by-Step Instructions
==========

Launch the docker container:

.. code-block:: console

  $ cd /path/to/the/artifact/
  $ ./start_container.py


.. note::

   All the following code are executed in the docker container.


.. warning::

   The full evaluation can take upto 20 hours on a 32-core machine. 
   It is thus recommended to open a ``tmux`` session to start it in the background and come back when the experiments are finished.

   Create a tmux session.

   .. code-block:: console

      $ tmux new -s eval   # create a tmux session.

   To leave the job running in the background:

   - ``ctrl`` + ``d``

   To resume the session:

   .. code-block:: console

      $ tmux at -t eval   # resume a tmux session.



Experimental Setup (Section 5.1)
------------

In the paper, we claimed that ``"we collected a function database of 553,246 functions"``
and we showed the statistics in Figure 9.

The function database is located in `/artifact/database/functions.json`.
You can check their contents.
We provide a script to calculate the size of this database and produce the Figure 9. Simply run:

.. code-block:: console

  $ cd /artifact/database
  $ ./get_stats.py

This script will print the number of functions in the json file and save a figure as `/artifact/database/database_hist.png`.
You can get a rough sense about this figure in the terminal by running:

.. code-block:: console

  $ chafa database_hist.png

If you want to view the figure clearly, you can copy it to your host machine by using `docker cp <https://docs.docker.com/reference/cli/docker/container/cp/>`_ command.


Bug-Finding (Section 5.2)
------------

Overall, you can find all bug details in `/artifact/bugs/bug_stat.json`, where we include the bug report links, bug states, affected versions, and symptoms.

We also include all bug-triggering testcases in `/artifact/bugs/testcases`. You can view the list of testcases by running

.. code-block:: console

  $ tree /artifact/bugs/testcases

In each bug directory, ``"orig.c"`` is the seed program, ``"case.c"`` is LegoFuzz-produced program, and ``"reduced.c"`` is the reduced bug-triggering test program. For some bugs, the filenames maybe a bit different but you should be able to know their purposes from the filenames.

Below we provide a set of scripts to extract information from `/artifact/bugs/bug_stat.json`.

First, shift to the working directory:

.. code-block:: console

  $ cd /artifact/bugs/


Number of bugs.
~~~~~~~~~~~~~~~~~~~~


You can reproduce Table 1 by running

.. code-block:: console

  $ ./gen_table_bug_summary.py

Types of bugs.
~~~~~~~~~~~~~~~~~~~~

We show in Table 2 that the number of crash and miscompilation bugs. You can reproduce this table by running

.. code-block:: console

  $ ./gen_table_bug_symptoms.py

Importance of bugs.
~~~~~~~~~~~~~~~~~~~~

We show the affected compiler versions in Figure 11. You can reproduce this figure by running

.. code-block:: console

  $ ./gen_figure_affected_versions.py

This script will save the figure into ``"bugs_affected_versions.png"`` and print the data, which should be consistent with Figure 9.
Again, you can get a rough sense about this figure in the terminal by running:

.. code-block:: console

  $ chafa bugs_affected_versions.png

If you want to view the figure clearly, you can copy it to your host machine by using `docker cp <https://docs.docker.com/reference/cli/docker/container/cp/>`_ command.



Affected compiler components.
~~~~~~~~~~~~~~~~~~~~

We show the number of bugs that affect each compiler components in Tables 3 and 4.
You can reproduce these two tables by running:

.. code-block:: console

  $ ./gen_affected_components.py

This script will extract the buggy commits of each bug from ``bug_stat.json`` and then check the affected components by querying the compiler repositories in ``"/compiler/repo/"``.

You're expected to see Loop Transformations and Peephole Optimizations are among the top ones, **demonstrating Claim 1**.

Code Coverage (Section 5.3, 5.4, 5.5 and 5.6)
------------

In Sections 5.3, 5.4, 5.5 and 5.6, we reported the code coverage of different approaches in Table 5 and Figure 12.
We here provide scripts to reproduce these data.

In the paper, all approaches ``Lego``, ``Lego-Seeds``, ``Lego-Functions``, ``Lego-1/4``, ``Lego-1/2``, ``Lego-iter_10``, ``Lego-iter_50``, ``Lego-iter_100``, ``Lego-iter_150``, ``Lego-iter_200``, ``Fuzz4All``, ``WhiteFox``, ``GPT-3.5`` and ``Qwen`` are generating 10,000 programs. 
This results in 10,000 programs for each of the approaches, except for ``Lego-Seeds``, which includes 1,000 seed programs. These seeds are used to generate 10 mutants each in the Lego pipeline.

We provide three evaluation modes:

1. **Direct Reproduction (fastest)**:

Original results are available in "/artifact/coverage/mutants-orig" and "/artifact/coverage/coverage_report-orig".
You can directly use these to reproduce the results and figures without re-running the pipeline.

2. **Small-Scale Reproduction**:

The full evaluation can take over 15 hours to complete. This mode runs at 1/10 scale—for example, generating 1,000 LegoFuzz programs instead of 10,000.
While minor differences from the paper's results may appear due to scale, the core insights remain consistent.
This mode typically takes around 5 hours.

3. **Full Reproduction**:

This mode re-executes the entire generation pipeline from scratch and replicates the original experimental setup in full.

.. note::
You may choose any of these modes based on your time and resource constraints. 
We have provided an OpenRouter API key for artifact evaluation in ``"/artifact/coverage/scripts/.env"``.

Direct Reproduction
~~~~~~~~~~~~~~~~~~~

We have included all the original mutants in ``"/artifact/coverage/mutants-orig"`` and the original coverage data in ``"/artifact/coverage/coverage_report-orig"``.
You can check them out for evaluation. 

**Generate Figure 12 by running:**

.. code-block:: console

  $ cd /artifact/coverage
  $ ./generate_figure_cov.py

The coverage data will be printed out and the figure ``"fig_line_cov.png"`` will be generated. 
You're expected to observe that **LegoFuzz achieves the highest coverage**, **demonstrating Claim 2**.
Also, you're expected to see the increase from Lego-1_4 to Lego-1_2,  **demonstrating Claim 4**. 

Small-Scale Reproduction and Full Reproduction
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The only difference between these two modes is the option ``"--small"``, which enables Small-Scale Reproduction.

**We now assume that you are in the Small-Scale Reproduction stage.**

First, generate mutant programs with each approach:

.. code-block:: console

  $ cd /artifact/coverage
  $ ./generate_all_mutants.py --cpu 64 --small # < 1 hour.

The results will be stored in ``"/artifact/coverage/mutants"``. 

.. warning::

   This script does **not** generate mutants for Fuzz4All and WhiteFox, as running those tools require a GPU with **at least 48 GB** of memory to run.  
   However, we provide an OpenRouter API key to support reproduction, which is under ``"coverage/scripts/.env"``.

   We also include **modified versions** of Fuzz4All and WhiteFox that support using OpenRouter to invoke APIs.  
   You can follow the provided README files in their respective directories to reproduce the results.

   For Fuzz4All, a configuration file is available at  
   ``/artifact/generators/fuzz4all/config/full_run/c_std_eval_for_legofuzz.yaml``  
   This configuration does not modify any core components or prompts.  
   The only change is switching the default prompt strategy from 2 to 0 to enable the  
   **"generate new code using separator"** option.  
   You can verify this by comparing ``c_std_eval_for_legofuzz.yaml`` with ``c_std.yaml``.

   For both tools, we provide **cached results** and **complete log files** generated during execution.  
   You can find them under ``"/artifact/coverage/mutants-orig"`` and ``/artifact/generators/logs``. 

**Second, get code coverage by running:**

.. code-block:: console

  $ ./analyze_all_coverage.py --cpu 64 --new  # ~4-5 hours. 

.. note::

   With 64 cores (``"--cpu 64"``), the script takes roughly 4 hours to finish.

This script will compile the mutant programs with GCC and LLVM, then analyze the compiler coverage.
The result coverage json files will be saved into ``"/artifact/coverage/coverage_report/"``. 

**Third, produce Figure 12 by running:**

.. code-block:: console

  $ ./generate_figure_cov.py --new

The coverage data will be printed out and the figure ``"fig_line_cov.png"`` will be generated. 
you can get a rough sense about this figure in the terminal by running:

.. code-block:: console

  $ chafa fig_line_cov.png

If you want to view the figure clearly, you can copy it to your host machine by using `docker cp <https://docs.docker.com/reference/cli/docker/container/cp/>`_ command.

You're expected to observe that **LegoFuzz achieves the highest coverage**, **demonstrating Claim 2**.
Also, you're expected to see the increase from Lego-1_4 to Lego-1_2,  **demonstrating Claim 4**. 


Generation Speed (Section 5.3 and Section 5.7)
------------

In Section 5.4, we claimed that LegoFuzz can produce mutants in a speed of "an average of 0.02 seconds per mutant". To verify that, running:

.. code-block:: console

  $ cd /artifact/speed
  $ ./generate_lego_mutants.py

This script will use a single core and invoke LegoFuzz to generate 10,000 mutants to ``"/artifact/speed/mutants/"``.
Total number of mutants, total time and calculated average time will be print out, **demonstrating Claim 3**.

Iteration Number (Section 5.6)
------------

In Section 5.6, we claimed that iteration number can affect the generation of LegoFuzz.
To generate Figure 13 and Figure 14, running:

.. code-block:: console

  $ cd /artifact/iteration
  $ ./generate_fig_iter_loc.py 
  $ ./generate_fig_iter.py 

.. note::
  The above commands use our original results to draw figures.
  If you want to use your own generated results from Small-Scale Reproduction or Full Reproduction, add ``"--new"`` like ``"./generate_fig_iter_loc.py --new"``.

This script will save the figure into ``"fig_iter_trend.png"`` and ``"fig_iter.png"`` , which should be consistent with Figure 13 and Figure 14.
Again, you can get a rough sense about this figure in the terminal by running:

.. code-block:: console

  $ chafa fig_iter_trend.png
  $ chafa fig_iter.png

If you want to view the figure clearly, you can copy it to your host machine by using `docker cp <https://docs.docker.com/reference/cli/docker/container/cp/>`_ command.

Congratulations! You have successfully finished all the main experiments.
~~~~~~~~~~~~~~~~~~~~

**If you want to use LegoFuzz to generate new programs, goto** :doc:`/legofuzz`