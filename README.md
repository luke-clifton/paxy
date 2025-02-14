# paxy

`paxy` is an HTTP proxy with support for PAC files.

The main use case is to expose a single HTTP proxy which follows a set of rules
to select different proxies depending on the hostname of the connection.

## Installation

```bash
go get -u -v github.com/robbiemcmichael/paxy
```

## Usage

```bash
paxy path/to/proxy.pac
```

Then configure your software to use the proxy. For example:

```bash
export http_proxy=http://127.0.0.1:8228
export https_proxy=http://127.0.0.1:8228
curl https://github.com
```

### Options

```
Usage: paxy [options] pac_file
  -p int
    	The port on which the server listens (default 8228)
```

### Examples

#### Pairing `paxy` with `cntlm`

Some corporate networks require users to access the internet through an HTTP
proxy that uses the NTLM protocol for authentication. `cntlm` handles NTLM
authentication and allows an unauthenticated HTTP proxy to be exposed on
localhost. However, if your network has complicated rules then they will be
difficult to express in the configuration file for `cntlm`.

Instead you can express your proxy rules in a PAC file and forward HTTP
connections that need to go through the corporate proxy to `cntlm` instead.

```js
function FindProxyForURL(url, host) {
  if (shExpMatch(host, "*.internal.example.com")) {
    return "DIRECT";
  }

  return "HTTP 127.0.0.1:3128";
}
```

#### SSH dynamic port forwarding

SSH dynamic port forwarding exposes a SOCKS proxy that allows you to tunnel
traffic through another host via SSH.

For example, if a section of your network has the domain name suffix
`.private.example.com` which is only accessible via the host `remote-host`, you
can create the dynamic port forward with:

```bash
ssh -D 8229 remote-host
```

The use the following PAC file:

```js
function FindProxyForURL(url, host) {
  if (shExpMatch(host, "*.private.example.com")) {
    return "SOCKS5 127.0.0.1:8229";
  }

  return "DIRECT";
}
```

This will route all HTTP connections to `*.private.example.com` via
`remote-host` while all other HTTP connections are made from your host
directly.
