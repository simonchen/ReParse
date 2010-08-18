# ReParse #

ReParse is a parser combinator library for Javascript like [Parsec][1]
for Haskell.

## Installation ##

Download `lib/reparse.js` and add it to your project.

[1]: http://legacy.cs.uu.nl/daan/parsec.html

## API ##

To use ReParse, construct a `new ReParse(input)` object, then call the
`.start()`, passing it the top-level production of your grammar.  Each
production may be a RegExp or a function.

A RegExp will produce the first captured group or the entire value it
matched if there are no groups.  Functions are called with `this`
bound to the parser.  They take no arguments.  A function should
invoke one or more parser methods, optionally transforming the results
into an appropriate return value.

This is a grammar for a subset of JSON that supports arrays and
positive integers.  See `examples/json.js` for a more complete
example:

    function parse(data) {
      return (new ReParse(data, true)).start(value);
    }

    function value() {
      return this.choice(number, array);
    }

    function array() {
      return this.between(/^\[/, /^\]/, elements);
    }

    function elements() {
      return this.sepBy(value, /^,/);
    }

    function number() {
      return iPart(this.match(/^\d+/));
    }

    function iPart(digits) {
      for (var i = 0, l = digits.length, r = 0; i < l; i++)
        r = r * 10 + digits.charCodeAt(i) - 48;
      return r;
    }

ReParse indicates failure internally using exceptions, so it's safe to
use the result of any production immediately without checking for an
error state.

### ReParse(input, [ignorews]) ###

Create a new parser for an `input` string.  If `ignorews` is `true`,
skip whitespace before and after each production.

#### .start(method) ####

Return the value produced by `method`.  All input must be consumed.

#### .eof() ####

Return `true` if the input is exhausted.

#### .fail() ####

Call this method to indicate that a production has failed; an
exception will be raised and caught by the previous production.

#### .produce(method) ####

Apply the production `method` to the input and return the result.

#### .maybe(method) ####

Apply the production `method` to the input, restoring the input to its
previous state if it fails.

#### .option(method, otherwise) ####

Try to produce `method`.  If it fails, restore the input and return
`otherwise`.

#### .match(regexp) ####

Match a regular expression against the input, returning the first
captured group.  If no group is captured, return the matched string.

#### .choice(alt1, alt2, ...) ####

Try several alternatives, returning the value of the first one that's
successful.

#### .seq(group1, group2, ...) ####

All groups must match the input.  Return an array of capture groups
like a regular expression match would.  Index 0 is the entire matched
string, index 1 corresponds to `group1`, etc.

#### .many(method) ####

Return an array of zero or more values produced by `method`.

#### .many1(method) ####

Return an array of one or more values produced by `method`.

#### .between(left, right, body) ####

This is equivalent to `.seq(left, body, right)[0]`.

#### .skip(method) ####

Ignore zero or more instances of `method`.  Return the parser object.

#### .skip1(method) ####

Ignore one or more instances of `method`.  Return the parser object.

#### .skipWS() ####

Ignore whitespace.  Return the parser object.

#### .sepBy(method, sep) ####

Return an array of zero or more values produced by `method`.  Each
value is separated by `sep`.

#### .sepBy1(method, sep) ####

Return an array of one or more values produced by `method`.  Each
value is separated by `sep`.

#### .endBy(method, end) ####

This is equivalent to `.many(method)` followed by `.option(end)`.

#### .endBy1(method, end) ####

This is equivalent to `.many1(method)` followed by `.option(end)`.

#### .sepEndBy(method, sep) ####

Return an array of zero or more values produced by `method`.  Each
value is separated by `sep` and the entire sequence is optionally
terminated by `sep`.

#### .sepEndBy1(method, sep) ####

Return an array of one or more values produced by `method`.  Each
value is separated by `sep` and the entire sequence is optionally
terminated by `sep`.

## Compatibility ##

ReParse has been tested with Node.JS version `v0.1.103`.  It should
work in a browser if you comment out the `exports` line.

## License ##

Copyright (c) 2010, Ben Weaver &lt;ben@orangesoda.net&gt;
All rights reserved.

Redistribution and use in source and binary forms, with or without
modification, are permitted provided that the following conditions are
met:

* Redistributions of source code must retain the above copyright
  notice, this list of conditions and the following disclaimer.

* Redistributions in binary form must reproduce the above copyright
  notice, this list of conditions and the following disclaimer in the
  documentation and/or other materials provided with the distribution.

* Neither the name of the <organization> nor the names of its
  contributors may be used to endorse or promote products derived from
  this software without specific prior written permission.

THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS
"AS IS" AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT
LIMITED TO, THE IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR
A PARTICULAR PURPOSE ARE DISCLAIMED. IN NO EVENT SHALL <COPYRIGHT
HOLDER> BE LIABLE FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL,
EXEMPLARY, OR CONSEQUENTIAL DAMAGES (INCLUDING, BUT NOT LIMITED TO,
PROCUREMENT OF SUBSTITUTE GOODS OR SERVICES; LOSS OF USE, DATA, OR
PROFITS; OR BUSINESS INTERRUPTION) HOWEVER CAUSED AND ON ANY THEORY OF
LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY, OR TORT (INCLUDING
NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE OF THIS
SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.

