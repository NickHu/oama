# **oama** - OAuth credential MAnager

Many IMAP/SMTP clients, like [msmtp](https://marlam.de/msmtp/),
[fdm](https://github.com/nicm/fdm),
[isync](http://isync.sourceforge.net/),
[neomutt](https://github.com/neomutt/neomutt) or
[mutt](http://www.mutt.org/) can use OAuth2 access tokens but lack the
ability to renew and/or authorize OAuth2 credentials. The purpose of
`oama` is to provide these missing capabilities by acting as a kind of
smart password manager. In particular, access token renewal happens
automatically in the background transparent to the user.

The OAuth2 credentials are kept *encrypted* in a **back end**.

### Provided back-end methods

- [GNU PG](https://www.gnupg.org/) **encrypted files**.  These files are kept
  in the `$XDG_STATE_HOME/oama` directory. If the `XDG_STATE_HOME`
  environment variable is not set then it defaults to `$HOME/.local/state`.

- A **keyring service** provided by any password manager with a
  FreeDesktop.org Secret Service compatible API. *Some examples* of such
  password managers are:

    - [Gnome keyring](https://wiki.gnome.org/Projects/GnomeKeyring/)
    - [KDE Wallet Manager](https://apps.kde.org/kwalletmanager5/)
    - [KeePaasXC](https://keepassxc.org/)

## Usage

Invoking `oama` without any arguments print a help message listing the
available commands:

    oama - OAuth credential MAnager with store/lookup, renewal, authorization.

    Usage: oama [--version] [-c|--config <config>] [--debug] COMMAND

      Oama is an OAuth credential manager providing store/lookup, automatic renewal
      and authorization operations. The credentials are stored either in the Gnome
      keyring or in files encrypted by GnuPG. Oama is useful for IMAP/SMTP or other
      network clients which cannot authorize and renew OAuth tokens on their own.

    Available options:
      -h,--help                Show this help text
      --version                Show version
      -c,--config <config>     Configuration file
                               (default: "~/.config/oama/config.yaml")
      --debug                  Print HTTP traffic to stdout

    Available commands:
      access                   Get the access token for email
      show                     Show current credentials for email
      renew                    Renew the access token of email
      authorize                Authorize OAuth2 for service/email
      printenv                 Print the current runtime environment
      template                 Print the default config template

More detailed help for individual commands can also be generated by appending
`-h` after the command. Shell completion for `bash`, `zsh` and `fish` shells
are provided.

Before `oama` is fully operational you need to create the necessary
configuration files. See details in [Configuration](#configuration).

## Configuration

`oama` has a simple configuration system. When you run `oama` at the first
time it will create the initial config file `config.yaml` in the
`$XDG_CONFIG_HOME/oama` directory. If the XDG_CONFIG_HOME environment
variable is not set then it defaults to `$HOME/.config`. You need to edit the
initially created config file. This YAML file is commented explaining your
options, just follow the instructions there.

First select the method of storing the OAuth credentials. Then configure the
services you are going to use. There are two kinds of services the *builtin*
ones which `oama` already knows and the *user configured* ones. The current
*builtin* services are `google` and `microsoft`.

For a *builtin* service the minimum information you need to provide are
`client_id` and `client_secret`. For *user configured* service there are a few
more required config options. See the initially created config file for more
details.

You can see **all the configurable options** in the `services:` section of
the output of the `oama printenv` command.

### Application `client_id` and `client_secret`

For institutional accounts your organization should provide the
`client_{id,secret}` pair regardless who is the service provider.

For personal accounts you can register your own *client application* at your
service provider and obtain a `client_{id,secret}` pair.

- Microsoft: [Register an application](https://learn.microsoft.com/en-us/entra/identity-platform/quickstart-register-app)
- Google: [Credentials page](https://console.developers.google.com/apis/credentials)

If that is too much hassle then you can try to find and use one of the open
source email clients' `client_{id,secret}` pair. Most of these desktop clients
are already registered at many service providers.

### Google Organizational Account

Invoke `oama` with no login hint:

    oama authorize google <you@company.email> --nohint

### Microsoft accounts

The default `tenant` for a Microsoft account is `common` which is also
included in the `auth_endpoint` and `token_endpoint` URL-s. If you need to
use a different `tenant` value then it is enough to specify only the `tenant`
field the `*_endpoint` URL-s will be automatically changed too.
 
#### Microsoft Organizational Account

Invoke `oama` using your proper organizational email:

    oama authorize microsoft <you@company.email>

Then visit the `http://localhost:<portnumber>/start` page to perform the steps
below:

 - Click "Sign in with another account"
 - Click "Sign-in options"
 - Click "Sign in to an organization"
 - Put in the correct domain name which matches your organization address above
 - Log in with your credentials at the organization.

## Authorization

After configuration, you must run the `authorize` command:

    oama authorize <service> <email>

That is an interactive process involving a browser since you need to login
and authorize access to your email account. `oama` will lead you through this
process.

## Running `oama` remotely

Install and configure `oama` on the remote host. Chose a back-end, it can be
either GPG or KEYRING. Make sure that the back-end installed and **works as
desired** on the remote machine.

Pick a free, non-privileged `<port-number` and include

    redirect_uri: http://localhost:<port-number>

into your service provider configuration section.

Login into the remote host using the command below:

    ssh -L <port-number>:localhost:<port-number> <remote-host>

Start the authorization on the remote host as usual:

    oama authorize <service> <email>

Then just follow the instructions and open the suggested URL in a browser
running on your *local* machine.

## Logging

All transactions and exceptions are logged to `syslog`. If your OS using
`systemd` you can inspect the log with a command like below:

    journalctl --identifier oama --identifier msmtp --identifier fdm -e

## Installation

### Compiled static binaries

Each release contains compiled executables of `oama` which should work on
most Linux distributions. Currently, Linux/x86_64 and Linux/aarch64(arm64)
binaries are provided. Select the version you want to download from
[releases](https://github.com/pdobsan/oama/releases).

#### Archlinux

For Archlinux users there is also a package on AUR:
[oama-bin](https://aur.archlinux.org/packages/oama-bin)

### Building from sources

To build `oama` from source you need a Haskell development environment,
with `ghc 9.4.8` or higher. Either your platform's package system can provide
this or you can use [ghcup](https://www.haskell.org/ghcup/). Once you have
the `ghc` Haskell compiler and `cabal` etc. installed, follow the steps
below:

Clone this repository and invoke `cabal`:

    git clone https://github.com/pdobsan/oama
    cd oama
    cabal update
    cabal install --install-method=copy

That installs `oama` into the `~/.cabal/bin/` directory.

## Issues

Please, report any problems, questions, suggestions regarding `oama` by opening
an issue or by starting a discussion.

### Guidelines for opening an issue

- Make sure that you are using the latest version of `oama`.

- Before opening an issue search old issues (both open and closed) and check whether 
  similar problems have been raised or solved before.

- Attach the **complete output of the `oama printenv`** command. Do not
  remove lines, get confidential values redacted by replacing them with
  `<some explanation>`. In particular, indicate what kind of `client_id/secret`
  you are using. For example, `<my own app id registered with google>`.

- Indicate what kind of account(s) you are using that is who is the service
  provider and whether your account is personal or institutional.

- Send also full error messages and related syslog entries. Even when `oama` was called
  by another program which could have hidden its error messages you might see
  them in the syslog.

## Alternatives

The programs below solve similar problems as `oama` does but have different
takes on them.

- [mutt_oauth2.py](https://gitlab.com/muttmua/mutt/-/blob/master/contrib/mutt_oauth2.py)

- [email-oauth2-proxy](https://github.com/simonrob/email-oauth2-proxy)

- [pizauth](https://github.com/ltratt/pizauth)

  
## License

`oama` is released under the 3-Clause BSD License, see the file
[LICENSE](LICENSE).

