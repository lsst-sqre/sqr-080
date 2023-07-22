:tocdepth: 1

.. sectnum::

.. Metadata such as the title, authors, and description are set in metadata.yaml

.. TODO: Delete the note below before merging new content to the main branch.

.. note::

   **This technote is a work-in-progress.**

Abstract
========

An organising structure and improvement roadmap for notebooks driven by mobu (and/or available for ad-hoc use) for the purposes of conducting system health checks.

Background
==========

SQuaRE's stable of services includes mobu, a harness that continuously uses flocks of bot users to perform realistic workloads on our systems, particularly in the Rubin Science Platform.
While mobu is agnostic about the kind of payload it executes, in practice the most widely used mobu content is notebooks that can be executed in the Rubin Science Platform.
Not only does this kind of payload provide deep integration testing of multiple services (and data holdings) under realistic usage conditions, but it also makes it easy for us to leverage content written for other purposes (eg science tutorials) for testing and monitoring.

At the time this technote was first written (July 2023) mobu runs payload from two repos:

- system-test

  This is the original testing repo.
  It contains a mix of simple tutorials, service demonstrations and ad-hoc troubleshooting notebooks for historical reasons.
  All run fast and use minimal amount of data, though they do make assumptions about what data collections are available.
  Contributions are generally from service developers and are curated by the Rubin Data Services teams.

- tutorial-notebooks

  These are notebooks containing substantive science usecases.
  This repo contains complete tutorials that need specific data collections and types and can be sensitive to specific versions of the Rubin Science Pipelines.
  Some of these notebooks take non-trivial amounts of time to run to completion, and could involve a non-transient load on underlying services.
  Contributions are primarily from scientists and are curated by the Rubin Community Science Team.

For reasons that relate to the capabilities of mobu's antecedent prototype, both repos reflect certain operational decisions such as:

- They have a "prod" branch as the production branch with "main" as the development branch.
- A checkout of the prod branch is automatically included (and updated) in each user's home space on data.lsst.cloud
- These checkouts are write-protected to avoid users ending with a dirty copy.

Goal
----

The goal of this technote is to outline a plan to address operational frictions involving primarily the system-test repo and identify improvements that can be made to support the growing number and purposes of the notebooks run by mobu in the various Rubin Science Platform deployments.

Issues and solutions
====================

Notebook payload repos
----------------------

**Before:** Notebooks are in system-test and tutorial-notebooks as described above.

**After:** Notebooks to be in three repos:

   - tutorial-notebooks (no change)
   - system-test
   - curation-check

**Discussion:**

With the advent of the tutorial-notebooks, we have a large suite of well-curated full-workflow notebooks to emulate realistic user workflows, and therefore we no longer need to use system-test notebooks for this purpose.
At the same time, we have a need for service checkout and monitoring in a large number of deployments, many of which do not have access to large facility datasets (such as the hardware teststands). So system-test should focus on

- single-service (to whatever degree practical) checkout and monitoring notebooks
- ad-hoc troubleshooting notebooks (not run under mobu)

In addition, we have identified a need for mobu-run notebooks that perform specific data checks, in some cases across deployments (for example: does the summit EFD and the USDF EFD agree on the number of measurements for a given topic during a particular time period).
Given the strong semantic awareness of these checks and the need to compare data holdings across cluster, these will live in a separate repo (curation-check) and will contain payloads in separate directories that either (a) run on a specific deployment or (b) compare data in different data facilities.
For the latter, given the fact that some of these comparisons will involve data holdings behind a VPN (such as the summit EFD, summit Butler etc), these should run from within a VPN deployment. Pressure of resources at the summit would suggest the best home for them would be to run under the base cluster's mobu.

Service-specific checks
-----------------------

**Before:** Mobu can be configured to run all notebooks in a given directory and branch, but beyond that all notebook payloads in that directory/branch must work.

**After:** By leveraging a naming convention that matches the application name in phalanx, mobu uses the ArgoCD UI or the environment manifest to only run notebooks corresponding to deployed services.

**Discussion:** We want a more fine-grained way of running only notebooks that are expected to complete successfully.
However an opt-in approach of creating manifests of specific notebooks in a repo (or a proxy of the same result, such as having mobu run off different branches in different environments) is a maintenance headache.
If instead we adopt a naming convention for notebooks that match the ArgoCD application name, mobu can be modified to run all notebooks from participating services.
For example, if at the La Serena base deployment nublado is set to true in the environment yaml and tap is set to false and there were notebooks in system-test named

.. code-block:: bash

      nublado_login.ipynb
      nublado_dask_cluster.ipynb
      tap.ipynb

mobu would run the nublado notebooks but not attempt the tap one.

The advantage of this approach is that developers can check in new notebooks for services without necessitating mobu changes.
Note that currently mobu caches notebooks on start-up and needs to be restarted to pick up new notebooks; an API call or other method to induce a running mobu to re-poll its notebook repos would be useful (but not critical).

Reliance on specific data holdings
----------------------------------

**Before:** System-test notebooks address specific data holdings

**After:** Notebooks perform a data discovery step and run on arbitrary holdings and/or opt out of data-holding specific checks.

**Discussion:**

From the beginning we have identified the need to have a small data-set that is available on all deployments to allow system-test notebooks to run everywhere.
While there is merit to this idea, in practice finding the effort to curate such a careful minimal in size but maximal in utility dataset has been hard to find.
With the advent of the tutorial-notebooks repo, the requirement for performing substantive computations and/or service load has been eliminated from system-test.
With the proposed data curation notebooks, that require specific data holdings can live elsewhere.
To the extent that this is practical, system-test service notebooks should be written with a data discovery or data check step to see what data is available (eg. in terms of available catalogs, tap_schema could be queried first to make sure unavailable catalogs are not being requested).
In practice this is may be an aspirational requirement.

Branches
--------

**Before:** Mobu typically runs the prod branch of the notebooks (though this is configurable) with notebooks having to be cherry-picked from main to prod.

**After:** End this madness.

**Discussion:**

The need to maintain two different branches has been eliminated with mobu's ability to easily be configured to run off different branches for cases where it is useful to have an "in-development" version deployed.
Hence cherry-picking is just annoying with no particular benefit.

Outputs
-------

**Before:** Notebook contributors need to remember to clear outputs before checking in new versions of the notebooks.

**After:** Have this happen automatically (via pre-commit hook or similar), or at least raise a CI warning if there is output checked in.

**Discussion:**

Having the human remember to clear outputs before saving and checking in is error prone. Even if the notebook ends in clear_outputs(), it still implies it was run to the end before commit.
Ideally something like https://github.com/srstevenson/nb-clean would be integrated in the development workflow.

This may also be of use to other notebook repo maintainers.


Write-Only
----------

**Before:** Notebook

**After:**

Directories
-----------

**Before:** Mobu runs notebooks at the root level of repos [?]

**After:** Indicate to mobu whether to run a subdirectory (through eg. a .run file)

**Discussion:** It would be nice to have a top-level directory structure for multiple runable repos, particularly for the data curation notebooks.
This is not terribly important.






.. Make in-text citations with: :cite:`bibkey`.
.. Uncomment to use citations
.. .. rubric:: References
..
.. .. bibliography:: local.bib lsstbib/books.bib lsstbib/lsst.bib lsstbib/lsst-dm.bib lsstbib/refs.bib lsstbib/refs_ads.bib
..    :style: lsst_aa
