## Introduction ##

You are here. You do not want to be here. Ganeti is missing that one feature you need and someone recommended that you edit the configuration. This page will tell you how, and try to dissuade you from doing so unless really necessary.

## Why you should (not) edit the configuration ##

We generally do not encourage people to edit the configuration, but there are certain snafus that are easiest to get out of by making a small change to the configuration. We try to fix these, so please make an issue, comment on an existing one, or ask around for alternative solutions in IRC or on the mailing list.

For resiliency purposes, the configuration is distributed and checked for unauthorized modifications, and changing it is intentionally not easy. You will have to go through a precise series of steps, and should you stray too far, the situation may go beyond repair. This is why you should not make a habit out of it.

## How to edit the configuration ##

  1. Pause the watcher with `gnt-cluster watcher pause 6h`. Give yourself more time if needed.
  1. Drain the job queue with `gnt-cluster queue drain`
  1. Wait until all jobs have been completed, as seen with `gnt-job list`
  1. Stop Ganeti on the master node.
  1. Backup your configuration. This is usually `/var/lib/ganeti/config.data`
  1. Seriously, create a backup.
  1. Make the current configuration readable by prettifying the JSON file  , e.g. `/usr/lib/ganeti/tools/fmtjson <config.data >config.readable`
  1. Make the edits that you want to make to the readable configuration.
  1. Replace the current configuration with the modified readable one.
  1. Start Ganeti on the master node.
  1. Undrain the queue with `gnt-cluster queue undrain`
  1. Run `gnt-cluster redist-conf`
  1. Make sure all is well with `gnt-cluster verify`, and whatever specific checks you need
  1. Unpause the watcher with `gnt-cluster watcher continue`

If anything goes wrong, you can restore the old configuration by using the same procedure. If you changed the state of the cluster in the meantime, Ganeti might complain upon seeing the state of the configuration not match reality.