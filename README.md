# scala-json-schema

A [Bowtie](https://github.com/bowtie-json-schema/bowtie) test harness.

Its image is published to `ghcr.io/bowtie-json-schema/scala-json-schema` and run via `bowtie run -i scala-json-schema`.

## Local development

To build the image and run it against Bowtie without waiting on CI:

```bash
sbt assembly                      # produces target/scala-3.3.3/bowtieJsonSchema.jar
docker build -t scala-json-schema-local .
uvx --from bowtie-json-schema bowtie smoke -i image:localhost/scala-json-schema-local
```

To check conformance against the official test suite, clone
[json-schema-org/JSON-Schema-Test-Suite](https://github.com/json-schema-org/JSON-Schema-Test-Suite)
(or use a local checkout of [jam01/json-schema](https://github.com/jam01/json-schema), which
bundles it under `shared/src/test/resources/test-suite/`) and point Bowtie at a dialect's
`tests/` directory:

```bash
uvx --from bowtie-json-schema bowtie suite -i image:localhost/scala-json-schema-local \
  path/to/test-suite/tests/draft2020-12 > report.json
uvx --from bowtie-json-schema bowtie summary -s failures -f markdown report.json
```

`tests/draft2020-12/optional/` isn't picked up this way, since Bowtie's `DIALECT` argument needs
the last path segment of the given directory to be a dialect name. Flatten it into its own tree
first, symlinking in `remotes/` so cases that reference remote schemas still resolve:

```bash
mkdir -p /tmp/suite-optional/tests/draft2020-12
cp path/to/test-suite/tests/draft2020-12/optional/*.json /tmp/suite-optional/tests/draft2020-12/
ln -s path/to/test-suite/remotes /tmp/suite-optional/remotes
uvx --from bowtie-json-schema bowtie suite -i image:localhost/scala-json-schema-local \
  /tmp/suite-optional/tests/draft2020-12 > report-optional.json
uvx --from bowtie-json-schema bowtie summary -s failures -f markdown report-optional.json
```
