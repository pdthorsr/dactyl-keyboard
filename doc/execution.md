# Execution guide

This document describes multiple ways to run the DMOTE application.
Please read the [introduction](intro.md) first.

## Method 1: GNU Make

Automation using [GNU Make](https://www.gnu.org/software/make/) has both pros
and cons:

* Pro: Convenient for a small set of designs.
* Pro: Fast iterations when you change a configuration.
* Con: You need Make.
* Con: Advanced configuration selection is awkward.
* Con: Slow iterations when you change application source code.

### Examples

To produce OpenSCAD files for the default configuration, just run `make` in
your terminal, from the project folder. As of v0.5.0, the default configuration
is a 62-key DMOTE.

To produce something more similar to Tom Short’s original Dactyl-ManuForm
instead of a DMOTE, run `make dactylmanuform_46key` instead of `make`. There is
a very short list of such canonical build targets in the `Makefile`. Notice
that Make echoes its commands to your terminal so you can see which
[YAML](https://en.wikipedia.org/wiki/YAML) files are used for each target, and
in what order.

If you want to weave in more such files, you can put more YAML paths on the
command line or name an intermediate canonical target, or do both. However,
this is not what Make is for, so you will need to finish with a named target
that actually calls the application, as in `make vis dmote_62key`, where
`dmote_62key` is the otherwise implicit default, and `vis` is a predefined
intermediate target for visualization. Ultimately, this is a limited method,
and therefore not the only method.

### Further automation

The `transpile.sh` shell script in the project root uses Make as well as
`inotify` to automatically retrigger Make on a named target when you save
changes to the configuration or application. `transpile.sh` can also send build
artefacts to a render farm with `rsync`. This may be convenient on Linux but
is not very portable.

## Method 2: Running with Leiningen

[Leiningen](https://leiningen.org/) is an automation tool used to manage the
DMOTE project. You can use it to run the application as a one-off CLI program.
Pros and cons:

* Pro: Portable. Pure Clojure, no extra tools.
* Pro: Good configuration control.
* Pro: Rendering to STL is automated. It’s just an extra flag.
* Con: Slow iterations no matter what you change.

Note: The default configuration with Leiningen is built into the application
itself and is not the same as you get with a plain `make`.

### Examples

For OpenSCAD files identical to what you get with a plain `make`, run `lein run
-c config/base.yaml -c config/dmote/base.yaml`. To render to STL automatically,
add the `--render` flag.

Notice that you are naming specific YAML files and combining them into one
configuration, in the order of your choice. This gives you full control over
the composite configuration so you can more easily work with your own original
projects.

Put your own file(s) last to get the most power. For example, to override all
the ALPS-style key mounts with MX-style mounts on a DMOTE, call `lein run -c
config/base.yaml -c config/dmote/base.yaml -c config/dmote/mx.yaml`.

## Method 3: REPL

The [REPL](https://clojure.org/guides/repl/introduction) is a command line
within the application itself. Pros and cons:

* Pro: Portable. You can get a REPL with Leiningen.
* Pro: Fast iterations even if you’re changing source code.
* Con: You need to know what you’re doing.

### Examples

Run `lein repl` and work interactively from there.

Let’s suppose you’re editing the body module
(`src/dactyl_keyboard/cad/body.clj`) in another window. To reload the body
module into the running application and then use it to build the bundled DMOTE
configuration without having to restart anything, enter these two lines at the
REPL prompt:

```clojure
(use 'dactyl-keyboard.cad.body :reload)
(run {:configuration-file ["config/base.yaml" "config/dmote/base.yaml"]})
```

Notice that `:configuration-file` corresponds directly to the `-c` flag used
with `lein run`. `-c` is short for `--configuration-file` and appends its
argument to the vector you recreate manually at the REPL.
