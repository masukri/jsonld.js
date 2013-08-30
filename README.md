jsonld.js
=========

[![Build Status][travis-ci-png]][travis-ci-site]
[travis-ci-png]: https://travis-ci.org/digitalbazaar/jsonld.js.png?branch=master
[travis-ci-site]: https://travis-ci.org/digitalbazaar/jsonld.js

Introduction
------------

JSON, as specified in RFC4627, is a simple language for representing
objects on the Web. Linked Data is a way of describing content across
different documents or Web sites. Web resources are described using
IRIs, and typically are dereferencable entities that may be used to find
more information, creating a "Web of Knowledge". JSON-LD is intended to
be a simple publishing method for expressing not only Linked Data in
JSON, but for adding semantics to existing JSON.

This library is an implementation of the [JSON-LD] specification
in JavaScript.

JSON-LD is designed as a light-weight syntax that can be used to express
Linked Data. It is primarily intended to be a way to express Linked Data
in Javascript and other Web-based programming environments. It is also
useful when building interoperable Web Services and when storing Linked
Data in JSON-based document storage engines. It is practical and
designed to be as simple as possible, utilizing the large number of JSON
parsers and existing code that is in use today. It is designed to be
able to express key-value pairs, RDF data, RDFa [RDFA-CORE] data,
Microformats [MICROFORMATS] data, and Microdata [MICRODATA]. That is, it
supports every major Web-based structured data model in use today.

The syntax does not require many applications to change their JSON, but
easily add meaning by adding context in a way that is either in-band or
out-of-band. The syntax is designed to not disturb already deployed
systems running on JSON, but provide a smooth migration path from JSON
to JSON with added semantics. Finally, the format is intended to be fast
to parse, fast to generate, stream-based and document-based processing
compatible, and require a very small memory footprint in order to operate.

## Quick Examples

```js
var doc = {
  "http://schema.org/name": "Manu Sporny",
  "http://schema.org/url": {"@id": "http://manu.sporny.org/"},
  "http://schema.org/image": {"@id": "http://manu.sporny.org/images/manu.png"}
};
var context = {
  "name": "http://schema.org/name",
  "homepage": {"@id": "http://schema.org/url", "@type": "@id"},
  "image": {"@id": "http://schema.org/image", "@type": "@id"}
};

// compact a document according to a particular context
// see: http://json-ld.org/spec/latest/json-ld/#compacted-document-form
jsonld.compact(doc, context, function(err, compacted) {
  console.log(JSON.stringify(compacted, null, 2));
  /* Output:
  {
    "@context": {...},
    "name": "Manu Sporny",
    "homepage": "http://manu.sporny.org/",
    "image": "http://manu.sporny.org/images/manu.png"
  }
  */
});

// compact using URLs
jsonld.compact('http://example.org/doc', 'http://example.org/context', ...);

// expand a document, removing its context
// see: http://json-ld.org/spec/latest/json-ld/#expanded-document-form
jsonld.expand(compacted, function(err, expanded) {
  /* Output:
  {
    "http://schema.org/name": [{"@value": "Manu Sporny"}],
    "http://schema.org/url": [{"@id": "http://manu.sporny.org/"}],
    "http://schema.org/image": [{"@id": "http://manu.sporny.org/images/manu.png"}]
  }
  */
});

// expand using URLs
jsonld.compact('http://example.org/doc', ...);

// flatten a document
// see: http://json-ld.org/spec/latest/json-ld/#flattened-document-form
jsonld.flatten(doc, function(err, flattened) {
  // all deep-level trees flattened to the top-level
});

// frame a document
// see: http://json-ld.org/spec/latest/json-ld-framing/#introduction
jsonld.frame(doc, frame, function(err, framed) {
  // document transformed into a particular tree structure per the given frame
});

// normalize a document
jsonld.normalize(doc, {format: 'application/nquads'}, function(err, normalized) {
  // normalized is a string that is a canonical representation of the document
  // that can be used for hashing
});

// use the promises API
var promises = jsonld.promises();

// compaction
var promise = promises.compact(doc, context);
promise.then(function(compacted) {...}, function(err) {...});

// expansion
var promise = promises.expand(doc);
promise.then(function(expanded) {...}, function(err) {...});

// flattening
var promise = promises.flatten(doc);
promise.then(function(flattened) {...}, function(err) {...});

// framing
var promise = promises.frame(doc, frame);
promise.then(function(framed) {...}, function(err) {...});

// normalization
var promise = promises.normalize(doc, {format: 'application/nquads'});
promise.then(function(normalized) {...}, function(err) {...});
```

Using the Command-line Tool
---------------------------

The jsonld command line tool can be used to:

 * Transform JSON-LD to compact, expanded, normalized, or flattened form
 * Transform RDFa to JSON-LD
 * Normalize JSON-LD/RDFa Datasets to NQuads

To install the tool, do the following (you will need git, nodejs, and
npm installed):

    git clone https://github.com/digitalbazaar/jsonld.js.git
    cd jsonld.js
    npm install

To compact a document on the Web using a JSON-LD context published on
the Web:

    ./bin/jsonld compact -c "https://w3id.org/payswarm/v1" "http://recipes.payswarm.com/?p=10554"

The command above will read in a PaySwarm Asset and Listing in RDFa 1.0 format,
convert it to JSON-LD expanded form, compact it using the
'https://w3id.org/payswarm/v1' context, and dump it out to the console in
compacted form.

    ./bin/jsonld normalize -q "http://recipes.payswarm.com/?p=10554"

The command above will read in a PaySwarm Asset and Listing in RDFa 1.0 format,
normalize the data using the RDF Dataset normalization algorithm, and
then dump the output to normalized NQuads format. The NQuads can then be
processed via SHA-256, or similar algorithm, to get a deterministic hash
of the contents of the Dataset.

Commercial Support
------------------

Commercial support for this library is available upon request from
Digital Bazaar: support@digitalbazaar.com

Source
------

The source code for the JavaScript implementation of the JSON-LD API
is available at:

http://github.com/digitalbazaar/jsonld.js

Tests
-----

This library includes a sample testing utility which may be used to verify
that changes to the processor maintain the correct output.

To run the sample tests you will need to get the test suite files by cloning
the [json-ld.org repository][json-ld.org] hosted on GitHub.

https://github.com/json-ld/json-ld.org

If the json-ld.org directory is a sibling of the jsonld.js directory:

    make test

If you installed the test suite elsewhere:

    make test JSONLD_TEST_SUITE={PATH_TO_YOUR_JSON_LD_ORG}/test-suite}

Code coverage output can be generated in `coverage.html`:

    make test-cov

The Mocha output reporter can be changed to min, dot, list, nyan, etc:

    make test REPORTER=dot

Remote context tests are also available:

    # run the context server in the background or another terminal
    node tests/remote-context-server.js

    make test JSONLD_TEST_SUITE=./tests

To generate earl reports:

    # generate the earl report for node.js
    ./node_modules/.bin/mocha -R list tests/test.js --earl earl-node.jsonld

    # generate the earl report for the browser
    ./node_modules/.bin/phantomjs tests/test.js --timeout 120 --earl earl-browser.jsonld


[JSON-LD]: http://json-ld.org/
[json-ld.org]: https://github.com/json-ld/json-ld.org

