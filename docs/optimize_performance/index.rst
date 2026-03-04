.. _optimize-performance:

Optimize your Cosmos Performance
================================

You can adjust how Cosmos works in your project setup to improve overall performance, and maximize the speed and resource use of Dag and dbt command execution.

#--- dbt command execution
#-- ~~~~~~~~~~~~~~~~~~~~~
#--- This section does not have any existing content to include yet.

Cosmos Dag-parsing
~~~~~~~~~~~~~~~~~~

Selecting the methods that Cosmos uses to load your dbt project can improve how fast code executes. Enabling partial-parsing or using other methods to load project information can improve parsing performance.

dbt core optimization
---------------------

In addition to choosing how to parse Dags, working with ``selector`` and ``run dbt ls`` in invocation mode can also improve parsing and execution speed.

Caching
~~~~~~~

You have control over what kind of artifacts you want to cache, depending on your Cosmos version. Working with a caching strategy improves performance by not requiring Cosmos to parse your project data with every run, instead, only processing changes.

.. toctree::
   :maxdepth: 1
   :hidden:
   :caption: Optimize Performance

   partial-parsing
   memory_optimization
   selecting-excluding
   invocation_modes
   caching
