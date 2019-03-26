# mtls-ssh-svc
Mutual TLS wrapped SSH, ssh configuration and a systemd unit file

## Summary

This is an example of a service that exists solely to ensure only users with a signed certificate and pinned CN can reach your SSH server. It is a bit ridiculous, but it is a good example of how flexible `socat` and OpenSSH are

## How-To

I'm not going to cover how to generate the certificates in this documnentation. If you want, you can use [quick-dirty-CA](https://github.com/mzpqnxow/quick-dirty-CA) to simplify this, or if you actually know your way around managing PKI on a small scale, do it your own way. You need the following

* A certificate authority (your own)
* A signed server certificate with a CN like "MTLS-SSH Server01"
* A signed client certificate with a CN like "MTLS-SSH Client01"
* OpenSSH
* `socat`
* (optional) `systemd`

### Generate the CA, issue the server and client certificates / keypairs

Not covered here as mentioned above

### Make sure `socat` is installed on both server and client

If you do not have `socat`, use `apt-get install socat` or `yum install socat`, or whatever. Or you can build it from [source](http://www.dest-unreach.org/socat/)

### Place your certificates / keys on each system

You need to make sure the certificates and keys are where they need to be

#### Server

The server needs the CA certificate to authenticate the client as well as the signed "MTLS-SSH Server01" certificate and key to allow the client to authenticate the server. You can concatenate the certificate and key into one PEM file if they are in PEM format

#### Client

The client needs the CA certificate to authenticate the server as well as the signed "MTLS-SSH Client01" certificate and key to allow the server to authenticate the client. You can concatenate the certificate and key into one PEM file if they are in PEM format

### Don't be stupid

Note the CA key is not involved anywhere here, hopefully you know that is only required when issuing new certificates :>

### Set up the systemd service on the server

In the end state, you want OpenSSH to listen only on loopback on the server. First you will need to get `socat` up on the server. You can do it as a one-line command, preferably in a GNU `screen` session, or you can do it properly via systemd. A unit file is provided, you may need to adjust it slightly. Once it is in place, you will need to use `systemctl daemon-reload` and `systemctl enable mtls-ssh` as well as `systemctl start mtls-ssh`

At this point, your TLS wrapper is up, you can confirm this by connecting to the port with `telnet`, `nc`, or `openssl s_client -connect <server>:<port>`. Remember that your `openssl` connection will be terminated unless you supply the required client certificate and keyfile.

### Set up ~/.ssh/config on the client

You can see the example in `ssh_config` in this repository. This makes it a seamless user experience

### Test

Use ssh to make sure things work! Or, use `socat` / `openssl s_client -connect ...` in baby steps to test each side

### Set OpenSSH to only listen on loopback

Edit `/etc/ssh/sshd_config` so there is only one line containing `ListenAddress` and it looks like this

`ListenAddress 127.0.0.1`

## EOF

Sorry for no hand-holding, this is just a reference. Just remember `openssl s_client -help` is your friend when trying to troubleshoot issues. The `s_client` subcommand allows you to specify your CA, client certificate and key and it also allows you to specify debug mode, etc ..