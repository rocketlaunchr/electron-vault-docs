---
title: Electron Vault API

language_tabs: # must be one of https://git.io/vQNgJ
  - info

toc_footers:
  - <a href='https://rocketlaunchr.cloud'>Sign Up for an API Token</a>

search: true
---

# Introduction

Electron.js is an amazing open-source project. It allows a single team of javascript/Node.js developers to create rich cross-platform Desktop applications.

Unfortunately, it's [not possible to secure your source code](https://github.com/electron/electron/issues/3041). This also means that sensitive credentials are fully exposed. These could be:

- REST API credentials
- OAuth credentials
- Passwords
- URLs not intended to be known to the public (i.e. private backend API endpoints)
- Any Keys and Secrets

For applications that heavily rely on communicating with a backend service, electron apps pose another layer of security concerns. It's very challenging to prove to your server that the client is your electron application with no unauthorized alterations. Furthermore, since network communication is easily observable, any communication your application does can be replicated with a simple REST client.

Electron Vault can be used to solve this issue.

[Read the blog](https://medium.com/@rocketlaunchr.cloud/introducing-electron-vault-d09ade2c2020) for further details.

# How It Works

Electron Vault creates a cross-platform launcher application which sits alongside your electron application. Instead of the user directly launching your electron application, they will instead run the launcher.

The launcher will scan all the files (including your ASAR file) for tampering. You must provide a list of files and their md5 hash for this feature.

It will then launch your application and pass your secrets as environment variables. Upon loading, you can check for the values. If they don't exist, you can gracefully exit after displaying a message.

All command line parameters to the launcher will be passed as command line parameters to your electron application.

# Security Features

There are 2 security features in action.

As you probably know, all strings are hardcoded in the actual binary itself. You can easily access them using a hex editor or the `strings` program that comes with Unix based operating systems such as Linux and macOS. In fact, just open a terminal and run `strings <executable path>` on your favorite applications to view all the embedded strings. For macOS, you may have to right-click and click "Show Package Content".

What Electron Vault does is stores your secrets in an encrypted form which then gets dynamically converted back to the original at runtime. That way, the original secret is not baked into the binary.

The second (and most important) feature will be kept secret for your application's protection.


# Preparation

You will need an API key which you can obtain from creating a [rocketlaunchr.cloud](https://rocketlaunchr.cloud) account.

You will need to package your source code into an [ASAR file](https://electronjs.org/docs/tutorial/application-packaging). This step is required.

You will need to compile a list of (ideally) all files that will be static so each file will have a stable md5 hash. This must be done separately for all your target operating systems (eg Windows, macOS and Linux).

> For macOS:

```info
electron/Electron.app/Contents/Resources/
  └── app.asar
  └── electron.asar

```

> For Windows/Linux:

```info
electron/resources/
  └── app.asar
  └── electron.asar
```


At an absolute minimum, the md5 is required for the `app.asar` file and the `electron.asar` file.
It is a **de-facto requirement** that you also obtain the md5 of the actual executable file that the user would ordinarily execute. This obviously differs based on the operating system.


All files are relative to:

OS | Root Path
--------- | -----------
mac | /Contents 
win | / 
linux | / 

<aside class="notice">
For macOS, click "Show Package Content" after right-clicking the app bundle file to view the directory structure.
</aside>

## Recommendations

You will need to uglify your javascript and Node.js code. Do everything you can to obfuscate your code. 

<aside class="success">
Electron Vault is used after you have finished development and prior to the distribution stage. This is because you need to obtain the md5 hash of certain files.
</aside>

# Development Advice

Since the launcher passes your secrets as environment variables, you should load them into a separate variable and then immediately clear them.

During local development, when you don't have the launcher present, pass the environment variables yourself directly.

It is imperative that you never expose the **DevTools** when your application is in production.

This includes immediately quitting your application if the command line parameters `--remote-debugging-port` or `--inspect` are present. See [generally here](https://github.com/electron/electron/issues/10445) and [here](https://www.sitepoint.com/debugging-electron-application/).

<aside class="warning">
Be careful of spawning child processes. Child processes will inherit the parent application's environment variables. This could occur when you launch another application from within your application.
</aside>


# Deployment Advice

After [packaging your application](https://electronjs.org/docs/tutorial/application-packaging), there are some extra steps that should be performed.

<aside class="notice">The generated launcher must be placed in the same root path as the electron application.</aside>

## Windows

It is recommended that you change the launcher's icon to something more suitable. You can find [guidance here](https://github.com/electron/rcedit). 

## macOS

To force the app bundle to start the launcher instead of the usual electron executable, you will need to *potentially* modify the `Info.plist` (text) file found in the `/Contents` directory.

<aside class="notice">You may have to right-click the app bundle and click <strong>Show Package Content</strong>. After modifying the Info.plist file, you may need to rename the app bundle and change it back to see the changes.</aside>

```info
    <key>CFBundleExecutable</key>
    <string>launcher</string>
```

You can change the string that corresponds to the `CFBundleExecutable` key to the name of the launcher file.

# Warnings

Electron Vault will happily generate a Linux launcher application. Since Linux is open-source, it is possible that someone might create a spurious alteration to the kernel in order to intercept the secrets when they are transferred *via* environment variables. For this reason, use Electron Vault at your own risk.

If you must have a Linux version of your electron application, then consider a different set of credentials to Windows/macOS so your backend service can invalidate them if they are found to be compromised. 

<aside class="warning">There is no foolproof methodology to 100% secure your credentials. Electron Vault should make it substantially harder for a talented but unscrupulous individual or team to discover your secrets. Please refer to the License agreement for further details.</aside>

# Authentication

Electron Vault uses an API token to prevent abuse of service.

You can obtain a **FREE** API Token from [rocketlaunchr.cloud](https://rocketlaunchr.cloud).

Electron Vault expects the API Token to be included in all API requests as a query string parameter. It should look like this:

`?api_token=meowmeowmeow`

<aside class="notice">
You must replace <code>meowmeowmeow</code> with your personal API token.
</aside>

# API Documentation

## Create Launcher Application

> The Request Body:

```json
{
    "launcher_file_name":"launcher",
    "app_file_name": "electron-quick-start",
    "app_human_name": "my app",

    "linux": false,

    "secrets": {
        "key1": "secret1",
        "key2": "secret2"
    },

    "app_asar_md5": "946733973687a4c53a7ac7c01b6223d7",
    "electron_asar_md5":"4a934f2e168eabadf26a0f56726d7362",
    "custom_md5": {
        "mac:MacOS/electron-quick-start": "27ba4a7063de580efa42382f88e253be"
    },

    "messages": {
        "ERROR_MESSAGE_UNKNOWN": "Something went wrong.",
        "ERROR_MESSAGE_CORRUPT": "The application has been tampered with."
    },
    "encryption_complexity":5
}
```

> The Response Body:

```json
{
    "signed_url": "https://download-url.com",
    "archive_size_kb": 3015,
    "mac_size_kb": 9710,
    "win_size_kb": 9710,
    "linux_size_kb": 9710
}
```

This endpoint will create a Windows, macOS and possibly Linux launcher application, and then provide the url to download it.

### HTTP Request

`POST https://api.rocketlaunchr.cloud/electron-vault`

### JSON Body Parameters

Parameter | Required | Default | Description
--------- | ------- | ------- | -----------
launcher_file_name | false | launcher | If set to true, the result will also include cats.
app_file_name | true | - | The filename of the electron application. Don't provide the '.exe'
app_human_name | true | - | See below
linux | false | false | Sets whether a Linux launcher will be built
secrets | true | - | The keys for the secrets will be the environment variable keys
app_asar_md5 | true | - | The md5 hash of the app.asar file
electron_asar_md5 | false | - | The md5 hash of the electron.asar file
custom_md5 | false | - | See tamper detection below
messages | false | - | See below
encryption_complexity | true | -| See below

<aside class="success">
Remember — the url provided will expire after a period of <strong>24 hours</strong>.
</aside>


### Messages

> The default alert messages:

```json
{
  "ERROR_MESSAGE_UNKNOWN": "An unexpected error occurred.", 
  "ERROR_MESSAGE_CORRUPT": "The application is corrupted."
}
```

If the launcher detects that your application has been tampered with, it will display a message. It can be customized by setting the `ERROR_MESSAGE_CORRUPT` key.

For any other errors such as not being able to launch your electron application, you can customize the message by setting the `ERROR_MESSAGE_UNKNOWN` key.

### App Human Name

If the launcher needs to show a modal alert to display an error message, this setting sets the alert's title bar.

<aside class="notice">
It should be set to what your users know your application by.
</aside>

### Understanding the encryption complexity (n)

The secrets are encrypted using an encryption algorithm. The higher the encryption complexity, the harder it is to "crack". Generally speaking, as `n` increases, so does the launcher's filesize. There is a random element to the encryption, so it is not always the case.


n | 1 | 5 | 10 | 15 | 20 | 25 
-- | -- | -- | -- | -- | -- | -- 
kB| 1,697 | 3,015 | 3,019 | 4,047 | 6,213 | 5,305

n | 30 | 35 | 40 | 45 | 50 | 100 | 150
-- | -- | -- | -- | -- | -- | -- | -- 
kB| 5,671 | 5,802 | 5,939 | 6,125 | 6,811 | 7,836 | 8,476

n | 200 | 250 | 300 | 350 | 400 | 450 | 500
-- | -- | -- | -- | -- | -- | -- | --
kB| 9,171 | 9,581 | 9,710 | 10,248 | 10,614 | 10,762 | 10,908

**Average filesize after 10 runs for a given `n`**

### Tamper Detection

```json
  "custom_md5": {
    "mac:MacOS/electron-quick-start": "27ba4a7063de580efa42382f88e253be",
    "win:electron-quick-start.exe": "2342342ba4a7063de580efa42382f99",
    "linux:electron-quick-start": "5346348ba4a7063de580efa42382f99"
  },
```

Before the launcher attempts to run your application, it will scan the specified files for a given operating system and compare their md5 hash. 

This can be set by configuring the `custom_md5` which takes a JSON object.

The key must have a prefix of `win:`, `mac:` or `linux:` followed by the file path.
The value is the md5 hash as a string.

All files paths are relative to:

OS | Root Path
--------- | -----------
mac | /Contents 
win | / 
linux | / 

<aside class="notice">
For macOS, click "Show Package Content" after right-clicking the app bundle file to view the directory structure.
</aside>

<aside class="warning">
If the file does not exist or has a different md5 hash, the launcher will gracefully exit.
</aside>

# License

Electron Vault is provided with a modified MIT license (mMIT). It has all the freedoms that the standard MIT license has but with the additional constraint that the tool is not used intentionally for military purposes.


# Terms of Service

Electron Vault is free to use. As such, I will be hosting it on a super-tiny instance to cut costs. Please be patient at times of high demand, as compiling a custom launcher application is CPU and memory intensive.

The service is governed by "fair use" provisions. Please use the API sparingly.

The only request I make is that you provide feedback on your experiences. This includes advice on how to improve the service.

# Questions

If you have any further questions, feel free to email [rocketlaunchr-cto@gmail.com](mailto:rocketlaunchr-cto@gmail.com). Having said that I’m sure you will appreciate that I will ignore any questions that ask for further details regarding the security features.


