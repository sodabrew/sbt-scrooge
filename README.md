# sbt11-scrooge

Sbt11-scrooge is an sbt 0.11 plugin that adds support for using scrooge to
perform thrift code generation.

## How it works

The plugin registers itself as a source generator for the compile phase.

It fetches scrooge from the public Twitter maven repository, caches it in your
project, and runs it against your thrift folder (usually `src/main/thrift`).
The generated code folder (usually `target/src_managed`) is then added to your
compile path.

## Using it

See [the sbt page on plugins](https://github.com/harrah/xsbt/wiki/Plugins) for
information on adding plugins. Usually, you need to add the following to your
`project/plugins.sbt` file:

    addSbtPlugin("com.twitter" % "sbt11-scrooge" % "1.0.0")

(But obviously, use the latest version number.)

If you use a `build.sbt` file, add this incantation:

    import com.twitter.sbt._

    seq(SbtScrooge.newSettings: _*)

If you use `Build.scala`, add `SbtScrooge.newSettings` to your settings list.

## Configuration

A full list of settings is in the (only) source file. Here are the ones you're
most likely to want to edit:

- `scroogeVersion: String`

  to use a different version of scrooge than the current default

- `scroogeCacheFolder: File`

  to unpack the downloaded scrooge release into a different folder

- `scroogeBuildOptions: Seq[String]`

  list of command-line arguments to pass to scrooge; by default this is
  usually `Seq("--finagle", "--ostrich", "--verbose")`

- `scroogeThriftIncludeFolders: Seq[File]`

  list of folders to search when processing "include" directives

- `scroogeThriftSourceFolder: File`

  where to find thrift files to compile

- `scroogeThriftOutputFolder: File`

  where to put the generated scala files


# Notes for helping work on sbt11-scrooge

## Building

To build the plugin locally and publish it to your local filesystem:

    $ sbt publish-local

## Testing

There is a really crude scripted plugin. You can run it with:

    $ sbt scripted

It currently has the version number hard-coded, so you may need to update that
manually. (Note: On my mac, this just consumes all memory and dies. Does this
work for anyone? -robey)


# Notes for people upgrading from very old versions
    
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
