---
title: "Two Scala Libraries Every Data Engineer Should Know"
date: 2022-07-09T10:01:13-04:00
draft: false
slug: "Two Scala Libraries Every Data Engineer Should Know"
---

As data engineers, we deal with a lot of JSON, which is ubiquitous since JSON is easy for developers to add to applications. However, JSON is not an efficient storage format for applications that frequently query or use the data at scale. Unlike Parquet, JSON is not a splittable file format making it less parallelizable in systems like Spark. Often JSON is not performant enough and requires further ETL to be converted to formats like Parquet which is a splittable file format and therefore parallelizable. In this article, I'm going to demonstrate how two Scala libraries can work together to convert JSON to Parquet without using Spark, [zio-json](https://github.com/zio/zio-json) and [Parquet4s](https://github.com/mjakubowski84/parquet4s). A working knowledge of functional programming in Scala and Zio is helpful, but not required.

For this example, we are going to use JSON data which reflects vehicle ownership and if the vehicle is registered or not. The conversion from JSON to Parquet is a two-step process.
1. Load the JSON into a Scala case class using zio-json.
2. Create the Parquet file from the Scala case class.  

Our data, notice that we have a field that is optional, `isRegistered`.
```[JSON]
{"VIN": "1A123", "make": "foo", "model": "bar", "year": 2002, "owner": "John Doe", "isRegistered": true} 
{"VIN": "1C123", "make": "foo", "model": "barV2", "year": 2022, "owner": "John Doe Jr.", "isRegistered": false}
{"VIN": "1C123", "make": "foo", "model": "barV2", "year": 2022, "owner": "John Doe Jr."}
```

Next, we construct a case class called Vehicle. The optional field, `isRegistered`, is an Option type. In the companion object, we create an implicit decoder.

```[scala]
import zio.json.*

final case class Vehicle(
  VIN: String,
  make: String,
  model: String,
  year: Int,
  owner: String,
  isRegistered: Option[Boolean]
)

object Vehicle {
    implicit val decoder: JsonDecoder[Vehicle] = DeriveJsonDecoder.gen[Vehicle]
}

```
Then we can decode the JSON with the `fromJson[type]` function.
```[scala]
def decodeJson(json: String) =
    ZIO.fromEither(json.fromJson[Vehicle])
```

In order to turn our JSON into Scala case classes it is a matter of passing each JSON string to `decodeJson` function.
```[scala]
import zio.*
import zio.json.*
import zio.Console._

object Main extends zio.ZIOAppDefault:
  val getData =
    ZIO.acquireReleaseWith(ZIO.attemptBlocking(io.Source.fromFile("src/main/resources/vehicles.json")))(file =>
      ZIO.attempt(file.close()).orDie
    )(file => ZIO.attemptBlocking(file.getLines().toList))

  def decodeJson(json: String) =
    ZIO.fromEither(json.fromJson[Vehicle])

  def program =
    for
      vehicleJson         <- getData
      _                   <- ZIO.attempt(vehicleJson.foreach(println))  // Display the raw JSON
      vehicles            <- ZIO.foreach(vehicleJson)(decodeJson)
    yield ()

  def run = program
```

The final step is to take our List of case classes and create a Parquet file from them. We need to do some Hadoop configuration, but you won't need to have the full Hadoop ecosystem installed for this example to work.
```[scala]
import com.github.mjakubowski84.parquet4s.{ParquetWriter, Path}
import org.apache.parquet.hadoop.metadata.CompressionCodecName
import org.apache.hadoop.conf.Configuration

val hadoopConf = new Configuration()
hadoopConf.set("fs.s3a.path.style.access", "true")

val writerOptions = ParquetWriter.Options(
compressionCodecName = CompressionCodecName.SNAPPY,
hadoopConf = hadoopConf
)

def saveAsParquet(vehicles: List[Vehicle]) = ZIO.attemptBlocking(
ParquetWriter
    .of[Vehicle]
    .options(writerOptions)
    .writeAndClose(Path("vehicles.parquet"), vehicles)
  )
 def program =
    for
      vehicleJson         <- getData
      _                   <- ZIO.attempt(vehicleJson.foreach(println))  // Display the raw JSON
      vehicles            <- ZIO.foreach(vehicleJson)(decodeJson)
      _                   <- saveAsParquet(vehicles) // Save to Parquet
    yield ()
```
At this point, we will have successfully transformed JSON to Parquet.  

