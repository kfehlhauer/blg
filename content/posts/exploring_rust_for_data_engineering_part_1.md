---
title: "Exploring Rust For Data Engineering Part 1"
date: 2023-06-20T08:46:35-04:00
draft: false
slug: "Exploring Rust For Data Engineering Part 1"
---
Rust is gaining increasing recognition as the most loved language in the [Stack Overflow developer surveys](https://survey.stackoverflow.co/2022#technology-most-loved-dreaded-and-wanted). As such, it's natural to wonder about its potential within the realm of data engineering. Will data engineers begin to love Rust too, or is it just hype? Love versus adoption are two different things as shown by [The RedMonk Programming Language Rankings: January 2023](https://redmonk.com/sogrady/2023/05/16/language-rankings-1-23/). Over the coming weeks and months, I aim to explore and share my insights into the possible trajectories for Rust within this domain.

Though Rust is undeniably elegant, it diverges from other languages in one significant respect – memory management. Unlike C/C++ or Java/Python Rust does not rely on manual memory allocation or garbage collection. It employs a unique concept called 'memory ownership', which may initially seem bewildering to engineers more familiar with garbage-collected languages like C#, Java, Python, and Scala, where manual memory management is a foreign concept. Those coming from C/C++ may have an easier journey using Rust because they have a headstart with pointers and references. Becoming comfortable with Rust's approach to memory management is an integral part of the learning curve. While many will find the continuous barking of errors by the Rust compiler to be annoying, those error messages will correct many subtle bugs that other languages just ignore. My advice to those keen on exploring Rust is to persevere through these challenges. Embracing Rust will refine your programming skills. Your perseverance will be rewarded with applications that have no buffer overflows, safe concurrency, and predictable latency.

Thus far, JVM languages like Java and Scala have ruled the big data engineering space, underpinning frameworks like Apache Flink and Spark. The JVM dominance in big data systems is a legacy of infrastructure laid down by Java in the Hadoop and MapReduce era. The JDK big data ecosystem is mature and battle-hardened and may have reached its pinnacle. However, as data sizes skyrocket, the downsides of JVM, such as significant memory consumption and garbage collection pauses, become increasingly evident, especially at scale. These issues trigger multi-level repercussions, leading to increased cloud costs, latency, failure rates, and other problems that data engineers grapple with daily. Rust can help you solve those issues.

The foundation for Rust's entry into the data engineering world is already in place. There are available libraries for dealing with Arrow, Parquet, and JSON parsing, high-performing caching libraries like [Mocha](https://github.com/moka-rs/moka), and the under-development [Polars](https://www.pola.rs/) dataframe library that's written in Rust but can be utilized in Python. Asynchronous runtimes like [Tokio](https://tokio.rs) and [Rayon](https://github.com/rayon-rs) enable multi-core CPU usage.

Furthermore, the introduction of [Delta Lake RS](https://github.com/delta-io/delta-rs) provides a pathway to incorporate Rust into existing Deltalakes. Powered by the [Apache Arrow](https://arrow.apache.org/) platform, Delta Lake RS opens opportunities to transfer some workloads away from Spark, although much work is still needed for broader adoption.

Let's get started by transforming JSON to parquet, a common use case. The JDK has strong support for that use case with even stronger support in Scala. See the previous post [Two Scala Libraries Every Data Engineer Should Know](../two-scala-libraries-every-data-engineer-should-know/). Rust has a framework called [Serde](https://docs.rs/serde/latest/serde/) which can serialize and deserialize various data formats to and from Rust structs. Serde will handle the conversion between JSON and structs but not Parquet. For that, we will need to use the [Parquet](https://docs.rs/parquet/41.0.0/parquet/) and [Arrow](https://docs.rs/arrow/41.0.0/arrow/) crates.

Given this JSON:

```[json]
{"VIN": "1A123", "make": "foo", "model": "bar", "year": 2002, "owner": "John Doe", "isRegistered": true} 
{"VIN": "1C123", "make": "foo", "model": "barV2", "year": 2022, "owner": "John Doe Jr.", "isRegistered": false}
{"VIN": "1C123", "make": "foo", "model": "barV2", "year": 2022, "owner": "John Doe Jr."}
```

I want the data stored in the Parquet file that I'm creating to have the following schema:
```
message arrow_schema {
  REQUIRED BYTE_ARRAY VIN (STRING);
  REQUIRED BYTE_ARRAY make (STRING);
  REQUIRED BYTE_ARRAY model (STRING);
  REQUIRED INT32 year (INTEGER(16,false));
  REQUIRED BYTE_ARRAY owner (STRING);
  OPTIONAL BOOLEAN isRegistered;
}
```
First, we start by creating a Rust struct that we can deserialize the JSON into.

```[rust]
use serde::{Deserialize, Serialize};

#[derive(Serialize, Deserialize, Debug)]
#[allow(non_snake_case)]
struct Vechicle {
    VIN: String,
    make: String,
    model: String,
    year: u16,
    owner: String,
    isRegistered: Option<bool>,
}
```
Using the attributes Serialize and Deserialize invoke compile time macros that create the boilerplate code to serialize and deserialize between structs and JSON. Rust uses particular casings that the compiler enforces through warnings. The `VIN` and the `isRegistered` fields are not in snake case, the attribute `allow(non_snake_case)` is used to suppress the warning.

Next, we read a file called `vehicles.json` containing the above JSON. Deserializing the JSON into Rust structs straight forward.
```[rust]
let v: Vechicle = serde_json::from_str(&js)?;
```
Reading in the entire file:
```[rust]
let mut vehicles: Vec<Vechicle> = Vec::new();
if let Ok(lines) = read_lines("vehicles.json") {
    for line in lines {
        if let Ok(js) = line {
            let v: Vechicle = serde_json::from_str(&js)?;
            vehicles.push(v);
        }
    }
}
```

Writing to a Parquet file in Rust currently involves some boilerplate code that is accomplished in three steps. Much of this boilerplate could be abstracted away using Rust macros. Macros are beyond the scope of this post.
1. Create arrays for each column.
```[rust]
use arrow::array::{ArrayRef, StringArray};

let vins = StringArray::from(
        vehicles
            .iter()
            .map(|v| v.VIN.clone())
            .collect::<Vec<String>>(),
    );
```
2. Create a RecordBatch to hold the column arrays. Notice the use of the `Arc`( ‘Arc’ stands for ‘Atomically Reference Counted’) type. It is a thread-safe reference-counting pointer. Those new to Rust will eventually want to obtain a cursory understanding of how threading works in Rust. For now, it's just an implementation detail.
```[rust]
use arrow::record_batch::RecordBatch;

let batch = RecordBatch::try_from_iter(vec![
        ("VIN", Arc::new(vins) as ArrayRef),
        ...
    ])
    .unwrap();
```
3. Use the ```ArrowWriter``` to write the record batch to a file. The compression is also set in this step. 
```[rust]
use parquet::arrow::arrow_writer::ArrowWriter;
use parquet::basic::Compression;

let file = File::create("vehicles.parquet").unwrap();
let props = WriterProperties::builder()
    .set_compression(Compression::SNAPPY)
    .build();

let mut writer = ArrowWriter::try_new(file, batch.schema(), Some(props)).unwrap();
    writer.write(&batch).expect("Unable to write batch");
    writer.close().unwrap();
```

The complete write function.
```[rust]
use arrow::array::{ArrayRef, BooleanArray, StringArray, UInt16Array};
use parquet::basic::Compression;
use parquet::file::properties::WriterProperties;
use std::sync::Arc;

fn write(vehicles: Vec<Vechicle>) -> Result<()> {
    let vins = StringArray::from(
        vehicles
            .iter()
            .map(|v| v.VIN.clone())
            .collect::<Vec<String>>(),
    );

    let makes = StringArray::from(
        vehicles
            .iter()
            .map(|v| v.make.clone())
            .collect::<Vec<String>>(),
    );

    let models = StringArray::from(
        vehicles
            .iter()
            .map(|v| v.model.clone())
            .collect::<Vec<String>>(),
    );

    let years = UInt16Array::from(vehicles.iter().map(|v| v.year).collect::<Vec<u16>>());

    let owners = StringArray::from(
        vehicles
            .iter()
            .map(|v| v.owner.clone())
            .collect::<Vec<String>>(),
    );

    let registrations =
        BooleanArray::from(vehicles.iter().map(|v| v.isRegistered).collect::<Vec<_>>());

    let batch = RecordBatch::try_from_iter(vec![
        ("VIN", Arc::new(vins) as ArrayRef),
        ("make", Arc::new(makes) as ArrayRef),
        ("model", Arc::new(models) as ArrayRef),
        ("year", Arc::new(years) as ArrayRef),
        ("owner", Arc::new(owners) as ArrayRef),
        ("isRegistered", Arc::new(registrations) as ArrayRef),
    ])
    .unwrap();

    let file = File::create("vehicles.parquet").unwrap();
    let props = WriterProperties::builder()
        .set_compression(Compression::SNAPPY)
        .build();

    let mut writer = ArrowWriter::try_new(file, batch.schema(), Some(props)).unwrap();
    writer.write(&batch).expect("Unable to write batch");
    writer.close().unwrap();

    Ok(())
}
```
Reading a Parquet file is much the same but in reverse.
```[rust]
use arrow::array::{ArrayRef, BooleanArray, StringArray, UInt16Array};
use arrow::record_batch::RecordBatch;
use parquet::arrow::arrow_reader::ParquetRecordBatchReaderBuilder;

#[allow(non_snake_case)]
fn read() -> Result<()> {
    let file = File::open("vehicles.parquet").unwrap();
    let arrow_reader = ParquetRecordBatchReaderBuilder::try_new(file).unwrap();
    let record_batch_reader = arrow_reader.build().unwrap();

    let mut vehicles: Vec<Vechicle> = vec![];

    for maybe_batch in record_batch_reader {
        let record_batch = maybe_batch.unwrap();
        let VIN = record_batch
            .column(0)
            .as_any()
            .downcast_ref::<StringArray>()
            .unwrap();
        let make = record_batch
            .column(1)
            .as_any()
            .downcast_ref::<StringArray>()
            .unwrap();
        let model = record_batch
            .column(2)
            .as_any()
            .downcast_ref::<StringArray>()
            .unwrap();
        let year = record_batch
            .column(3)
            .as_any()
            .downcast_ref::<UInt16Array>()
            .unwrap();
        let owner = record_batch
            .column(4)
            .as_any()
            .downcast_ref::<StringArray>()
            .unwrap();
        let isRegistered = record_batch
            .column(5)
            .as_any()
            .downcast_ref::<BooleanArray>();

        for i in 0..record_batch.num_rows() {
            vehicles.push(Vechicle {
                VIN: VIN.value(i).to_string(),
                make: make.value(i).to_string(),
                model: model.value(i).to_string(),
                year: year.value(i),
                owner: owner.value(i).to_string(),
                isRegistered: isRegistered.map(|a| a.value(i)),
            });
        }
    }
```

In conclusion, transforming data from JSON to Parquet is a straightforward process requiring only a few Rust crates. As the Rust ecosystem is further developed I expect data engineering tasks to become more commonplace. In my subsequent posts, I'll delve into the nitty-gritty of using other libraries for various data engineering tasks. Stay tuned for more insights into the exciting possibilities that Rust brings to the table.

The full source code is available [here](https://github.com/kfehlhauer/json_to_parquet).