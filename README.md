# imap-server

IMAP Server module for Nodejs; utilizes MongoDB for message-store as part of the
Mismo Messaging Systems (https://mismo.email/).

All features of an IMAP server are implemented as plugins; currently, only the
'announce' plugin is fully implemented.  It appears that the original author
started a 'STARTTLS' module, also.


## Installation

```sh
npm install mismo-imapd
```

## Usage

```javascript
var ImapServer = require('mismo-imapd');
var server = ImapServer();

const tlsOpts = {
    const tlsOptions = {
    key: fs.readFileSync(process.env.IMAP_TLS_KEY || '/opt/Mismo/ssl/imapd-key.pem'),
    cert: fs.readFileSync(process.env.IMAP_TLS_CERT || '/opt/Mismo/ssl/imapd-cert.pem'),
    ca: fs.readFileSync('/etc/ssl/certs/ca-certificates.crt'),
    rejectUnauthorized: false,      // Reject invalid/expired certs
    minVersion: 'TLSv1.3',          // Only the most recent version of TLS
    handshakeTimeout: 15000,        // 15 seconds for TLS handshake to complete; else, error
    keepAlive: true
};

// use plugin
var plugins = require('mismo-imapd/plugins');
server.use(plugins.announce);
/* use more builtin or custom plugins... */

const tls = require('tls');
const tlsPort = process.env.IMAP_TLS_PORT || 993;
const tlsHost = '127.0.0.1';
tls.createServer(tlsOpts, server).listen(tlsPort, tlsHost, () => {
    console.log('Initialized IMAP+TLS Service on ' + tlsHost + ':' + tlsPort);
});
```

## Plugins

### Built-in plugins

#### announce

Required by [IMAP4rev1][imap]. This plugin also send the optional capability list.

#### starttls

Provide encrypted communication for IMAP via the [STARTTLS][starttls] command.

```javascript
server.use(plugins.starttls, {
    /* mandatory hash of options for crypto.createCredentials
     * http://nodejs.org/api/crypto.html#crypto_crypto_createcredentials_details
     * with at least key & cert
     */
    key: Buffer,
    cert: Buffer
});
```

#### debug

This plugin log various information.

### authentification helper

Here's how to implement auth plain without worrying about the underlying protocol:
```javascript
var WrapAuthPlain = require('imap-server/util/auth_plain_wrapper');

exports.auth_plain = WrapAuthPlain(function(connection, username, password, next) {
    if(username == "john.doe@example.com" && password == "foobar") {
        next(null, 'OK');
    }
    else {
        next(null, 'NO');
    }
});
```

## Notes

* Default port : 143
* SSL port : 993
* rfc3501 (IMAP4rev1) : http://tools.ietf.org/html/rfc3501
* return flags : OK, NO, BAD

* getCapabilities ( connection ) Sync, return [cap, ...]
* register
* connection ( connection, next )
* starttls ( next )
* auth_* ( next )
* unknown_command ( connection, line, next )


[imap]: http://tools.ietf.org/html/rfc3501 "RFC 3501"
[starttls]: http://tools.ietf.org/html/rfc2595 "RFC 2595"
[sasl-ir]: http://tools.ietf.org/html/rfc4959 "RFC 4959"
