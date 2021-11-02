## sdig
A little script does DNS lookup to get Akamai staging network IP address for a domain

## Usage
```
$ sdig www.foo.com [options]
```

## Installation
```
$ brew tap ricky840/sdig
$ brew install sdig
```

## Options
```
Usage: sdig www.foo.com [options]
    -v, --verbose                    Verbose output. Show whole resolution chain.
    -n, --nameserver NAMESERVER      Use this nameserver to resolve.
    -a, --add                        Add staging IP spoofing to the hosts file.
    -r, --remove                     Delete all spoofing entries for the domain from the hosts file.
    -e, --etn NUMBER(1~11)           Add ETN server spoofing to hosts file.
    -h, --help                       Display help message.
```

## Examples
To get Akamai staging network IP address for www.akamai.com
```
$ sdig www.akamai.com
e1699.dscc.akamaiedge-staging.net.
23.62.71.31
```
Use a specific nameserver to resolve the domain
```
[ricky@workbox ~ ]$ sdig www.akamai.com -n 1.1.1.1
e1699.dscx.akamaiedge-staging.net.
104.81.198.238

[ricky@workbox ~ ]$ sdig www.akamai.com -n 8.8.8.8
e1699.dscx.akamaiedge-staging.net.
23.51.1.37
```

Add Spoofing staging IP address for www.akamai.com
```
$ sdig www.akamai.com -a
e1699.dscx.akamaiedge-staging.net.
23.51.1.37
sdig: 23.51.1.37 was added to /etc/hosts.
```

Remove www.akamai.com entry from /etc/hosts file.
```
$ sdig www.akamai.com -r
sdig: removed www.akamai.com from /etc/hosts.
```

Add Spoofing ETN test server IP address.
```
$ sdig www.akamai.com -e 7
sdig: etn7.akamai.com(205.185.220.230) was added to /etc/hosts.
```
