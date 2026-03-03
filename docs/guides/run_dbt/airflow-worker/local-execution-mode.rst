.. _local-execution:

Local Execution Mode
====================

By default, Cosmos uses the ``local`` execution mode.

The ``local`` execution mode is the fastest way to run Cosmos operators since they don't neither install ``dbt`` nor build docker containers. If you use managed Airflow services such as
Google Cloud Composer, you might want to use a different execution mode, since Airflow and ``dbt`` dependencies can conflict (:ref:`execution-modes-local-conflicts`). On an managed Airflow service, you you might not be able to install ``dbt`` in a custom path.

The ``local`` execution mode assumes that the Airflow worker node can access a ``dbt`` binary.

If ``dbt`` was not installed as part of the Cosmos packages, you can define a custom path to ``dbt`` by declaring the argument ``dbt_executable_path``.

.. note::
    Starting in the 1.4 version, Cosmos tries to leverage the dbt partial parsing (``partial_parse.msgpack``) to speed up task execution.
    This feature is bound to `dbt partial parsing limitations <https://docs.getdbt.com/reference/parsing#known-limitations>`_.
    Learn more: :ref:`partial-parsing`.

When using the ``local`` execution mode, Cosmos converts Airflow Connections into a native ``dbt`` profiles file (``profiles.yml``).

Example of how to use, for instance, when ``dbt`` was installed together with Cosmos:

.. literalinclude:: ../../../../dev/dags/basic_cosmos_dag.py
    :language: python
    :start-after: [START local_example]
    :end-before: [END local_example]

.. _execution-modes-local-conflicts:

Airflow and dbt dependencies conflicts
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

When using the local execution mode, you can encounter dependency conflicts between
`Apache Airflow® <https://airflow.apache.org/>`_ and dbt. The conflicts might increase depending on the Airflow providers and dbt adapters you use.

If you find errors, we recommend you isolate the installation of dbt from the Airflow installation. For your Local execution mode, install dbt in a separate
Python virtualenv and set the `ExecutionConfig.dbt_executable_path <../reference/configs/execution-config.html>`_  and
`RenderConfig.dbt_executable_path <../reference/configs/render-config.html>`_ parameters.

Cosmos includes many other `execution mode methods <execution-modes.html>`__ that support isolating dbt from Airflow.

In the following table, ``x`` represents combinations that lead to conflicts (vanilla ``apache-airflow`` and ``dbt-core`` packages):

+---------------+-----+-----+-----+-----+-----+-----+-----+-----+-----+-----+------+
| Airflow / DBT | 1.0 | 1.1 | 1.2 | 1.3 | 1.4 | 1.5 | 1.6 | 1.7 | 1.8 | 1.9 | 1.10 |
+===============+=====+=====+=====+=====+=====+=====+=====+=====+=====+=====+======+
| 2.2           |     |     |     | x   | x   | x   | x   | x   | x   | x   |  x   |
+---------------+-----+-----+-----+-----+-----+-----+-----+-----+-----+-----+------+
| 2.3           | x   | x   |     | x   | x   | x   | x   | x   | x   | x   |  x   |
+---------------+-----+-----+-----+-----+-----+-----+-----+-----+-----+-----+------+
| 2.4           | x   | x   | x   |     |     |     |     |     |     |     |      |
+---------------+-----+-----+-----+-----+-----+-----+-----+-----+-----+-----+------+
| 2.5           | x   | x   | x   |     |     |     |     |     |     |     |      |
+---------------+-----+-----+-----+-----+-----+-----+-----+-----+-----+-----+------+
| 2.6           | x   | x   | x   | x   | x   |     |     |     |     |     |      |
+---------------+-----+-----+-----+-----+-----+-----+-----+-----+-----+-----+------+
| 2.7           | x   | x   | x   | x   | x   |     |     |     |     |     |      |
+---------------+-----+-----+-----+-----+-----+-----+-----+-----+-----+-----+------+
| 2.8           | x   | x   | x   | x   | x   |     |  x  |     |     |     |  x   |
+---------------+-----+-----+-----+-----+-----+-----+-----+-----+-----+-----+------+
| 2.9           | x   | x   | x   | x   | x   |     |     |     |     |     |      |
+---------------+-----+-----+-----+-----+-----+-----+-----+-----+-----+-----+------+
| 2.10          | x   | x   | x   | x   | x   |     |     |     |     |     |      |
+---------------+-----+-----+-----+-----+-----+-----+-----+-----+-----+-----+------+
| 2.11          | x   | x   | x   | x   | x   |     |     |     |     |     |      |
+---------------+-----+-----+-----+-----+-----+-----+-----+-----+-----+-----+------+
| 3.0           | x   | x   | x   | x   | x   | x   | x   | x   |     |     |  x   |
+---------------+-----+-----+-----+-----+-----+-----+-----+-----+-----+-----+------+

