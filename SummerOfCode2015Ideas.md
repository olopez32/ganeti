# Google Summer of Code 2015 #

Here are our project ideas for Google Summer of Code 2015

It might be interesting to have a look at ideas in [2013](SummerOfCode2013Ideas.md) and [2014](SummerOfCode2014Ideas.md).

# Project proposals #

## Improvements to the instance bin-packing tool (hsqueeze) ##
(language: haskell) [design](http://docs.ganeti.org/ganeti/master/html/design-hsqueeze.html)

The hsqueeze tool is used to move virtual machines of a cluster to a subset of its physical machines, allowing to powering down the rest of physical machines to conserve energy.

**Task:** Make hsqueeze use the dynamic CPU data that is now collected thanks to previous summer of code projects.

Note: This project is well suited for someone who is already using hsqueeze or who cooperates with an organization that uses it.

## Location awareness ##
(language: haskell) [design](http://docs.ganeti.org/ganeti/2.14/html/design-location.html)

Location awareness allows Ganeti to avoid common physical causes of failures when placing virtual machines on physical nodes.

**Tasks:**

  * Improve the current implementation (i.e., finish implementing the design).
  * Extends to more aspects of location.

## Monitoring daemon ##
(haskell)

The monitoring daemon collects information from physical machines of a Ganeti cluster.

**Tasks:**

  * Implement data collectors for CPU (and maybe memory) usage for Xen-based clusters;
  * possibly use this information for load-based cluster balancing.
  * Add an interface for generic plugins that collect arbitrary user-defined data.