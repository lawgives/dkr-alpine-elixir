# Elixir on Alpine Linux

This is based on `bitwalker/docker-elixir-phoenix`, but only runs Elixir. It specifically
compiles against `lawgives/dkr-alpine-erlang`, does not include node.js, and does not use
edge Alpine packages. The README is from the original, and does not reflect how this will
actually run. This is intended to work with Exrm (or Distillery whenever that comes out
of beta).

## Usage

NOTE: This image is intended to run in unprivileged environments, it sets the home directory to `/home/app`, and makes it globally
read/writeable. If run with a random, high-index user account (say 1000001), the user can run an app, and that's about it. If run
with a user of your own creation, this doesn't apply (necessarily, you can of course implement the same behaviour yourself).
It is highly recommended that you add a `USER default` instruction to the end of your Dockerfile so that your app runs in a non-elevated context.

To boot straight to a prompt in the image:

```
$ docker run --rm -it --user=9999 legalio/alpine-elixir iex
Erlang/OTP 18 [erts-7.3] [source] [64-bit] [smp:2:2] [async-threads:10] [kernel-poll:false]

Interactive Elixir (1.2.4) - press Ctrl+C to exit (type h() ENTER for help)
iex(1)>
BREAK: (a)bort (c)ontinue (p)roc info (i)nfo (l)oaded
       (v)ersion (k)ill (D)b-tables (d)istribution
a
```

Extending for your own application:

```dockerfile
FROM bitwalker/alpine-elixir-phoenix:latest

# Set exposed ports
EXPOSE 5000
ENV PORT=5000 MIX_ENV=prod

# Cache elixir deps
ADD mix.exs mix.lock ./
RUN mix do deps.get, deps.compile

ADD . .

# Run frontend build, compile, and digest assets
RUN brunch build --production && \
    mix do compile, phoenix.digest

USER default

CMD ["mix", "phoenix.server"]
```

It is recommended when using this that you have the following in `.dockerignore` when running `docker build`:

```
_build
deps
node_modules
test
```

This will keep the payload smaller and will also avoid any issues when compiling dependencies.

## License

MIT
