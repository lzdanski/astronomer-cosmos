Troubleshooting
===============

.. toctree::
   :maxdepth: 1
   :hidden:
   :caption: Troubleshoot Cosmos

   self
   execution-modes-local-conflicts

Common issues
~~~~~~~~~~~~~

I cannot see my DbtDag
-----------------------

1. Check to see if your Dag files contain the words ``Dag`` and ``Airflow``.
2. Set the Airflow environment variable, ``AIRFLOW__CORE__DAG_DISCOVERY_SAFE_MODE=False``.

I still cannot see my DbtDags
-----------------------------

1. Find if you are using ``LoadMode.AUTOMATIC`` (default) or ``LoadMode.DBT_LS``.
2. Increase the ``AIRFLOW__CORE__DAGBAG_IMPORT_TIMEOUT``.
3. Check the Dag processor/ scheduler logs for errors.

The performance is suboptimal due to latency or resource utilization
--------------------------------------------------------------------

1. Try using the latest Cosmos release.
2. Leverage Cosmos `caching <../../optimize_performance/caching.html>`_ mechanisms.
3. For very large dbt pipelines, use recommended ``LoadMode.DBT_MANIFEST``.
4. Pre-install dbt deps in your Airflow environment.
5. If possible, use ``ExecutionMode.LOCAL`` or ``InvocationMode.DBT_RUNNER``.

Improve Dag parsing
~~~~~~~~~~~~~~~~~~~

To reduce the Dag parsing time:

1. Precompute the ``manifest.json`` as part of the CI. This might not be applicable for some projects, but you can complete by running ``dbt ls``.
2. Run ``dbt deps`` as part of the CI, but because this duplicates the process in Cosmos, be sure to disable it in Cosmos.
3. Select a subset of nodes for large projects.
4. Consider using the `build test behavior <../../guides/translate_dbt_to_airflow/configure-tests/testing-behavior.html>`_ to have a single node per model (& its tests)
5. dbt Core optimization, like using dbt as a library instead of a binary with `RenderConfig <../../guides/translate_dbt_to_airflow/render-config.html>`_. Use this to install dbt and its adapters in the ``requirements.txt``, but this might not work depending on dependency conflicts.
6. Use Cosmos native operators to group non-critical parts of the pipeline
7. Try to use `dbt Fusion <../../guides/dbt_setup/dbt-fusion.html>`_ with Cosmos, which can improve the performance of larger dbt projects.

Improve task execution time
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

To reduce the task execution time,in addition to the Dag parsing optimizations:

1. Use the `airflow_async execution mode <../../guides/run_dbt/airflow-worker/async-execution-mode.html>`_. This mode is currently only available for BigQuery.
2. Use the `local execution mode <../../guides/run_dbt/airflow-worker/local-execution-mode.html>`_.
3. Run ``dbt deps`` as part of the CI, but because this duplicates the process in Cosmos, be sure to disable it in Cosmos.
4. dbt Core optimization, like using dbt as a library instead of a binary with `RenderConfig <../../guides/translate_dbt_to_airflow/render-config.html>`_. Use this to install dbt and its adapters in the ``requirements.txt``, but this might not work depending on dependency conflicts.
5. For non-critical parts of the pipeline, consider grouping tasks by instantiating Cosmos operators.
6. Try to use `dbt Fusion <../../guides/dbt_setup/dbt-fusion.html>`_ with Cosmos, which can improve the performance of larger dbt projects.

Set up dbt core installation
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The following workflow can help you decide how to install dbt core for your Cosmos implementation.

1. Determine if you can install dbt and Airflow in the same Python environment
-------------------------------------------------------------------------------

If you can, no cadditional onfiguration is needed.

By default, Cosmos uses:

- ``ExecutionMode.LOCAL``
- ``InvocationMode.DBT_RUNNER``

