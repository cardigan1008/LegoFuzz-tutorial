Reusability Guide: Run the tool-LegoFuzz
========

**LegoFuzz** is an LLM-based fuzzing framework. It currently supports testing C compilers, such as GCC and LLVM.

The sourcecode of Creal is located in ``/artifact/generators/LegoFuzz/``.
Here is the detailed explanation of core files:

.. code-block:: console

    ├── synthesize.py         # For iterative program synthesis
    ├── fuzz.py               # For conducting fuzzing
    ├── transformer           # LLMs-based real code-aligned code generation
    │   ├── config            # Configuration for LLMs
    │   └── generate.py       # For generating code with LLMs
    ├── databaseconstructor   # Constructing function database
    │   ├── functionextractor  
    │   │   └── extract.py    # For extracting valid functions
    │   └── generate.py       # For generating IO for functions
    ├── profiler              
    │   └── profile.py        # For profiling functions
    └── utils                 # Development utilities

Quick Start: Direct Program Synthesis
-------------------------------------

To synthesize programs directly using a pre-built function database, run:

.. code-block:: shell

    ./synthesize.py --src functions.json --dst ./tmp --iter 10

This generates a synthesized program in ``./tmp`` using 10 iterations.   

Full Workflow: From Code Generation to Fuzzing
----------------------------------------------

Phase 1: Offline Code Database Construction
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

**Step 1: Real Code-aligned Generation with LLMs**

All model and prompt configurations are defined in ``transformer/config/config.yaml``.

Start by setting your API key:

.. code-block:: shell

    cd transformer
    echo "<API_KEY_NAME>=<API_KEY_SECRET>" > .env

Supported API keys:

- ``OPENAI_API_KEY``
- ``TOGETHER_API_KEY``
- ``DEEPSEEK_API_KEY``

.. note::
   To integrate other providers (e.g., OpenRouter), modify ``transformer/config/models.py`` and set  
   ``base_url="https://openrouter.ai/api/v1"`` in the client initialization.

   If you are using a **local LLM**, you can simply implement a method  
   ``create_chat_completion(self, messages, **kwargs)``  
   that returns a response in the same format as OpenAI's API. This allows you to seamlessly plug in local models without changing the rest of the pipeline.

To generate aligned C code:

.. code-block:: shell

    ./generate.py --src <DIR_SRC> --dst <DIR_DST> openai

Parameters explanation:

::

    --src SRC             Path to the source directory containing C files
    --dst DST             Directory to save generated C files
    --model {openai,deepseek,togetherai}
                          Which LLM model to use
    --max_files MAX_FILES
                          Maximum number of C files to process (Optional)

**Step 2: Construct the Function Database**

Follow these steps to extract and process functions from generated code.

Extract functions:

.. code-block:: shell

    cd databaseconstructor/functionextractor
    ./extract.py --src <DIR_C_FILES> --dst ./functions.json

Generate input/output pairs:

.. code-block:: shell

    cd ..
    ./generate.py --src functions.json --dst ./functions_io.json

Profile the functions:

.. code-block:: shell

    cd ../profiler
    ./profile.py --src ../databaseconstructor/functions_io.json --dst ./functions_profiled.json

.. note::
   If duplicate function names exist, run: ``./dedup.py functions_profiled.json``

At this point, you have a fully profiled function database.

Phase 2: Online Iterative Program Synthesis
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

With a profiled database (e.g., ``profiler/functions_profiled.json``), run:

.. code-block:: shell

    ./synthesize.py --src profiler/functions_profiled.json --dst ./tmp --prob 80 --num_mutant 10 --iter 100

Parameters explanation:

::

    --src SRC                Path to the function database json file.
    --dst DST                Path to the destination dir.
    --prob PROB              Probability of replacing an expression (default=80).
    --num_mutant NUM_MUTANT  Number of mutants to generate (default=1).
    --iter ITER              Number of iterations for one synthesis (default=100).
    --no-rand                Randomize the number of iterations.
    --inline                 Inline the function call.
    --debug                  Print debug information.

Fuzzing Execution
~~~~~~~~~~~~~~~~~

Configure the compiler settings by copying the example file:

.. code-block:: shell

    cp compilers.in.example compilers.in

Then edit ``compilers.in`` to list the compiler commands to test, for example:

::

    gcc -O0
    gcc -O1

Start fuzzing:

.. code-block:: shell

    ./fuzz.py --cpu 4 --config compilers.in

This launches fuzzing using 4 CPU cores. Synthesized mutants will be tested, and bugs will be saved under the ``bugs`` directory. Intermediate results will appear under ``fuzz``.

Parameters explanation:

::

    --cpu CPU        Number of CPUs to run in parallel (default: all available cores)
    --config CONFIG  Path to compiler config file (default: ./compilers.in)
