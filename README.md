# runit-configs

[Runit](http://smarden.org/runit/) configs for local development of various
services at work. Runit is a simple process supervisor available in most package
managers.

Start everything up:

```
runsvdir -P ./sv
```

It's important that you pass `-P` to `runsvdir`, so that the process group clean
up logic at service shutdown can actually be restricted to the relevant process
group... you run this config at your own risk.

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

## Further work

* Enable/disable services by symlinking directories into sv, so that runsvdir
  doesn't have to operate on everything in the repo.
