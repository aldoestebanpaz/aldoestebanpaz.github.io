# Curl usage notes

## Test connection without a DNS entry in the hosts file:

This is helpful when you don't want to allow regular DNS lookups for a request.

Usually, the solution would be to update /etc/hosts, and hardcode the IP address. But that requires root privileges, and there's always the risk that you forget you made the change, resulting in garbage in your system.

Since curl v7.21.3 you can use the --resolve flag, which allows forcing curl to not perform lookups, and instead use the IP address and port number provided.

Example:

```sh
# check insecure connection with IP address
curl -v 192.168.33.11

# check insecure connection with domain and without www
curl -v http://myapp.dev.test \
  --resolve myapp.stg.test:80:192.168.33.11
# check insecure connection with domain and with www
curl -v http://www.myapp.dev.test \
  --resolve www.myapp.stg.test:80:192.168.33.11

# check HTTPS redirection with domain and without www
curl -v -k -L http://myapp.dev.test \
  --resolve myapp.dev.test:80:192.168.33.11 \
  --resolve myapp.dev.test:443:192.168.33.11
# check HTTPS redirection with domain and with www
curl -v -k -L http://www.myapp.dev.test \
  --resolve www.myapp.dev.test:80:192.168.33.11 \
  --resolve www.myapp.dev.test:443:192.168.33.11
```

Where:
- `-v, --verbose` - give me details
- `-k, --insecure` -  skip the secure connection verification step (varification of TLS certificate)
- `-L, --location` - Follow redirects
- `--resolve ` - Provide a custom address for a specific host and port pair. Using this, you can make the curl requests(s) use a specified address and prevent the otherwise normally resolved address to be used. Consider it a sort of /etc/hosts alternative provided on the command line.

