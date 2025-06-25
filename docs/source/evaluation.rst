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

   - ``ctr`` + ``b``

   - ``d``

   To resume the session:

   .. code-block:: console

      $ tmux at -t eval   # resume a tmux session.



Experimental Setup (Section 5.1)
------------

In the paper, we claimed that ``"we collected a function database of 51,356 functions"``
and we showed the statistics in Figure 8.

The function database is located in `/artifact/database/functions.json`.
You can check their contents.
We provide a script to calculate the size of this database and produce the Figure 8. Simply run:

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

In each bug directory, ``"orig.c"`` is the seed program, ``"case.c"`` is Creal-produced program, ``"reduced.c"`` is the reduced bug-triggering test program, and ``"removed.c"`` is the reduced ``"case.c"`` by removing unnecessary functions. For some bugs, the filenames maybe a bit different but you should be able to know their purposes from the filenames.

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

We show the affected compiler versions in Figure 9. You can reproduce this figure by running

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

Bug Characteristics (Section 5.3)
------------

We show the number of inserted functions in each bug-triggering testcases in Figure 10.
You can reproduce Figure 10 by running the following script, which analyzes each ``removed.c`` in each bug testcase `/artifact/bugs/testcases`

.. code-block:: console

  $ cd /artifact/characteristics/
  $ ./num_inserted.py

This script will also print the data, which is consistent to the produced figure ``figure_num_inserted_functions.png``.

For the unique features in bug-triggering functions, we did manual analysis on each ``removed.c``.


Code Coverage (Section 5.4, 5.5, and 5.6)
------------

In Sections 5.4, 5.5, and 5.6, we reported the code coverage of different approaches in Table 5 and Figure 11.
We here provide scripts to reproduce these data.

In the paper, all approaches ``Creal``, ``Creal-1/4``, ``Creal-1/2``, ``Creal-Csmith``, and ``Hermes`` are generating 10 mutants from 1000 seed programs.
This results in 10,000 programs for each of the approaches.

The 1000 seeds are located in ``"/artifact/coverage/seeds_csmith_full"``.
Fully reproduce the coverage using 1000 seeds requires >= 17 hours on a 64-core machine.

To reduce the evaluation time, we prodive a small set of seeds (100) for quickly validating the results (~3 hours).
This small set of seeds are in ``"/artifact/coverage/seeds_csmith_small"``.

It is upto you to select which seeds for evaluation. 
Using full seed set can reproduce data in the paper while requiring more than 10 hours of evaluation.
Using small seed set can still get the key message as in the paper, **i.e., Creal achieves the highest coverage**.

**We now assume that you are using the small seed set. First, generate mutant programs with each approach:**

.. code-block:: console

  $ cd /artifact/coverage/
  $ ./generate_all_mutants.py --cpu 64 --small   # ~10 minutes

This script will invoke a set of scripts in ``"/artifact/coverage/scripts/"`` to generate mutants with each approach.
The generated mutants will be saved into ``"/artifact/coverage/mutants/"``.

**Second, get code coverage by running:**

.. code-block:: console

  $ ./analyze_all_coverage.py --cpu 64  # ~3-4 hours if used `--small` above, otherwise ~10-15 hours.

.. note::

   With 64 cores (`"`--cpu 64"``), the script takes roughly 3 hours to finish.
   Change ``"--small"`` to ``"--full"`` to generate mutants on the 1000 seeds (~10-15 hours).


This script will compile the mutant programs with GCC and LLVM, then analyze the compiler coverage.
The result coverage json files will be saved into ``"/artifact/coverage/coverage_report/"``.

**Third, produce Table 5 and Figure 11 by running:**

.. code-block:: console

  $ ./generate_coverage_table.py
  $ ./generate_figure_cov.py

The coverage table and data will be printed out. 
Because we do not use the full set for evaluation, the resulying figures in the table will be different from the paper.
However, you're expected to observe that **Creal achieves the highest coverage**.

You can also use the full set for evaluation, but due to randomness, number differences (< 5%) are expected.


Generation Speed (Section 5.4)
------------

In Section 5.4, we claimed that Creal can produce mutants in a speed of ``"an average of 0.87 seconds per mutant"``.
To verify that, running:

.. code-block:: console

  $ cd /artifact/speed
  $ ./generate_creal_mutants.py

This script will use a single core and invoke Creal to generate 10 mutants on each program in ``"/artifact/speed/seeds/"``.


Congratulations! You have successfully finished all the main experiments.
~~~~~~~~~~~~~~~~~~~~

**If you want to use Creal to generate new programs, goto** :doc:`/legofuzz`