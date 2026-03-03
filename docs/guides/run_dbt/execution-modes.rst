.. _execution-modes:

Execution Modes
===============

*Execution modes* describe *where* and *how* Cosmos runs dbt commands.

On the Airflow worker or triggerer
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

- `local <airflow-worker/local-execution-mode.html>`_: Run ``dbt`` commands using a local ``dbt`` installation (default)
- `watcher <airflow-worker/watcher-execution-mode.html>`_: (experimental since Cosmos 1.11.0) Run a single ``dbt build`` command from a producer task and have sensor tasks to watch the progress of the producer, with improved DAG run time while maintaining the tasks lineage in the Airflow UI, and ability to retry failed tasks.
- `virtualenv <airflow-worker/cosmos-managed-venv.html>`_: Run ``dbt`` commands from Python virtual environments managed by Cosmos
- `airflow_async <airflow-worker/async-execution-mode.html>`_: (stable since Cosmos 1.9.0) Run the dbt resources from your dbt project asynchronously, by submitting the corresponding compiled SQLs to Apache Airflow's `Deferrable operators <https://airflow.apache.org/docs/apache-airflow/stable/authoring-and-scheduling/deferring.html>`__

In a container
~~~~~~~~~~~~~~

You can also execute dbt commands in a container outside of the Airflow environment.

- `docker <container/docker.html>`_ : Run ``dbt`` commands from Docker containers managed by Cosmos (requires a pre-existing Docker image)
- `kubernetes <container/kubernetes.html>`_: Run ``dbt`` commands from Kubernetes Pods managed by Cosmos (requires a pre-existing Docker image)
- `watcher_kubernetes <container/watcher-kubernetes-execution-mode.html>`_: (experimental since Cosmos 1.13.0) Combines the speed of the watcher execution mode with the isolation of Kubernetes. Check the :ref:`watcher-kubernetes-execution-mode` for more details.
- `azure_container_instance <container/azure-container-instance>`_: Run ``dbt`` commands from Azure Container Instances managed by Cosmos (requires a pre-existing Docker image)
- `aws_ecs <container/aws-container-run-job>`_: Run ``dbt`` commands from AWS ECS instances managed by Cosmos (requires a pre-existing Docker image)
- `aws_eks <container/aws-eks.html>`_: Run ``dbt`` commands from AWS EKS Pods managed by Cosmos (requires a pre-existing Docker image)
- `gcp_cloud_run_job <container/gcp-cloud-run-job>`_: Run ``dbt`` commands from GCP Cloud Run Job instances managed by Cosmos (requires a pre-existing Docker image)

The choice of the ``execution mode`` can vary based on your needs and concerns.

.. _execution-modes-comparison:

Execution modes comparison
~~~~~~~~~~~~~~~~~~~~~~~~~~

.. list-table:: Execution Modes Comparison
   :widths: 25 25 25 25
   :header-rows: 1

   * - Execution Mode
     - Task Duration
     - Environment Isolation
     - Cosmos Profile Management
   * - Local
     - Fast
     - None
     - Yes
   * - Watcher
     - Very Fast
     - None
     - Yes
   * - Virtualenv
     - Medium
     - Lightweight
     - Yes
   * - Airflow Async
     - Very Fast
     - Medium
     - Yes
   * - Docker
     - Slow
     - Medium
     - No
   * - Kubernetes
     - Slow
     - High
     - No
   * - AWS_EKS
     - Slow
     - High
     - No
   * - Azure Container Instance
     - Slow
     - High
     - No
   * - GCP Cloud Run Job Instance
     - Slow
     - High
     - No
   * - AWS ECS
     - Slow
     - High
     - No
   * - Watcher Kubernetes
     - Fast
     - High
     - No

