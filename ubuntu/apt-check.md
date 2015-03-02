# apt-check

_apt\_check_ is a small utility used by some ubuntu scripts to get information about packages that can be updated. It's used to update the MOTD on Ubuntu to check how many packages can be updated and which of these are security.

Example:

    /usr/lib/update-notifier/apt-check --human-readable

    31 packages can be updated.
    0 updates are security updates.


Very useful to find when you have security updates pending.
