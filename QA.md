<h1> Introduction </h1>

Testing Ganeti is done by unit tests and by QA. On this wikipage, we document our QA design and setup.

You can run QA on our public [BuildBot](BuildBot.md) setup or on your own test cluster. (TODO: document that).

<h2>Table of contents</h2>


# Adding Code to QA #

The QA scripts can and should be extend when functionality of ganeti grows. The QA scripts reside in the `qa` directory in the repository. Consider the following steps:

  * Add a new QA script or edit an existing one. Make sure it is called in `ganeti-qa.py`.

  * If necessary, add new configuration data to the qa-sample.json file. Be aware that this is not the configuration file that is used to actually run the QA on the QA machines, for that you need to change the machine's configuration.

  * If you add a new qa file `qa_something.py`, don't forget to add it to the `Makefile.in` and run `make pylint-qa` to lint your changes.

  * If you don't intend to change the QA machines' configuration, but you want to test your code on the QA machines, temporarily enable your tests unconditionally (thus invoking them with `RunTest(...)` instead of `RunTestIf(...)`. Of course, in this case your scripts must use some reasonable defaults, since no (new) configuration is available.

  * If you added some new QA think about the cleanliness assumptions the test makes, and in which way it might leave traces in case of failure.

# Design ideas #

These are a couple of ideas that stand behind QA. Try to comply to them when you extend the QA scripts. If you think, there should be more here, feel free to extend the list.

## General ideas ##

  * The main idea of QA is to run commands on an actual cluster.
  * The output of the `gnt-xxx` info commands is in YAML, which makes it easy to parse and check the outcome of the tests.

## Files and their purposes ##

The QA files and their purposes:
  * `ganeti-qa`, main script to run the QA
  * `qa_entity` for the various entities of a cluster, like `qa_node`, `qa_instance`
  * `qa_config` contains utility functions to deal with the QA config
  * `qa_error` contains definitions of exceptions
  * `qa_util` contains various helper functions

The qa scripts should depend as little as possible on the ganeti code. This is not strictly the case right now, since we are dependent on `constants.py` for example, but generally we should try to not add more dependencies.

The different `qa_entity` scripts should not depend on each other. If you have a test that covers more than one entity, you should add this test to the main `ganeti-qa` script.

## Ramp-up and clean-up ##

  * Right now, a cluster is only created once per QA run and then all tests are run on it. That is no reason to not add more tests for `gnt-cluster init`.
  * Each test should leave the cluster in the state that it was before the test. Make sure you clean up after yourself. (Currently, there is no function to test this, see http://code.google.com/p/ganeti/issues/detail?id=453 for that.)
  * Consider running and checking `cluster verify` at the end of each test.

## Test selection ##

  * Keep in mind that every additional test prolongs the total time QA takes to run.
  * Make it possible to dis/enable your tests via the config file.
  * When adding a new parameter to the config file, make up your mind whether or not it should run on which QA setup. Don't forget to update our QA machine's config files if necessary.
  * It is the test's responsibility to check if it actually makes sense to run. For example if a test makes only sense on drbd instances, it should not run for any other disk template.

## Node and Instance allocation ##

`qa_config` contains various utility functions to deal with the configuration of the QA.

In particular, it provides functions to acquire nodes and instances.
  * Note that acquired nodes are not marked as busy, thus it is possible to acquire the same node more than once. The idea is just to have some kind of load balancing on the QA nodes to not run all tests on one node and let the others idle.
  * Note that acquired instances are busy and cannot be acquired by other tests.

## Sets of tests ##

Most of the tests are single steps in the QA run, except for the instance loop, which loops over a set of disk templates and tests various instance tests on each of those instances.

All the tests enabled in the configuration file are run. If some test is not supported (e.g.g, it doesn't make sense or it's broken), it's the responsibility of the single test to check if the cluster/instance state makes sense for the test or return early. `qa_instance.py` and `qa_config.py` contain helper functions that simplify checking for such conditions, e.g., `IsFailoverSupported`, `IsMigrationSupported` in the former, and `IsTemplateSupported` in the latter.