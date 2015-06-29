# Ganeti Config Archive Vulnerability #

Published 2014-08-12

Ganeti, an open source virtualisation manager, suffered from an insecure file permission vulnerability that leads to sensitive information disclosure. This issue was fixed with versions 2.10.7 and 2.11.5.

The Ganeti upgrade command ‘gnt-cluster upgrade’ creates an archive of the current configuration of the cluster (e.g. the contents of `/var/lib/ganeti`). The archive is named following the pattern `ganeti*.tar` and is written to `/var/lib/`. Such archives were written with too lax permissions that made it possible to read them as unprivileged user, on the master node.

The configuration archive contains sensitive information, including SSL keys for the inter-node communication via RPC as well as the credentials for the remote API (RAPI). Such information can be used to control various operations of the cluster, including shutting down and removing instances and nodes from the cluster, or assuming the identity of the cluster in a MITM attack.

This vulnerability only affects Ganeti clusters meeting the following criteria:
  * The cluster is running Ganeti version 2.10.0 or higher.
  * The upgrade command was run, for example when upgrading from 2.10 to 2.11.
  * Unprivileged users have access to the host machines and in particular to the cluster master node.

With the fixed release, the upgrade command will set the permissions of the archives properly. However, in case previous versions have created an unsafe archive already, the following mitigations are advised:
  * Remove the access to the archive for unprivileged users (for example by running `chmod 400 /var/lib/ganeti*.tar`).
  * Renew the SSL keys by running “gnt-cluster renew-crypto”. You may need to pass the --new-cluster-certificate, --new-confd-hmac-key, --new-rapi-certificate, --new-spice-certificate, --new-cluster-domain-secret flags, and (for version 2.11 only) the --new-node-certificates flag.
  * Renew the RAPI credentials by editing the /var/lib/ganeti/rapi\_users file.
  * Update RAPI, confd and other clients with the new secrets and certificates, if applicable.
  * Look for any other information regarded as secret in /var/lib/ganeti and change it. For example VNC and SPICE passwords are not by default kept there, but could, if Ganeti is so configured.

This vulnerability will be published as oCert-2014-006 on ocert.org; CVE ID is pending. Thanks to Apollon Oikonomopoulos for reporting and fixing this issue.

Affected versions: 2.10.0 - 2.10.6, 2.11.0 - 2.11.4

Fixed versions: 2.10.7, 2.11.5

Filed under: CVE-2014-5247 and http://www.ocert.org/advisories/ocert-2014-006.html