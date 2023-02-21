# runit-configs

[Runit](http://smarden.org/runit/) configs for local development of various
services at work. Runit is a simple process supervisor available in most package
managers.

**Before you start**: this repository is not intended to be a generic tool. It
is intended as something to be customised. Read the `env` script, which is
sourced by most (all?) control scripts. It makes an assumption about your
environment: namely that all services are checked out in a directory
`$apps_base`. If that is not the case, you likely can't get much value from this
repo, without making it the case (e.g. via moving, or symlinks).

Read each service's `run` (and `finish`, if present) script. As well as
containing a startup command that may well make sense for you, e.g. `go run
...`, or `npm run dev`, they also contain assumptions about my personal
workflow! I use [asdf-vm](https://asdf-vm.com/), direnv, and asdf-direnv, so I'm
sourcing `.envrc` files everywhere to set my tooling to the correct versions.
This probably isn't what you are doing, and you will need to customise the
scripts!

Overall, please understand that this brings little to the party except a list of
services, and some suggestions for control scripts that are probably _close_ to
things that will work for developers of those services after some customisation,
and a process group hack that I take no legal responsibility for.

Enable the services you want:

```
# Get service names from `ls ./available`
./enable registry
./enable app-ui
```

You only have to do that once.

Start everything up:

```
runsvdir -P ./sv
```

It's important that you pass `-P` to `runsvdir`, so that the process group clean
up logic at service shutdown can actually be restricted to the relevant process
group... you run this config at your own risk.

**If you don't pass -P to `runsvdir`, each control script might kill a lot more
processes than you might guess on shutdown.**

Tail all logs:

```
tail -F ./sv/**/log/current
```

See what's up:

```
sv status ./sv/*
```

Don't interrupt that runsvdir process yet if you want all services to stop
gracefully. To stop all services and completely shut everything down, you can
send SIGHUP to the runsvdir process. If you're only running one:

```
pkill -HUP runsvdir
```

To stop services in preparation for starting them again (runsv monitors will all
keep running):

```
sv stop ./sv/*
```

When you've been making changes to a service, and want to kick the tyres
integrating it with everything else:

```
# Assuming your working dir is in the service's repo
sv restart ../runit-configs/sv/$service_name
```

To disable a service so that it doesn't start when you run `runsvdir`:

```
# For example
unlink ./sv/registry
```

## Why are you doing that thing with process groups?

The services being monitored here are not designed to be daemons, they are
development commands that were designed to be run in the foreground, and fork
arbitrarily many times. A typical command will call a make, which calls `go
run`, which then becomes the parent of the service process itself. A developer
will typically interrupt the foregrounded service process, which causes `go run`
to exit once it's done waiting, and likewise with make. As a process supervisor,
we have to get creative with how to kill our whole descendent tree.

Ensuring we run each service in a new process group, and killing that whole
process group, seemed good enough for a developer laptop flow. Something with
pid namespaces and PR_SET_CHILD_SUBREAPER could probably be made to work on
Linux, and would be extremely cool, but is a non-starter on Mac.
