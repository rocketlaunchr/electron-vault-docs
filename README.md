# Electron Vault

**Introduction**

Electron.js is an amazing open-source project. It allows a single team of javascript/Node.js developers to create rich cross-platform Desktop applications.

Unfortunately, it's [not possible to secure your source code](https://github.com/electron/electron/issues/3041). This also means that sensitive credentials are fully exposed. These could be:

- REST API credentials
- OAuth credentials
- Passwords
- URLs not intended to be known to the public (i.e. private backend API endpoints)
- Any Keys and Secrets

For applications that heavily rely on communicating with a backend service, electron apps pose another layer of security concerns. It's very challenging to prove to your server that the client is your electron application with no unauthorized alterations. Furthermore, since network communication is easily observable, any communication your application does can be replicated with a simple REST client.

Electron Vault can be used to solve this issue.

**Further Reading**

- [Blog Post](https://medium.com/@rocketlaunchr.cloud/introducing-electron-vault-d09ade2c2020)
- [Documentation](https://rocketlaunchr.github.io/electron-vault-docs/)
- [rocketlaunchr.cloud](https://rocketlaunchr.cloud) - Obtain an API Token for free