Examples of errors
------------------

.. code-block:: bash

  The conflict is caused by:
    apache-airflow 2.8.0 depends on pydantic>=2.3.0
    dbt-semantic-interfaces 0.4.2 depends on pydantic~=1.10
    apache-airflow 2.8.0 depends on pydantic>=2.3.0
    dbt-semantic-interfaces 0.4.2.dev0 depends on pydantic~=1.10
    apache-airflow 2.8.0 depends on pydantic>=2.3.0
    dbt-semantic-interfaces 0.4.1 depends on pydantic~=1.10
    apache-airflow 2.8.0 depends on pydantic>=2.3.0
    dbt-semantic-interfaces 0.4.0 depends on pydantic~=1.10


.. code-block:: bash

  ERROR: Cannot install apache-airflow==2.2.4 and dbt-core==1.5.0 because these package versions have conflicting dependencies.
  The conflict is caused by:
    apache-airflow 2.2.4 depends on jinja2<3.1 and >=2.10.1
    dbt-core 1.5.0 depends on Jinja2==3.1.2

.. code-block:: bash

  ERROR: Cannot install apache-airflow==2.6.0 and dbt-core because these package versions have conflicting dependencies.
  The conflict is caused by:
    apache-airflow 2.6.0 depends on importlib-metadata<5.0.0 and >=1.7; python_version < "3.9"
    dbt-semantic-interfaces 0.1.0.dev7 depends on importlib-metadata==6.6.0

.. code-block:: bash

    ERROR: Cannot install apache-airflow, apache-airflow==2.7.0 and dbt-core==1.4.0 because these package versions have conflicting dependencies.

    The conflict is caused by:
        dbt-core 1.4.0 depends on pyyaml>=6.0
        connexion 2.12.0 depends on PyYAML<6 and >=5.1
        dbt-core 1.4.0 depends on pyyaml>=6.0
        connexion 2.11.2 depends on PyYAML<6 and >=5.1
        dbt-core 1.4.0 depends on pyyaml>=6.0
        connexion 2.11.1 depends on PyYAML<6 and >=5.1
        dbt-core 1.4.0 depends on pyyaml>=6.0
        connexion 2.11.0 depends on PyYAML<6 and >=5.1
        apache-airflow 2.7.0 depends on jsonschema>=4.18.0
        flask-appbuilder 4.3.3 depends on jsonschema<5 and >=3
        connexion 2.10.0 depends on jsonschema<4 and >=2.5.1

.. code-block:: bash

    ERROR: Cannot install apache-airflow and dbt-core==1.10.0 because these package versions have conflicting dependencies.

    The conflict is caused by:
        dbt-core 1.10.0 depends on pydantic<2
        apache-airflow-core 3.0.0 depends on pydantic>=2.11.0



How to reproduce
----------------

The table was created by running  `nox <https://nox.thea.codes/en/stable/>`__ with the following ``noxfile.py``:

.. code-block:: python

    import nox

    nox.options.sessions = ["compatibility"]
    nox.options.reuse_existing_virtualenvs = True


    @nox.session(python=["3.10"])
    @nox.parametrize(
        "dbt_version",
        ["1.0", "1.1", "1.2", "1.3", "1.4", "1.5", "1.6", "1.7", "1.8", "1.9", "1.10"],
    )
    @nox.parametrize(
        "airflow_version",
        ["2.2.4", "2.3", "2.4", "2.5", "2.6", "2.7", "2.8", "2.9", "2.10", "2.11", "3.0"],
    )
    def compatibility(session: nox.Session, airflow_version, dbt_version) -> None:
        """Run both unit and integration tests."""
        session.run(
            "pip3",
            "install",
            "--pre",
            f"apache-airflow=={airflow_version}",
            f"dbt-core=={dbt_version}",
        )