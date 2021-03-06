# sbt-scrooge

Sbt-scrooge is an sbt plugin that adds a mixin for doing thrift code
auto-generation during your compile phase. Just mix in `CompileThriftScrooge`
into your project, and by default, all thrift files matching
`src/thrift/*.thrift` will be used to auto-generate scala sources, written to
target/gen-scala.


## Building

To build, use sbt:

    $ sbt package-dist


## How it works

The plugin injects itself into your compile phase.

It fetches scrooge from the public Twitter maven repository and caches it in
`project/build/target`. (You can override this folder -- see below.) It then
runs scrooge against your thrift folder, re-generating files only if they've
changed. The generated code folder (usually `target/gen-scala`) is then added
to your compile path.


## Combining with sbt-thrift

If you still want to use the apache thrift generator in sbt-thrift (for
example, to generate java or ruby bindings), use the `CompileThriftScroogeMixin`
mixin instead. It leaves paths undefined so they can be defined once, in your
sbt-thrift config.


## Configuration

- `override def scroogeVersion = "1.1.7"`

  to use a different version of scrooge.

- `override def scroogeBuildOptions = List("--finagle", "--ostrich")`

  to change the generated code options. By default, finagle client/server and
  ostrich service stubs are generated.

- `override def scroogeDebug = false`

  to see more debug info (like the scrooge command line)

- `override def scroogeCacheFolder = ("project" / "build" / "target" / scroogeName) ##`

  to change the folder that cached versions of scrooge are stored in.

- `override def thriftSources = (mainSourcePath / "thrift" ##) ** "*.thrift"`

  to change where thrift files are found.

- `override def generatedScalaPath = (outputPath / "gen-scala") ##`

  to change the default destination folder for generated scala files.

- `override def thriftIncludeFolders: Seq[String] = Nil`

  to set some default include paths for thrift.


## Upgrading from sbt-thrift

The scala bindings are not 100% compatible with the scala bindings that were
generated by sbt-thrift. Here are the notable differences:

- You need to add "scrooge-runtime" as a dependency:

    `val scrooge_runtime = "com.twitter" % "scrooge-runtime" % "1.0.3"`

- The names of the interfaces have changed:

  - `(service).ServiceIface` is now `(service).FutureIface`

  - `(service).Service` is now `(service).FinagledService`

  - `(service).Client` is now `(service).FinagledClient`

  - `BinaryThriftSerializer` is now `BinaryThriftStructSerializer[A]`, like:

      `val serializer = new BinaryThriftStructSerializer[Beer] { def codec = Beer }`


## Upgrading from java

If you're switching from java generated code, there are a few other (fairly
large) differences:

- All container types are scala-native (Seq, Set, Map).

- Enums are case objects (but still subclass TEnum).

- Class, struct, and enum names are all converted to StudyCaps, so for
  example `ERROR` becomes `Error`.

- Method names are all converted to camelCase, so for example `GetName`
  becomes `getName` and `set_name` becomes `setName`.

- Structs are case classes, so they are immutable and have real, actual
  constructors (!). Multi-line field-setting struct construction should be
  converted to a single-line case class constuctor call.

- Optional fields are implemented as `Option[A]`.
