# tls

This plugin enables the use of TLS (via `STARTTLS`) in Haraka.

For this plugin to work you must have SSL certificates installed correctly.

## Install Location

Key and certificate chain default locations are as follows. The paths
can be overridden in the `config/tls.ini` file using `key` and `cert` options.

    key=tls_key.pem
    cert=tls_cert.pem

## Purchased Certificate

If you have a purchased certificate, append any intermediate/chained/ca-cert
files to the certificate in this order:

1. The CA signed SSL cert
2. Any intermediate certificates
3. The CA root certificate

Example:

    cat mail.example.com.crt intermediary_cert.crt ca-cert.crt > config/tls_cert.pem

See also [Setting Up TLS](https://github.com/haraka/Haraka/wiki/Setting-up-TLS-with-CA-certificates)

## Self Issued (unsigned) Certificate

Create a certificate and key file in the config directory with the following
command:

    openssl req -x509 -nodes -days 2190 -newkey rsa:2048 \
            -keyout config/tls_key.pem -out config/tls_cert.pem

You will be prompted to provide details of your organization. Make sure the
Common Name is set to your servers Fully Qualified Domain Name, which should
be the same as the contents of your `config/me` file.

## Configuration

The following settings can be specified in `config/tls.ini`.

### key

Specifies an alternative location for the key file. If multiple keys are to be
specified, use `key[]=` assignment for each of them. Non-absolute paths are relative
to the `config/` directory.

For example, to configure single key and cert chain files, located in the `config/`
directory, use the following in `tls.ini`:

    key=example.com.key.pem
    cert=example.com.crt-chain.pem

If multiple pairs of key and cert chain files should be used, outside of the haraka
`config/` directory, configure instead:

    key[]=/etc/ssl/private/example.com.rsa.key.pem
    cert[]=/etc/ssl/private/example.com.rsa.crt-chain.pem
    key[]=/etc/ssl/private/example.com.ecdsa.key.pem
    cert[]=/etc/ssl/private/example.com.ecdsa.crt-chain.pem

### cert

Specifies an alternative location for the certificate chain file. If multiple
certificate chains are to be used, use `cert[]=` assignment for each of them.
Non-absolute paths are relative to the `config/` directory. See the description of
the `key` parameter for specific use.

### no_tls_hosts

If needed, add this section to the `config/tls.ini` file and list any IP ranges that have
broken TLS. Ex:

    [no_tls_hosts]
    192.168.1.3
    172.16.0.0/16


The [Node.js TLS](http://nodejs.org/api/tls.html) page has additional information
about the following options.

### ciphers

A list of allowable ciphers to use. Example:

    ciphers=EECDH+AESGCM:EDH+aRSA+AESGCM:EECDH+AES256:EDH+aRSA+AES256:EECDH+AES128:EDH+aRSA+AES128:RSA+AES:RSA+3DES

See also: [Strong SSL Ciphers](http://cipherli.st) and the [SSLlabs Test Page](https://www.ssllabs.com/ssltest/index.html)

### honorCipherOrder

If specified, the list of configured ciphers is treated as the cipher priority from
highest to lowest. The first matching cipher will be used, instead of letting the
client choose the cipher. The default is `false`.

### ecdhCurve

Specifies the elliptic curve used for ECDH or ECDHE ciphers.
Only one curve can be specified. The default is `prime256v1` (NIST P-256).

### dhparam

Specifies the file containing the diffie-hellman parameters to
use for DH or DHE key exchange. Create such a file using `openssl dhparam`.
No DH ciphers can be used without this parameter given.

    openssl dhparam -out config/dhparams.pem 2048


### requestCert

Whether Haraka should request a certificate from a connecting client.

    requestCert=[true|false]  (default: true)

### rejectUnauthorized

Reject connections from clients without a CA validated TLS certificate.

    rejectUnauthorized=[true|false]  (default: false)

### secureProtocol

Specifies the OpenSSL API function used for handling the TLS session. Choose
one of the methods described at the
[OpenSSL API page](https://www.openssl.org/docs/manmaster/ssl/ssl.html).
The default is `SSLv23_method`.

## Inbound Specific Configuration

By default the above options are shared with outbound mail (either
using `smtp_forward`, `smtp_proxy` or plain outbound mail heading to
an external destination). To make these options specific to inbound
mail, put them under an `[inbound]` parameter group. Outbound options
can go under an `[outbound]` parameter group, and plugins that use
SMTP tls for queueing such as `smtp_proxy` and `smtp_forward` can
use that plugin name for plugin specific options.