.. code-block:: python

   FROM quay.io/astronomer/astro-runtime:13.4.0

   RUN python -m venv dbt_venv && \
      source dbt_venv/bin/activate && \
      pip install --no-cache-dir<your-dbt-adapter> && \
      deactivate

2. Decide if you can create and manage a dedicated Python environment alongside Airflow
---------------------------------------------------------------------------------------

To use a dedicated Python virtual environment, you might need to configure two additional steps:

1. If using Astro, create the virtualenv as part of your Docker image build.
2. Define where the dbt binary for Cosmos to use. This still uses the default ``ExecutionMode.LOCAL``:

.. code-block:: python

   FROM quay.io/astronomer/astro-runtime:13.4.0

   RUN python -m venv dbt_venv && \
      source dbt_venv/bin/activate && \
      pip install --no-cache-dir<your-dbt-adapter> && \
      deactivate
   DbtDag(
   ...,
   execution_config=ExecutionConfig(
         dbt_executable_path=Path("/usr/local/airflow/dbt_venv/bin/dbt")
         operator_args={“py_requirements": ["dbt-postgres==1.6.0b1"]}
   ))

3. Decide if you want to install dbt in Airflow nodes
-----------------------------------------------------

If you want to install dbt in Airflow nodes, Cosmos can create and manage the dbt Python virtualenv for you with ``ExecutionMode.VIRTUALENV``.

.. code-block:: python

   FROM quay.io/astronomer/astro-runtime:13.4.0

   RUN python -m venv dbt_venv && \
      source dbt_venv/bin/activate && \
      pip install --no-cache-dir<your-dbt-adapter> && \
      deactivate
   DbtDag(
   ...,
   execution_config=ExecutionConfig(
         execution_mode=ExecutionMode.VIRTUALENV,
   )
   )

4. Use Cosmos without dbt in the Airflow nodes
----------------------------------------------

You don’t have to use dbt in the Airflow nodes to benefit from Cosmos. Instead you can leverage ``LoadMode.DBT_MANIFEST`` or ``ExecutionMode.KUBERNETES``.

.. code-block:: python

   FROM quay.io/astronomer/astro-runtime:13.4.0

   RUN python -m venv dbt_venv && \
      source dbt_venv/bin/activate && \
      pip install --no-cache-dir<your-dbt-adapter> && \
      deactivate
   DbtDag(
   ...,
   execution_config=ExecutionConfig(
         execution_mode=ExecutionMode.VIRTUALENV,
   )
   )

Set up dbt deps
~~~~~~~~~~~~~~~

1. Determine if you can pre-install dbt dependencies in your Airflow deployment.

If you can, this is the most efficient approach. You can tell Cosmos to ignore the dbt ``packages.yml`` file.

.. code-block:: python

   DbtDag(
  ...,
  render_config=RenderConfig(dbt_deps=False),
  execution_config=ExecutionConfig(
      operator_args={"install_deps": False}
  ))

If you can't pre-install dbt dependencies, Cosmos automatically runs dbt deps before running any dbt command, and does not require any additional configuration.

Set up database connections
~~~~~~~~~~~~~~~~~~~~~~~~~~~

If you manage Airflow database credentials, Cosmos has an extensible set of ``ProfileMapping`` classes, that can automatically create the dbt ``profiles.yml`` from Airflow Connections.

.. code-block:: yaml

   profile_config = ProfileConfig(
    profile_name="my_profile_name",
    target_name="my_target_name",
    profile_mapping=SnowflakeUserPasswordProfileMapping(
        conn_id="my_snowflake_conn_id",
        profile_args={
            "database": "my_snowflake_database",
            "schema": "my_snowflake_schema",
         },
      ),
   )
   dag = DbtDag(
      profile_config=profile_config,
   )

If you don't manage Airflow database credentials, Cosmos also allows you to define your own profiles.yml.

.. code-block:: yaml

   profile_config = ProfileConfig(
    profile_name="my_snowflake_profile",
    target_name="dev",
    profiles_yml_filepath="/path/to/profiles.yml",
   )


