# \*.test Host Resolution

When using Debian, foo.localhost resolves to localhost. There are several options for adding other hosts to do the same, and one is using NetworkManager.

First, add a /etc/NetworkManager/dnsmasq.d/00-test.conf file with the following content:

```
address=/test/127.0.0.1
```

Then, add a `dns=dnsmasq` line to the `[main]` block in /etc/NetworkManager/NetworkManager.conf:

```
# Before
[main]
plugins=ifupdown,keyfile

# After
[main]
plugins=ifupdown,keyfile
dns=dnsmasq
```

After that, reload the NetworkManager using `sudo systemctl reload NetworkManager` and foo.test will resolve to localhost.
