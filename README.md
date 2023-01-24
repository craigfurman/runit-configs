# runit-configs

[Runit](http://smarden.org/runit/) configs for local development of various
services at work. Runit is a simple process supervisor available in most package
managers.

Start everything up:

```
runsvdir ./sv
```

Tail all logs:

```
tail -F ./sv/**/log/current
```

See what's up:

```
sv status ./sv/*
```

Don't interrupt that runsvdir process yet if you want all services to stop
gracefully. To stop all services:

```
sv stop ./sv/*
```

When you've been making changes to a service, and want to kick the tyres
integrating it with everything else:

```
# Assuming you're working dir is in the service's repo
sv restart ../runit-configs/sv/$service_name
```
