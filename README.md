# mailctl

`mailctl` is a utility to control an email system comprised of
[msmpt](https://marlam.de/msmtp/),
[fdm](https://github.com/nicm/fdm)
with email clients like
[neomutt](https://github.com/neomutt/neomutt)
or
[mutt](http://www.mutt.org/)
running on Linux or Unix like systems. `mailctl` acts as a proxy to password
managers and takes care of the management of OAuth2 authorization with service
providers like Google, Yahoo etc.

## Usage

`mailctl -h` generates this help message:

    mailctl - control an msmpt/fdm/mutt email system

    Usage: mailctl [--version] [-c|--config-file <config>] [--run-by-cron] COMMAND

      mailctl is a program to control an msmpt/fdm/mutt email system, using pass as
      pasword manager, and google OAuth2 method.

    Available options:
      -h,--help                Show this help text
      --version                Show version
      -c,--config-file <config>
                               Configuration file
      --run-by-cron            mailctl invoked by cron

    Available commands:
      password                 get the password
      access                   get the access token
      renew                    renew the access token
      authorize                authorize an email entry for Oauth2
      fetch                    get fdm to fetch all or the given accounts
      cron                     Manage running by cron
      list                     list all accounts in fdm's config
      printenv                 Print the current Environment

More detailed help for individual commands can also be generated:

    % mailctl cron -h
    Usage: mailctl cron [--status | --enable | --disable]

      Manage running by cron

    Available options:
      --status                 Show cron status.
      --enable                 Enable running by cron.
      --disable                Disable running by cron.
      -h,--help                Show this help text

### Shell completion
 
Shell completion for `bash`, `zsh` and `fish` shells can be automatically
generated. Here is how to do it for `zsh`:

      mailctl --zsh-completion-script $(which mailctl) > ~/.local/share/zsh/site-functions/_mailctl


## Configuartion

The files in the `configs/` directory are configuration templates for
`mailctl`, `msmtp` and `fdm`. The `configs/services-template.json` file
contains details for `google`, `microsoft` and `yahoo`. You need to provide
your own `client_id` and `client_secret` of your application or of a
suitable FOSS registered application.

Edit them to adjust to your environment.

The base password manager used here is
[pass](https://www.passwordstore.org/)
The Oauth2 credentials are kept encrypted using
[GNU PG](https://www.gnupg.org/).

Since both `pass` and `mailctl` are using `gpg` it is assumed that an
authorized `gpg-agent` is running.

`fdm` is run by `cron` with a `crontab` like this:

    PATH  = /YOUR_HOME_DIRECTORY/.cabal/bin:/usr/bin:/bin
    # fetch mail in every 10 minutes
    */10 * * * *  mailctl --run-by-cron fetch


## Installation

Clone this repository and invoke `cabal`:

    git clone https://github.com/pdobsan/mailctl.git
    cd mailctl
    cabal install

`mailctl` is known to build with `ghc 8.10.7` and `ghc 9.2.3`

### Logging

All transactions and exceptions are logged to `syslog`. If your OS using
`systemd` you can inspect the log with the command below:

    journalctl --identifier fdm --identifier mailctl --identifier msmtp -e


## License

`mailctl` is released under the 3-Clause BSD License, see the file
[LICENSE](LICENSE).

