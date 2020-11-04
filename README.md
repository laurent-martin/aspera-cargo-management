<h1>Remote Management of IBM Aspera Cargo</h1>

IBM Aspera Cargo version: 3.2

IBM Aspera Cargo documentation can be found here:

[https://www.ibm.com/support/knowledgecenter/SSH2ME](https://www.ibm.com/support/knowledgecenter/SSH2ME)

# Concepts

Cargo is run as a service (Windows) or Daemon (Mac/Linux).

It maintains its configuration in internal Databases (`asperacargo.data` and `keychain.data`) located in `<install>/var`

  * Macos: `/Library/Aspera/Cargo/var`
  * Windows: `C:\Program Files (x86)\Aspera\Aspera Cargo`
  * Linux: `/opt/aspera/cargo`

The configuration is managed using the cargo command line itself using options:

* `-f` to update the configuration with a json file
* `-w` to read the current configuration into a json file
* `-u` to delete all current configuration (deletes `asperacargo.data`)

Those options require that cargo is not running.

The procedure for scripted cargo configuration management is:

* stop cargo
* change configuration: save, reset or load
* start cargo

# Remote management

Any remote management of Cargo will rely on the mechanism highlighted in the previous section.

For remote management, a remote management system must be in place.

On any system, it can be as simple as a remote SSH connection. (Macos and Linux have this in standard, and Windows 10 supports SSH as well).

It could also be some WS-Management platform (e.g. WinRM)

# Examples

## remote management of cargo on MacOS from a Central Linux

On the central server:

```
# cat<<EOF>newconf.json
{
    "conf": {
        "accounts": [
            {
                "authentication_type": "BASIC",
                "host": "https://sample.com",
                "name": "Sample Faspex",
...
}
EOF
# ssh admin@192.168.0.223 /Library/Aspera/cargo/sbin/cargoctl stop
# ssh admin@192.168.0.223 /Library/Aspera/cargo/sbin/cargoctl reset
# ssh admin@192.168.0.223 /Library/Aspera/cargo/sbin/cargoctl load - < newconf.json
# ssh admin@192.168.0.223 /Library/Aspera/cargo/sbin/cargoctl start
```

Or alternatively in one line:

```
# ssh admin@192.168.0.223 '/Library/Aspera/cargo/sbin/cargoctl load =' <<EOF
{
    "conf": {
...
    }
}
```

On Windows, a similar script can be written with PowerShell.

# Note on Mac version

## Better startup

The way to start cargo on MacOS is currently very basic (see documentation of v3.2).

A better method is:

* startup

```
sudo nohup /Library/Aspera/Cargo/sbin/cargo > /dev/null &
```

* shutdown:

```
sudo killall cargo
```

## Even better startup

An alternate install/startup script using `launchd`/`launchctl` is provided here: `cargoctl`

```
$ sudo ./cargoctl
Usage: cargoctl install|uninstall|start|stop|status|logfolder|journal|save|load|reset|debug
    save - : display configuration
    load - : load configuration from stdin
    load + : stop daemon, load from stdin, start daemon
    load = : stop daemon, delete configuration, load from stdin, start daemon
```