We can verify our write step was successful by reading back the parquet file that was just created.  We read the Parquet functionally and handle closing file resources gracefully. Then we display the contents in the vehicle case classes.
```[scala]
import com.github.mjakubowski84.parquet4s.ParquetReader

def readParquet(file: Path): Task[List[Vehicle]] =
    ZIO.acquireReleaseWith(ZIO.attemptBlocking(ParquetReader.as[Vehicle].read(file)))(file =>
      ZIO.attempt(file.close()).orDie
    )(file => ZIO.attempt(file.foldLeft(List[Vehicle]())((acc, vehicle) => acc :+ vehicle)))

def program =
    for
      wd                  <- ZIO.attempt(java.lang.System.getProperty("user.dir"))
      vehiclesFromParquet <- readParquet(Path(s"${wd}/vehicles.parquet")) // Read back the data we just saved
      _                   <- ZIO.attempt(vehiclesFromParquet.foreach(println)) // Display the decoded Parquet
    yield ()
```
The output will look like this.
```
Vehicle(1A123,foo,bar,2002,John Doe,Some(true))
Vehicle(1C123,foo,barV2,2022,John Doe Jr.,Some(false))
Vehicle(1C123,foo,barV2,2022,John Doe Jr.,None)
```
Not let us put it all together. So that we can:
1. Read the JSON into Scala case classes.
2. Write the case classes out as a parquet file.
3. Display the contents of the parquet file.
4. Perform some cleanup.
```[scala]
import zio.*
import zio.json.*
import zio.Console._
import java.io.File
import com.github.mjakubowski84.parquet4s.{ParquetReader, ParquetWriter, Path}
import org.apache.parquet.hadoop.metadata.CompressionCodecName
import org.apache.hadoop.conf.Configuration

final case class Vehicle(
  VIN: String,
  make: String,
  model: String,
  year: Int,
  owner: String,
  isRegistered: Option[Boolean]
)

object Vehicle {
    implicit val decoder: JsonDecoder[Vehicle] = DeriveJsonDecoder.gen[Vehicle]
}

object Main extends zio.ZIOAppDefault:
  val getData =
    ZIO.acquireReleaseWith(ZIO.attemptBlocking(io.Source.fromFile("src/main/resources/vehicles.json")))(file =>
      ZIO.attempt(file.close()).orDie
    )(file => ZIO.attemptBlocking(file.getLines().toList))

  def decodeJson(json: String) =
    ZIO.fromEither(json.fromJson[Vehicle])

  val hadoopConf = new Configuration()
  hadoopConf.set("fs.s3a.path.style.access", "true")

  val writerOptions = ParquetWriter.Options(
    compressionCodecName = CompressionCodecName.SNAPPY,
    hadoopConf = hadoopConf
  )

  def saveAsParquet(vehicles: List[Vehicle]) = ZIO.attemptBlocking(
    ParquetWriter
      .of[Vehicle]
      .options(writerOptions)
      .writeAndClose(Path("vehicles.parquet"), vehicles)
  )

  def readParquet(file: Path): Task[List[Vehicle]] =
    ZIO.acquireReleaseWith(ZIO.attemptBlocking(ParquetReader.as[Vehicle].read(file)))(file =>
      ZIO.attempt(file.close()).orDie
    )(file => ZIO.attempt(file.foldLeft(List[Vehicle]())((acc, vehicle) => acc :+ vehicle)))

  val cleanUp =
    for
      wd <- ZIO.attempt(java.lang.System.getProperty("user.dir"))
      _  <- ZIO.attemptBlocking(new File(s"${wd}/vehicles.parquet").delete)
    yield ()

  def program =
    for
      vehicleJson         <- getData
      _                   <- ZIO.attempt(vehicleJson.foreach(println))  // Display the raw JSON
      vehicles            <- ZIO.foreach(vehicleJson)(decodeJson)
      _                   <- saveAsParquet(vehicles) // Save to Parquet
      wd                  <- ZIO.attempt(java.lang.System.getProperty("user.dir"))
      vehiclesFromParquet <- readParquet(Path(s"${wd}/vehicles.parquet")) // Read back the data we just saved
      _                   <- ZIO.attempt(vehiclesFromParquet.foreach(println)) // Display the decoded Parquet
      _                   <- cleanUp
    yield ()

  def run = program
```
In this recipe, we used two libraries zio-json and Parquet4s to easily create Parquet files from JSON. The source code is available [here](https://github.com/kfehlhauer/json2parquet).