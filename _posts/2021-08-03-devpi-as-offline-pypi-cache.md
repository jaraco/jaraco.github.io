---
layout: post
title: Using devpi as an offline PyPI cache
---

Want to develop Python applications, but realize the dependence on the network and PyPI is a hindrance? [devpi](https://devpi.net) provides a robust way to keep a cache of all the packages you need to develop. And it's super-easy.

# Install devpi

First, install devpi as an application somewhere on your system. [pipx](https://pypa.github.io/pipx/) provides a great way to do that:

```

~ $ pipx install devpi-server
  installed package devpi-server 6.1.0, Python 3.9.6
  These apps are now globally available
    - devpi-export
    - devpi-fsck
    - devpi-gen-config
    - devpi-gen-secret
    - devpi-import
    - devpi-init
    - devpi-passwd
    - devpi-server
done! ✨ 🌟 ✨

~ $ devpi-init
INFO  NOCTX Loading node info from /Users/jaraco/.devpi/server/.nodeinfo
INFO  NOCTX generated uuid: 63c633d39fb545f08f1432172c5ee4eb
INFO  NOCTX wrote nodeinfo to: /Users/jaraco/.devpi/server/.nodeinfo
INFO  NOCTX DB: Creating schema
INFO  [Wtx-1] setting password for user 'root'
INFO  [Wtx-1] created user 'root'
INFO  [Wtx-1] created root user
INFO  [Wtx-1] created root/pypi index
INFO  [Wtx-1] fswriter0: commited at 0
~ $ devpi-server
2021-08-03 08:02:33,706 INFO  NOCTX Loading node info from /Users/jaraco/.devpi/server/.nodeinfo
2021-08-03 08:02:33,707 INFO  NOCTX wrote nodeinfo to: /Users/jaraco/.devpi/server/.nodeinfo
2021-08-03 08:02:33,787 WARNI NOCTX No secret file provided, creating a new random secret. Login tokens issued before are invalid. Use --secretfile option to provide a persistent secret. You can create a proper secret with the devpi-gen-secret command.
2021-08-03 08:02:36,913 INFO  NOCTX devpi-server version: 6.1.0
2021-08-03 08:02:36,913 INFO  NOCTX serverdir: /Users/jaraco/.devpi/server
2021-08-03 08:02:36,913 INFO  NOCTX uuid: 63c633d39fb545f08f1432172c5ee4eb
2021-08-03 08:02:36,913 INFO  NOCTX serving at url: http://localhost:3141 (might be http://[localhost]:3141 for IPv6)
2021-08-03 08:02:36,913 INFO  NOCTX using 50 threads
2021-08-03 08:02:36,913 INFO  NOCTX bug tracker: https://github.com/devpi/devpi/issues
2021-08-03 08:02:36,913 INFO  NOCTX IRC: #devpi on irc.freenode.net
2021-08-03 08:02:36,913 INFO  NOCTX Hit Ctrl-C to quit.
2021-08-03 08:02:36,921 INFO  Serving on http://127.0.0.1:3141
2021-08-03 08:02:36,921 INFO  Serving on http://[::1]:3141
```

Now a locally-running PyPI mirror is running at `localhost:3141/root/pypi/`!

Leave that server running in its own window or tab.

# Prime the cache

Although the server is running, it does not yet have a mirror of the packages needed to build and test the project. To get those, in a separate terminal, install the relevant project(s) or run the project's test suite using devpi as the package index:

```sh
~ $ $PIP_INDEX_URL = 'http://localhost:3141/root/pypi/'
~ $ # or in bash:
~ $ # export PIP_INDEX_URL=http://localhost:3141/root/pypi/
~ $ pip-run -q jaraco.mongodb pmxbot pip-run jupyter -- -c pass
```

By setting `PIP_INDEX_URL`, pip will now request packages from the local index, which will mirror the packages from PyPI and keep a local cache.

Although this example uses [pip-run](https://pypi.org/project/pip-run) to download the packages, one could just as easily use `pip` or `tox`.

If priming the cache using tox, be sure to pass `-r` to tox to rebuild the environments from scratch to get all packages.


# Testing the cache

Next, test that the cache is working by disabling the network and putting devpi in offline mode.

```
~ $ sudo ifconfig en0 down
Password:
~ $ ping pypi.org
ping: cannot resolve pypi.org: Unknown host
```

To restart devpi in offline mode, kill the existing process by hitting Ctrl+C in the devpi-server terminal and re-launch with `--offline-mode`:

```
...
2021-08-03 08:10:22,861 INFO  [req365] GET /root/pypi/+f/144/75042e2849910/idna-3.2-py3-none-any.whl
2021-08-03 08:10:22,874 INFO  [req366] GET /root/pypi/certifi/
2021-08-03 08:10:22,923 INFO  [req367] GET /root/pypi/+f/50b/1e4f8446b06f4/certifi-2021.5.30-py2.py3-none-any.whl
^C~ $ devpi-server --offline-mode
2021-08-03 08:32:12,603 INFO  NOCTX Loading node info from /Users/jaraco/.devpi/server/.nodeinfo
2021-08-03 08:32:12,606 INFO  NOCTX wrote nodeinfo to: /Users/jaraco/.devpi/server/.nodeinfo
2021-08-03 08:32:12,712 WARNI NOCTX No secret file provided, creating a new random secret. Login tokens issued before are invalid. Use --secretfile option to provide a persistent secret. You can create a proper secret with the devpi-gen-secret command.
2021-08-03 08:32:16,659 INFO  NOCTX devpi-server version: 6.1.0
2021-08-03 08:32:16,659 INFO  NOCTX serverdir: /Users/jaraco/.devpi/server
2021-08-03 08:32:16,660 INFO  NOCTX uuid: 63c633d39fb545f08f1432172c5ee4eb
2021-08-03 08:32:16,660 INFO  NOCTX serving at url: http://localhost:3141 (might be http://[localhost]:3141 for IPv6)
2021-08-03 08:32:16,660 INFO  NOCTX using 50 threads
2021-08-03 08:32:16,661 INFO  NOCTX bug tracker: https://github.com/devpi/devpi/issues
2021-08-03 08:32:16,661 INFO  NOCTX IRC: #devpi on irc.freenode.net
2021-08-03 08:32:16,661 INFO  NOCTX Hit Ctrl-C to quit.
2021-08-03 08:32:16,673 INFO  Serving on http://127.0.0.1:3141
2021-08-03 08:32:16,673 INFO  Serving on http://[::1]:3141
```

Now observe that the install commands that ran previously will succeed entirely without network access:

```
~ $ pip-run -q jaraco.mongodb pmxbot pip-run jupyter -- -c "import jupyter; print('worked!')"
worked!
```

# Beyond a mirror

Devpi can do so much more, including providing development indexes to store non-public packages or packages under development (while still mirroring upstream repos).

It also provides for repository replication, allowing your team to share a devpi instance with public and private packages and then developers can sync a replica of that instance with all the public and private packages used by the team.

Be sure to check out https://devpi.net for more details.
