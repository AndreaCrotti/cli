# babashka.cli

Easy command line parsing for Clojure.

## API

See [API.md](API.md).

## Rationale

Parsing command line arguments in clojure and babashka CLIs often are in the form of

``` clojure
<subcommand> :opt1 v1 :opt2 :v2
```

This library eases that style of command line parsing.

This library does not support validation, or everything that you might expect
from an arg-parse library. I think a lot of these things can be done using spec
or otherwise, nowadays.

It does not convert options into EDN automatically which, arguably, is more
convenient for command line usage. This library does offer a light-weight way to
coerce strings.

Adding support for `babashka.cli` coercions to Clojure functions does not
introduce a dependency on `babashka.cli` itself.  It can be done via metadata
and core functions. Perhaps doing so will cause less friction with shell usage
(Windows, etc). You can have support on the same function for both `clojure -X`
and `clojure -M` style invocations without writing extra boilerplate.

## Quickstart

``` clojure
(require '[babashka.cli :as cli])
(cli/parse-args ["server" ":port" "1339"] {:coerce {:port parse-long}})
;;=> {:cmds ["server"] :opts {:port 1339}}
```

## Usage with the clojure CLI

In your `deps.edn` `:aliases` entry, add:

``` clojure
:exec {:deps {org.babashka/cli {:git/url "https://github.com/babashka/cli"
                                :git/sha "<latest-sha>"}}
       :main-opts ["-m" "babashka.cli.exec"]}
```

There-after you can call any function that accepts a map argument. E.g.:

``` clojure
$ clojure -M:exec clojure.core/prn :a 1 :b 2
{:a "1", :b "2"}
```

Functions that are annotated with `:babashka/cli` metadata can add coerce options:

``` clojure
(ns my-ns)

(defn foo
  {:babashka/cli {:coerce {:a symbol :b parse-long}}}
  ;; map argument:
  [m]
  ;; print map argument:
  (prn m))
```

``` clojure
$ clojure -M:exec my-ns/foo :a foo/bar :b 2 :c vanilla
{:a foo/bar, :b 2, :c "vanilla"}
```

Note that any library can add support for babashka CLI without depending on
babashka CLI.

An example that combines `babashka.cli` and another tool:

``` clojure
:exec {:deps {org.babashka/cli {:git/url "https://github.com/babashka/cli"
                                :git/sha "<latest-sha>"}}
       :main-opts ["-m" "babashka.cli.exec"]}
:new {:extra-deps {com.github.seancorfield/clj-new {:mvn/version "1.2.381"}}}
```

``` clojure
$ clojure -M:exec:new :template app :name myname/myapp
```

## License

Copyright © 2022 Michiel Borkent

Distributed under the MIT License. See LICENSE.
