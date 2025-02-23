# Want access?

1. You must read literally this entire README. It is critically
   important that you do so.
2. Open a PR adding yourself to `users.nix`.

I'll grant access to well known members of the community, and people
well known members in the community trust.

## Notes on Security and Safety

***TLDR:*** a trusted but malicious actor could hack your system through
this builder. Do not use this builder for secret builds. Be careful
what you use this system for. Do not trust the results. For a more
nuanced understanding, read on.

For someone to use a server as a remote builder, they must be a
`trusted-user` on the remote builder. `man nix.conf` has this to say
about Trusted Users:

> User that have additional rights when connecting to the Nix daemon,
> such as the ability to specify additional binary caches, or to
> import unsigned NARs.
>
> Warning: The users listed here have the ability to compromise the
> security of a multi-user Nix store. For instance, they could install
> Trojan horses subsequently executed by other users. So you should
> consider carefully whether to add users to this list.

Nix's model of remote builders requires users to be able to directly
import files in to the Nix store, and there is no guarantee what they
import hasn't been maliciously modified.

The following is written as me, @winterqt:

I trust everyone who has access, but with limits:

1. ***DO NOT*** trust this builder for systems that contain private
   data or tools.

2. ***DO NOT*** trust this builder to make binary bootstrap tools,
   because we have to trust those bootstrap tools for a long time to
   not be compromised.

3. ***DO NOT*** trust this builder to make tools used to make binary
   bootstrap tools, because we have to trust those bootstrap tools for
   a long time to not be compromised.

IF YOU ARE: making binary bootstrap tools, please only use tools
built by Hydra on a system which have never been exposed to things
built from this server.

# Configuring your computer for remote builds

First, put this in your `configuration.nix`:

```nix
{
  programs.ssh.knownHosts."darwin-build-box.winter.cafe".publicKey =
    "ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIB0io9E0eXiDIEHvsibXOxOPveSjUPIr1RnNKbUkw3fD";

  nix = {
    distributedBuilds = true;
    buildMachines = [
      {
        hostName = "darwin-build-box.winter.cafe";
        maxJobs = 4;
        sshKey = "/root/a-private-key";
        sshUser = "your-user-name";
        systems = [ "aarch64-darwin" "x86_64-darwin" ];
      }
    ];
  };
}
```

**Note:** Make sure the SSH key specified above does *not* have a
password, otherwise `nix-build` will give an error along the lines of:

> unable to open SSH connection to
> 'ssh://your-user-name@darwin-build-box.winter.cafe': cannot connect to
> 'your-user-name@darwin-build-box.winter.cafe'; trying other available
> machines...

Then run an initial SSH connection as root to setup the trust
fingerprint:

```
$ sudo su
# ssh your-user-name@darwin-build-box.winter.cafe -i /root/a-private-key
```

The fingerprint should always be:

```
ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIB0io9E0eXiDIEHvsibXOxOPveSjUPIr1RnNKbUkw3fD
```

***If it is not, please open an issue!***

Finally, `nix-build . -A hello --argstr system aarch64-darwin`.

If this doesn't work, reach out and I can help debug.

# Want to support this?

The hosting costs for this machine are paid for by the [Nix 🖤 macOS Collective](https://opencollective.com/nix-macos). If you'd like to support not only this machine but also toonn's work on the SDK bump, consider contributing.

# Acknowledgements

- [Domen Kožar](https://github.com/domenkozar), for running the [Nix 🖤 macOS Collective](https://opencollective.com/nix-macos).
- [Graham Christensen](https://github.com/grahamc), for running the [aarch64 build box](https://github.com/nix-community/aarch64-build-box), where I took the structure of this README from.
