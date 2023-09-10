re2 [![Build Status](https://github.com/mudge/re2/actions/workflows/tests.yml/badge.svg?branch=main)](https://github.com/mudge/re2/actions)
===

Ruby bindings to [RE2][], a "fast, safe, thread-friendly alternative to
backtracking regular expression engines like those used in PCRE, Perl, and
Python".

**Current version:** 2.4.3  
**Supported Ruby versions:** 2.6, 2.7, 3.0, 3.1, 3.2  
**Bundled RE2 version:** libre2.11 (2023-11-01)  
**Supported RE2 versions:** libre2.0 (< 2020-03-02), libre2.1 (2020-03-02), libre2.6 (2020-03-03), libre2.7 (2020-05-01), libre2.8 (2020-07-06), libre2.9 (2020-11-01), libre2.10 (2022-12-01), libre2.11 (2023-07-01)

Installation
------------

The gem comes bundled with a version of [RE2][] and will compile itself (and
any dependencies) on install. As compilation can take a while, precompiled
native gems are available for Linux, Windows and macOS.

In v2.0 and later, precompiled native gems are available for Ruby 2.6 to 3.2
on these platforms:

- `aarch64-linux` (requires: glibc >= 2.29)
- `arm-linux` (requires: glibc >= 2.29)
- `arm64-darwin`
- `x64-mingw32` / `x64-mingw-ucrt`
- `x86-linux` (requires: glibc >= 2.17)
- `x86_64-darwin`
- `x86_64-linux` (requires: glibc >= 2.17)

If you wish to opt out of using the bundled libraries, you will need RE2
installed as well as a C++ compiler such as [gcc][] (on Debian and Ubuntu, this
is provided by the [build-essential][] package). If you are using macOS, I
recommend installing RE2 with [Homebrew][] by running the following:

    $ brew install re2

If you are using Debian, you can install the [libre2-dev][] package like so:

    $ sudo apt-get install libre2-dev

Recent versions of RE2 require [CMake](https://cmake.org) and a compiler with
C++14 support such as [clang](http://clang.llvm.org/) 3.4 or
[gcc](https://gcc.gnu.org/) 5.

If you are using a packaged Ruby distribution, make sure you also have the
Ruby header files installed such as those provided by the [ruby-dev][] package
on Debian and Ubuntu.

You can then install the library via RubyGems with `gem install re2 --platform=ruby --
--enable-system-libraries` or `gem install re2 --platform=ruby -- --enable-system-libraries
--with-re2-dir=/path/to/re2/prefix` if RE2 is not installed in any of the
following default locations:

* `/usr/local`
* `/opt/homebrew`
* `/usr`

Alternatively, you can set the `RE2_USE_SYSTEM_LIBRARIES` environment variable instead of passing `--enable-system-libraries` to the `gem` command.

If you're using Bundler, you can use the
[`force_ruby_platform`](https://bundler.io/v2.3/man/gemfile.5.html#FORCE_RUBY_PLATFORM)
option in your Gemfile.

Windows users attempting to compile [abseil] must use pkgconf 2.1.0 or
later, or builds will fail with [`undefined reference` errors](https://github.com/pkgconf/pkgconf/issues/322):

    pacman -Sy mingw64/mingw-w64-x86_64-pkgconf

This is not needed when using the precompiled gem or building against a system RE2 library.

Documentation
-------------

Full documentation automatically generated from the latest version is
available at <http://mudge.name/re2/>.

Note that RE2's regular expression syntax differs from PCRE and Ruby's
built-in [`Regexp`][Regexp] library, see the [official syntax page][] for more
details.

Usage
-----

While re2 uses the same naming scheme as Ruby's built-in regular expression
library (with [`Regexp`](http://mudge.name/re2/RE2/Regexp.html) and
[`MatchData`](http://mudge.name/re2/RE2/MatchData.html)), its API is slightly
different:

```console
$ irb -rubygems
> require 're2'
> r = RE2::Regexp.new('w(\d)(\d+)')
=> #<RE2::Regexp /w(\d)(\d+)/>
> m = r.match("w1234")
=> #<RE2::MatchData "w1234" 1:"1" 2:"234">
> m[1]
=> "1"
> m.string
=> "w1234"
> m.begin(1)
=> 1
> m.end(1)
=> 2
> r =~ "w1234"
=> true
> r !~ "bob"
=> true
> r.match("bob")
=> nil
```

As
[`RE2::Regexp.new`](http://mudge.name/re2/RE2/Regexp.html#initialize-instance_method)
(or `RE2::Regexp.compile`) can be quite verbose, a helper method has been
defined against `Kernel` so you can use a shorter version to create regular
expressions:

```console
> RE2('(\d+)')
=> #<RE2::Regexp /(\d+)/>
```

Note the use of *single quotes* as double quotes will interpret `\d` as `d` as
in the following example:

```console
> RE2("(\d+)")
=> #<RE2::Regexp /(d+)/>
```

As of 0.3.0, you can use named groups:

```console
> r = RE2::Regexp.new('(?P<name>\w+) (?P<age>\d+)')
=> #<RE2::Regexp /(?P<name>\w+) (?P<age>\d+)/>
> m = r.match("Bob 40")
=> #<RE2::MatchData "Bob 40" 1:"Bob" 2:"40">
> m[:name]
=> "Bob"
> m["age"]
=> "40"
```

As of 0.6.0, you can use `RE2::Regexp#scan` to incrementally scan text for
matches (similar in purpose to Ruby's
[`String#scan`](http://ruby-doc.org/core-2.0.0/String.html#method-i-scan)).
Calling `scan` will return an `RE2::Scanner` which is
[enumerable](http://ruby-doc.org/core-2.0.0/Enumerable.html) meaning you can
use `each` to iterate through the matches (and even use
[`Enumerator::Lazy`](http://ruby-doc.org/core-2.0/Enumerator/Lazy.html)):

```ruby
re = RE2('(\w+)')
scanner = re.scan("It is a truth universally acknowledged")
scanner.each do |match|
  puts match
end

scanner.rewind

enum = scanner.to_enum
enum.next #=> ["It"]
enum.next #=> ["is"]
```

As of 1.5.0, you can use `RE2::Set` to match multiple patterns against a
string. Calling `RE2::Set#add` with a pattern will return an integer index of
the pattern. After all patterns have been added, the set can be compiled using
`RE2::Set#compile`, and then `RE2::Set#match` will return an `Array<Integer>`
containing the indices of all the patterns that matched.

```ruby
set = RE2::Set.new
set.add("abc") #=> 0
set.add("def") #=> 1
set.add("ghi") #=> 2
set.compile #=> true
set.match("abcdefghi") #=> [0, 1, 2]
set.match("ghidefabc") #=> [2, 1, 0]
```

As of 1.6.0, you can use [Ruby's pattern matching](https://docs.ruby-lang.org/en/3.0/syntax/pattern_matching_rdoc.html) against `RE2::MatchData` with both array patterns and hash patterns:

```ruby
case RE2('(\w+) (\d+)').match("Alice 42")
in [name, age]
  puts "My name is #{name} and I am #{age} years old"
else
  puts "No match!"
end
# My name is Alice and I am 42 years old


case RE2('(?P<name>\w+) (?P<age>\d+)').match("Alice 42")
in {name:, age:}
  puts "My name is #{name} and I am #{age} years old"
else
  puts "No match!"
end
# My name is Alice and I am 42 years old
```

Encoding
--------

Note RE2 only supports UTF-8 and ISO-8859-1 encoding so strings will be
returned in UTF-8 by default or ISO-8859-1 if the `:utf8` option for the
`RE2::Regexp` is set to false (any other encoding's behaviour is undefined).

For backward compatibility: re2 won't automatically convert string inputs to
the right encoding so this is the responsibility of the caller, e.g.

```ruby
# By default, RE2 will process patterns and text as UTF-8
RE2(non_utf8_pattern.encode("UTF-8")).match(non_utf8_text.encode("UTF-8"))

# If the :utf8 option is false, RE2 will process patterns and text as ISO-8859-1
RE2(non_latin1_pattern.encode("ISO-8859-1"), :utf8 => false).match(non_latin1_text.encode("ISO-8859-1"))
```

Features
--------

* Pre-compiling regular expressions with
  [`RE2::Regexp.new(re)`](https://github.com/google/re2/blob/2016-02-01/re2/re2.h#L100),
  `RE2::Regexp.compile(re)` or `RE2(re)` (including specifying options, e.g.
  `RE2::Regexp.new("pattern", :case_sensitive => false)`

* Extracting matches with `re2.match(text)` (and an exact number of matches
  with `re2.match(text, number_of_matches)` such as `re2.match("123-234", 2)`)

* Extracting matches by name (both with strings and symbols)

* Checking for matches with `re2 =~ text`, `re2 === text` (for use in `case`
  statements) and `re2 !~ text`

* Incrementally scanning text with `re2.scan(text)`

* Search a collection of patterns simultaneously with `RE2::Set`

* Checking regular expression compilation with `re2.ok?`, `re2.error` and
  `re2.error_arg`

* Checking regular expression "cost" with `re2.program_size`

* Checking the options for an expression with `re2.options` or individually
  with `re2.case_sensitive?`

* Performing a single string replacement with `pattern.replace(replacement,
  original)`

* Performing a global string replacement with
  `pattern.replace_all(replacement, original)`

* Escaping regular expressions with
  [`RE2.escape(unquoted)`](https://github.com/google/re2/blob/2016-02-01/re2/re2.h#L418) and
  `RE2.quote(unquoted)`

* Pattern matching with `RE2::MatchData`

Contributions
-------------

* Thanks to [Jason Woods](https://github.com/driskell) who contributed the
  original implementations of `RE2::MatchData#begin` and `RE2::MatchData#end`.
* Thanks to [Stefano Rivera](https://github.com/stefanor) who first contributed
  C++11 support.
* Thanks to [Stan Hu](https://github.com/stanhu) for reporting a bug with empty
  patterns and `RE2::Regexp#scan`, contributing support for libre2.11
  (2023-07-01) and for vendoring RE2 and abseil and compiling native gems in
  2.0.
* Thanks to [Sebastian Reitenbach](https://github.com/buzzdeee) for reporting
  the deprecation and removal of the `utf8` encoding option in RE2.
* Thanks to [Sergio Medina](https://github.com/serch) for reporting a bug when
  using `RE2::Scanner#scan` with an invalid regular expression.
* Thanks to [Pritam Baral](https://github.com/pritambaral) for contributing the
  initial support for `RE2::Set`.
* Thanks to [Mike Dalessio](https://github.com/flavorjones) for reviewing the
  precompilation of native gems in 2.0.
* Thanks to [Peter Zhu](https://github.com/peterzhu2118) for
  [ruby_memcheck](https://github.com/Shopify/ruby_memcheck) and helping find
  the memory leaks fixed in 2.1.3.
* Thanks to [Jean Boussier](https://github.com/byroot) for contributing the
  switch to Ruby's `TypedData` API and the resulting garbage collection
  improvements in 2.4.0.

Contact
-------

All issues and suggestions should go to [GitHub Issues](https://github.com/mudge/re2/issues).

License
-------

This library is licensed under the BSD 3-Clause License, see `LICENSE.txt`.

Dependencies
------------

The source code of [RE2][] is distributed in the `ruby` platform gem. This code is licensed under the BSD 3-Clause License, see `LICENSE-DEPENDENCIES.txt`.

The source code of [Abseil][] is distributed in the `ruby` platform gem. This code is licensed under the Apache License 2.0, see `LICENSE-DEPENDENCIES.txt`.

  [RE2]: https://github.com/google/re2
  [gcc]: http://gcc.gnu.org/
  [ruby-dev]: http://packages.debian.org/ruby-dev
  [build-essential]: http://packages.debian.org/build-essential
  [Regexp]: http://ruby-doc.org/core/classes/Regexp.html
  [MatchData]: http://ruby-doc.org/core/classes/MatchData.html
  [Homebrew]: http://mxcl.github.com/homebrew
  [libre2-dev]: http://packages.debian.org/search?keywords=libre2-dev
  [official syntax page]: https://github.com/google/re2/wiki/Syntax
  [Abseil]: https://abseil.io
