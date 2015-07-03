# HAProxy SNI Demo

Illustrates how to set up HAProxy as an TLS terminator for several virtual hosts, served by one or more Apache webservers.
The point is to show that there is no need to use ACLs and/or to explicitly list any domain names in `haproxy.cfg`.

# Get started
* Install [Vagrant](https://www.vagrantup.com)
* Clone the repository, e. g. into `~/vagrant/haproxy-sni`
* In the checked out directory execute `vagrant up`

This will start 2 virtual machines:

1. _web_ -- Running Apache with several virtual hosts, all running on the same IP and port.
1. _haproxy-1_ -- Running haproxy 1.5, configured to terminate TLS and direct traffic to _web_

Make sure your _host's_ `/etc/hosts` file has the following entries. Otherwise the demo will not work:

127.0.0.1         localhost _potentially other names_ www.example.com be01.example.com be01 be02.example.com be02 be03.example.com be03

Once Vagrant has started the VMs you can access these URLs from a browser of your choice:

  * [haproxy-1 status page](http://localhost:8404/monitor)
  * [virtualhost be01](https://be01.example.com:8443)
  * [virtualhost be02](https://be02.example.com:8443)
  * [virtualhost be03](https://be03.example.com:8443)

You will get a certificate warning, because the certificate authority that issued the individual certificates for each virtual hosts is of course not deemed trustworthy by your browser.
This is normal and will not happen if you use certificates issued by any CA your system trusts. Of course, you could also add the CA certificate `ca.crt` from the `ssl` subfolder in this repo.

## Creating the certificates

The following commands were used to create the files in the `ssl` subdirectory.
Obviously, `vim` is only a placeholder; you can check the contents of the files for yourself.
The relevant files are then copied into the haproxy VM via `Vagrantfile`.
**Warning**: The OpenSSL config file is by no means a best practice example. For the purpose of the example it is very much irrelevant which exact hash algorithms, key lengths or other settings are included.

```
$ openssl genrsa -out ca.key 2048
$ # vim ca.cnf
$ mkdir newcerts
$ touch index.txt
$ echo '01' > serial
$ # vim be01-san.cnf be02-san.cnf be03-san.cnf
$ openssl genrsa -out bexx.key.pem 2048
$ openssl req  -new -key bexx.key.pem -out be01.csr
$ openssl req  -new -key bexx.key.pem -out be02.csr
$ openssl req  -new -key bexx.key.pem -out be03.csr
$ openssl ca -extensions server -extfile be01-san.cnf -config ca.cnf -in be01.csr -out be01.crt -notext
$ openssl ca -extensions server -extfile be02-san.cnf -config ca.cnf -in be02.csr -out be02.crt -notext
$ openssl ca -extensions server -extfile be03-san.cnf -config ca.cnf -in be03.csr -out be03.crt -notext
$ cat be01.crt bexx.key.pem > be01_combined.crt
$ cat be02.crt bexx.key.pem > be02_combined.crt
$ cat be03.crt bexx.key.pem > be03_combined.crt
```

