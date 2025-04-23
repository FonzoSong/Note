### tls: failed to verify certificate: x509: cannot validate certificate for IP because it doesn't contain any IP SANs

**两种解决方式：**

1. Connect by name (either implement DNS, or if your environment is very static, add the hostnames to /etc/hosts on the prometheus server)
3. Reissue the certs to include an IP SAN for the host's IP
