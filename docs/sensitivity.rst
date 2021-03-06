Global Sensitivity Analysis
===========================

1. **Prepare the model**

   Sterope finds which variables will be analyzed using
   the symbol ``//`` (doble slash, as C/C++) followed by:

	* Type of bound: ``range`` or ``factor``
	* Bounds for each parameter: ``[min max]``. In the case of ``factor``,
	  the original parameter value is multiply by (1 - min) and (1 + max),
	  therefor, ``min`` and ``max`` should by between zero and one.

   For instace, the Thomas' model was configured as:

.. code-block:: c

	%var: 'KD1__FREE__' 1.000000e+00 // range[0.01 100]
	%var: 'km1__FREE__' 1.000000e+00 // range[0.01 100]
	%var: 'K2RT__FREE__' 1.000000e+00 // range[0.01 100]
	%var: 'km2__FREE__' 1.000000e+00 // range[0.01 100]
	%var: 'kphos__FREE__' 1.000000e+00 // range[0.01 100]
	%var: 'kdephos__FREE__' 1.000000e+00 // range[0.01 100]

or the following configuration if the model is written in syntax 3:

.. code-block:: perl

	%var: 'KD1__FREE__' 1.000000e+00 # range[0.01 100]
	%var: 'km1__FREE__' 1.000000e+00 # range[0.01 100]
	%var: 'K2RT__FREE__' 1.000000e+00 # range[0.01 100]
	%var: 'km2__FREE__' 1.000000e+00 # range[0.01 100]
	%var: 'kphos__FREE__' 1.000000e+00 # range[0.01 100]
	%var: 'kdephos__FREE__' 1.000000e+00 # range[0.01 100]

.. note::
	Sterope is compatible with KaSim4 only. The software could simulate
	*kappa* models written in syntax 3 (See below option ``syntax``).

2. **Configure Sterope**

	Write a ``script.sh`` file with the following, configured with a number of processes
	that will not interfere with other user's plans to lunch jobs in a cluster. Also and only
	if the job is launched outside SLURM (see below), the number of processess could not be greater than
	the number of cores the node (your laptop, desktop, or server) has.

.. code-block:: bash

	#!/bin/sh

	#SBATCH --no-requeue
	#SBATCH --partition=cpu

	#SBATCH --nodes=1
	#SBATCH --ntasks=1
	#SBATCH --cpus-per-task=1

	#SBATCH --job-name=sterope
	#SBATCH --output=stdout.txt
	#SBATCH --error=stderr.txt

	export PYTHONPATH="$PYTHONPATH:$HOME/opt/sterope"

	MODEL=pysbmodel-example6-kasim.kappa
	FINAL=660
	STEPS=10 # KaSim4 interprets as the period, not how many points to report!

	NUM_LEVELS=1000

	python3 -m sterope.kasim --model=$MODEL --final=$FINAL --tmin=600 --steps=$STEPS \
	--grid=$NUM_LEVELS --kasim=kasim4 --syntax 3 --method sobol --nprocs 250 --memory 2000MB

3. **Execute Sterope**

.. code-block:: bash

	sbatch script.sh

OR

.. code-block:: bash

	nohup sh script.sh &

.. note::
	The Dynamic Influence Network changes over the course of a simulation. To report time window
	smaller than the simulation time, set ``--tmin float`` as the starting point and ``--tmax float``
	as the finish point. The ``--tmin`` default is zero and ``--tmax`` default is ``--final``.

.. note::
	**Sensitivity methods** type ``python3 -m sterope.kasim --help`` to find out the
	available methods. Sobol is the default and we implemented all the available in the SALib
	python package.

.. note::
	``--memory`` is not necessary when running sterope without SLURM. Also, the ``#SBATCH`` directives
	are omited when executing the script wiht ``sh`` so they are safe to leave.
