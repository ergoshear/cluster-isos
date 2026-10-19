# cluster-isos

## Installer package updates

The Fedora updates repository supplies current packages during target
installation and requires network access. Do not run `dnf update` in
`%pre`: that runs inside Anaconda's installer environment, not the target
system, and is not a supported way to update installation packages.
`%post` enables `fastestmirror=1` in the installed system's
`/etc/dnf/dnf.conf`, then a final `%post` runs `dnf update -y` inside the
installed system. Update failures stop installation and are logged to
`/root/ks-post-update.log` on the target system.

## Cluster node identity

The [kickstart configuration](kickstart/ks.cfg) leaves networking on DHCP.
Reserve `192.168.202.10` through `192.168.202.19` on the router for
`mlops-node-0` through `mlops-node-9`, respectively.

Before each Salt minion start, a systemd pre-start script waits for
NetworkManager's online check, sets the machine's persistent hostname from
its assigned cluster IP, and writes `/etc/salt/minion.d/minion.conf`:

```yaml
id: mlops-node-0
master: salt.ergoshear.dev
```

The ID varies with the node's reserved address. No static IP configuration
is written. If no matching address is present, or addresses for multiple
nodes are present, the script logs an error to the service journal and
prevents Salt from starting with the wrong identity. After correcting the
network, retry with `sudo systemctl restart salt-minion`.

Rebuild the installation ISO with `just iso` to include kickstart changes.