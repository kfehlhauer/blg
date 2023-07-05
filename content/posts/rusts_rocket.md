---
title: "Rust's Rocket"
date: 2023-07-05T07:03:52-04:00
draft: false
slug: "Rust's Rocket"
---

The [Rocket](https://rocket.rs) [crate](https://crates.io/crates/rocket) provides a lot of web server functionality that is simple to use. It compares well to other web-server backend libraries like [Flask](https://flask.palletsprojects.com) in Python. Currently, no Rust framework even registers on the [Hacker Rank](https://survey.stackoverflow.co/2023/?mod=djemCIO#section-most-popular-technologies-web-frameworks-and-technologies) surveys of favorite web frameworks. That will change. Rocket is easy to use with great documentation and has over 21k stars on GitHub. This is a brief but detailed introduction to using Rocket, by the end of this post you should have a basic understanding of how to get, post, and put using Rocket. 

To get started with Rocket you need to add it to your `Cargo.toml` file.
```[toml]
[package]
name = "my-first-rocket"
version = "0.1.0"
edition = "2021"

[dependencies]
rocket = "0.5.0-rc.3"
```

Adding methods and routes to your `main.rs` file.
```[rust]
#[macro_use]
extern crate rocket;

#[get("/")]
fn index() -> &'static str {
    "Hello, world!"
}

#[launch]
fn rocket() -> _ {
    rocket::build()
        .mount("/", routes![index])
}
```
Then run `cargo run` and enter `http://127.0.0.1:8000/` into your browser. You should see "Hello, world!". 


For comparison, the Python example coded in Flask is comparable to the Rust version coded in Rocket.
```[python]
from flask import Flask
app = Flask(__name__)

@app.route("/")
    def index():
        return "Hello World!"

if __name__ == "__main__":
    app.run()
```
This is about the same amount of code as needed in Python's Flask. With some extra code going towards specifying types and brackets. For a small investment in additional syntax, you gain all the benefits of type safety, no garbage collection, and not having to install Python's runtimes to deploy this code. Creating and deploying the Rocket version is as simple as running `cargo build --release` and then copying the binary to the deployment location.

The `get` was pretty easy. Next, I will demonstrate the `post` and `put` to get a better understanding of what it is like to code using Rocket. This is where life gets a little more interesting. One of the selling points of Rust is ["fearless concurrency"](https://doc.rust-lang.org/book/ch16-00-concurrency.html) which is not even a thing in frameworks like Flask. Flask also has different goals than Rocket, it doesn't pretend to be highly concurrent. Rocket like Rust wants to make use of all your CPU cores. That comes with the cost of learning how to write concurrent code in Rust.

In this demo, I will be replicating a database in a hashmap data structure using `HashMap<u32, String>`. However, since we need to make this thread safe it will need to be wrapped in a mutex and held in a Rocket State like this.

```[rust]
use rocket::tokio::sync::Mutex;
use rocket::State;

type DataHashMap = Mutex<HashMap<u32, String>>;
type Database<'r> = &'r State<DataHashMap>;
```

The State reference requires a [lifetime](https://doc.rust-lang.org/book/ch10-03-lifetime-syntax.html) which is what `<'r>` syntax represents.  

So far we have only defined the types that make up the database. We have not instantiated it yet. Later, when we wire up the routes we will instantiate the hashmap and add it as a managed resource. This will make it available to functions using Rocket's annotations.
```[rust]
#[launch]
fn rocket() -> _ {
    rocket::build()
        .mount("/", routes![index])
        .manage(DataHashMap::new(HashMap::new()))
}
```

JSON payloads tend to be ubiquitous on the Web whether it is for REST payloads or other purposes. In this post, we will examine two ways to represent deserialized JSON within a struct.
First, the obvious approach. It has the benefit of being easy to understand at the cost of constantly having to own the string when you need to modify the value. The entire point of using Rust is to be efficient, so this is not the most idiomatic approach.

```[rust]
use rocket::serde::{Deserialize, Serialize};

#[derive(Serialize, Deserialize)]
#[serde(crate = "rocket::serde")]
struct Data {
    id: u32,
    value: String,
}
```
The more idiomatic approach is to use the Cow smart pointer since you don't have to think about if the value is just borrowed or owned. However, you will need to consider lifetimes in functions that use it. A small penalty to pay for the performance gain. However, depending on your use case. If your endpoint is under a light load, the additional memory spent may be worth it, but then you probably don't need Rust or Rocket either. 
```[rust]
use rocket::serde::{Deserialize, Serialize};
use std::borrow::Cow;

#[derive(Serialize, Deserialize)]
#[serde(crate = "rocket::serde")]
struct Data<'r> {
    id: u32,
    value: Cow<'r, str>,
}
```

First, I will implement the post, put, and get using non-reference versions of the struct.

```[rust]
#[post("/insert", format = "json", data = "<data>")]
async fn new(data: Json<Data>, database: Database<'_>) -> (Status, Value) {
    let mut db = database.lock().await;
    if db.contains_key(&data.id) {
        return (
            Status::BadRequest,
            json!({ "status": "failed - record already exists" }),
        );
    }

    db.insert(data.id, data.value.to_string());
    (Status::Accepted, json!({ "status": "ok" }))
}

#[launch]
fn rocket() -> _ {
    rocket::build()
        .mount("/", routes![new]) // Added the route for the "new" function
        .manage(DataHashMap::new(HashMap::new()))
}
``` 
The function `new` is annotated with the relative path, the format of the payload, JSON in this case and the variable that will hold the payload.
```[rust]
#[post("/insert", format = "json", data = "<data>")]
```

Since Rocket is designed to be concurrent we are going to use `async` functions. The function receives the JSON payload which is deserialized by Serde which is what `data: Json<Data>` does. The final argument passes in the database which is nothing more than a hashmap wrapped in a mutex that has its state managed by Rocket as declared earlier. The code `database: Database<'_>` may look odd, this is where we have to use lifetime notation to assist the Rust borrow checker because the type Database is a reference type. This ensures that the reference will remain valid through the life of the function call. The `<'_>` represents an anonymous lifetime. When you see <'_> in a type annotation or a function signature, it means that the reference has a specific, but unnamed, lifetime. The compiler will infer the actual lifetime based on the context where the reference is used. The function then returns a tuple of its status and a detailed message in a returned JSON payload.
```[rust]
async fn new(data: Json<Data>, database: Database<'_>) -> (Status, Value)
``` 
Since our database is wrapped in a mutex we must obtain a lock to it, so the state does not change while we attempt to retrieve a value for a given key in the hashmap. The code also checks for a unique key and does not allow duplicate entries for a given id.
```[rust]
let mut db = database.lock().await; // Get lock on the hashmap
    if db.contains_key(&data.id) {  // Check for duplicate id!
        return (                    // If a duplicate return with approriate status message
            Status::BadRequest,
            json!({ "status": "failed - record already exists" }),
        );
    }
```
Once we are sure we don't have a duplicate entry in our database we do the insert and return success results to the client.
```[rust]
db.insert(data.id, data.value.to_string());
    (Status::Accepted, json!({ "status": "ok" }))
```
At this point, the complete working example.
```[rust]
use rocket::http::Status;
use rocket::serde::json::{json, Json, Value};
use rocket::serde::{Deserialize, Serialize};
use rocket::tokio::sync::Mutex;
use rocket::State;
use std::collections::HashMap;

#[macro_use]
extern crate rocket;

type DataHashMap = Mutex<HashMap<u32, String>>;
type Database<'r> = &'r State<DataHashMap>;

#[derive(Serialize, Deserialize)]
#[serde(crate = "rocket::serde")]
struct Data {
    id: u32,
    value: String,
}

#[post("/insert", format = "json", data = "<data>")]
async fn new(data: Json<Data>, database: Database<'_>) -> (Status, Value) {
    let mut db = database.lock().await;
    if db.contains_key(&data.id) {
        return (
            Status::BadRequest,
            json!({ "status": "failed - record already exists" }),
        );
    }

    db.insert(data.id, data.value.to_string());
    (Status::Accepted, json!({ "status": "ok" }))
}

#[launch]
fn rocket() -> _ {
    rocket::build()
        .mount("/", routes![new])
        .manage(DataHashMap::new(HashMap::new()))
}
```

To retrieve the data stored we need to implement the `get` function and add its route to the `mount`. Also, notice the use of pattern matching which makes the code more compact and easier to comprehend.
```[rust]
#[get("/getdata/<id>")]
async fn get_my_data(id: u32, database: Database<'_>) -> (Status, Value) {
    match database.lock().await.get(&id) {
        Some(d) => (Status::Accepted, json!({ "status": "ok", "data": d })),
        None => (
            Status::BadRequest,
            json!({ "status": "failed - no data found"}),
        ),
    }
}

#[launch]
fn rocket() -> _ {
    rocket::build()
        .mount("/", routes![get_my_data, new])
        .manage(DataHashMap::new(HashMap::new()))
}
```
Now we can insert data and return data. Using the VS Code plugin [Rest Client](https://marketplace.visualstudio.com/items?itemName=humao.rest-client)
```
POST http://127.0.0.1:8000/insert HTTP/1.1
content-type: application/json

{
    "id": 1,
    "value": "NEW DATA"
}
```
To get the data back, in a browser enter `http://127.0.0.1:8000/getdata/1` or use the Rest Client.
```
GET http://127.0.0.1:8000/getdata/1 HTTP/1.1
```
The results are:
```
HTTP/1.1 202 Accepted
content-type: application/json
server: Rocket
x-content-type-options: nosniff
x-frame-options: SAMEORIGIN
permissions-policy: interest-cohort=()
content-length: 15
date: Tue, 04 Jul 2023 12:58:05 GMT

{
  "status": "ok"
}
```
The update function using a `put` will be more of the same.
```[rust]
#[put("/update", format = "json", data = "<data>")]
async fn update(data: Json<Data>, database: Database<'_>) -> (Status, Value) {
    match database.lock().await.get_mut(&data.id) {
        Some(d) => {
            *d = data.value.to_string();
            (Status::Accepted, json!({ "status": "ok" }))
        }
        None => {
            let status = format!(
                "failed - record does not exist for {}",
                &data.id.to_string()
            );
            (Status::BadRequest, json!({ "status": status }))
        }
    }
}

#[launch]
fn rocket() -> _ {
    rocket::build()
        .mount("/", routes![get_my_data, new, update])
        .manage(DataHashMap::new(HashMap::new()))
}
```
The complete example.
```[rust]
use rocket::http::Status;
use rocket::serde::json::{json, Json, Value};
use rocket::serde::{Deserialize, Serialize};
use rocket::tokio::sync::Mutex;
use rocket::State;
use std::collections::HashMap;

#[macro_use]
extern crate rocket;

type DataHashMap = Mutex<HashMap<u32, String>>;
type Database<'r> = &'r State<DataHashMap>;

#[derive(Serialize, Deserialize)]
#[serde(crate = "rocket::serde")]
struct Data {
    id: u32,
    value: String,
}

#[post("/insert", format = "json", data = "<data>")]
async fn new(data: Json<Data>, database: Database<'_>) -> (Status, Value) {
    let mut db = database.lock().await;
    if db.contains_key(&data.id) {
        return (
            Status::BadRequest,
            json!({ "status": "failed - record already exists" }),
        );
    }

    db.insert(data.id, data.value.to_string());
    (Status::Accepted, json!({ "status": "ok" }))
}

#[put("/update", format = "json", data = "<data>")]
async fn update(data: Json<Data>, database: Database<'_>) -> (Status, Value) {
    match database.lock().await.get_mut(&data.id) {
        Some(d) => {
            *d = data.value.to_string();
            (Status::Accepted, json!({ "status": "ok" }))
        }
        None => {
            let status = format!(
                "failed - record does not exist for {}",
                &data.id.to_string()
            );
            (Status::BadRequest, json!({ "status": status }))
        }
    }
}

#[get("/getdata/<id>")]
async fn get_my_data(id: u32, database: Database<'_>) -> (Status, Value) {
    match database.lock().await.get(&id) {
        Some(d) => (Status::Accepted, json!({ "status": "ok", "data": d })),
        None => (
            Status::BadRequest,
            json!({ "status": "failed - no data found"}),
        ),
    }
}

#[launch]
fn rocket() -> _ {
    rocket::build()
        .mount("/", routes![get_my_data, new, update])
        .manage(DataHashMap::new(HashMap::new()))
}
```

Next, let's refactor this code to make it more idiomatic Rust. We will replace:
```[rust]
#[derive(Serialize, Deserialize)]
#[serde(crate = "rocket::serde")]
struct Data {
    id: u32,
    value: String,
}
```
with
```[rust]
use rocket::serde::{Deserialize, Serialize};
use std::borrow::Cow;

#[derive(Serialize, Deserialize)]
#[serde(crate = "rocket::serde")]
struct Data<'r> {
    id: u32,
    value: Cow<'r, str>,
}
```
The `post` will also be refactored to use pattern matching. 
```[rust]
use rocket::http::Status;
use rocket::serde::json::{json, Json, Value};
use rocket::serde::{Deserialize, Serialize};
use rocket::tokio::sync::Mutex;
use rocket::State;
use std::borrow::Cow;
use std::collections::HashMap;

#[macro_use]
extern crate rocket;

type DataHashMap = Mutex<HashMap<u32, String>>;
type Database<'r> = &'r State<DataHashMap>;

#[derive(Serialize, Deserialize)]
#[serde(crate = "rocket::serde")]
struct Data<'r> {
    id: u32,
    value: Cow<'r, str>,
}

#[post("/insert", format = "json", data = "<data>")]
async fn new(data: Json<Data<'_>>, database: Database<'_>) -> (Status, Value) {
    let mut db = database.lock().await;

    match db.contains_key(&data.id) {
        false => {
            db.insert(data.id, data.value.to_string());
            (Status::Accepted, json!({ "status": "ok" }))
        }
        true => (
            Status::BadRequest,
            json!({ "status": "failed - record already exists" }),
        ),
    }
}

#[put("/update", format = "json", data = "<data>")]
async fn update(data: Json<Data<'_>>, database: Database<'_>) -> (Status, Value) {
    match database.lock().await.get_mut(&data.id) {
        Some(d) => {
            *d = data.value.to_string();
            (Status::Accepted, json!({ "status": "ok" }))
        }
        None => {
            let status = format!(
                "failed - record does not exist for {}",
                &data.id.to_string()
            );
            (Status::BadRequest, json!({ "status": status }))
        }
    }
}

#[get("/getdata/<id>")]
async fn get_my_data(id: u32, database: Database<'_>) -> (Status, Value) {
    match database.lock().await.get(&id) {
        Some(d) => (Status::Accepted, json!({ "status": "ok", "data": d })),
        None => (
            Status::BadRequest,
            json!({ "status": "failed - no data found"}),
        ),
    }
}

#[launch]
fn rocket() -> _ {
    rocket::build()
        .mount("/", routes![get_my_data, new, update])
        .manage(DataHashMap::new(HashMap::new()))
}
```
In the final refactored version we now take advantage of smart pointers, however, now we need to deal with more lifetimes.
```[rust]
async fn new(data: Json<Data<'_>>, database: Database<'_>) -> (Status, Value) // data: Json<Data<'_>> has lifetime notation
async fn update(data: Json<Data<'_>>, database: Database<'_>) -> (Status, Value) // data: Json<Data<'_>> has lifetime notation
```

### Conclusion
Rocket is simple and it shares a lot in common with other libraries for different languages. Becoming productive in Rust is possible once you overcome the learning curve of the borrow checker and learn some concurrent programming skills. If you already have those skills, using Rocket is just as simple as using frameworks in more dynamic languages. Also, does not take six months to become proficient in Rust, you can become productive in [two months](https://opensource.googleblog.com/2023/06/rust-fact-vs-fiction-5-insights-from-googles-rust-journey-2022.html).

Rust's ecosystem for web development is growing. It doesn't have complete batteries-included (opinionated) frameworks like Django but that also mirrors the Rust standard runtime which is not prescriptive and encourages the use of the crates that meet your needs. Rocket is one of many frameworks under active development such as [Actix Web](https://actix.rs/) and Axum(https://github.com/tokio-rs/axum). The ["Are we web yet?"](https://www.arewewebyet.org/) is a great resource for learning about the state of web development in Rust.

### Other thoughts
While not readily apparent from this intro to Rocket, managing memory as efficiently as possible will have an impact on performance and cost. Does it matter? It depends on your company's goals and what its values are. Most companies don't care about writing efficient code for the sake of writing efficient code. Most companies value developer productivity more than server costs which is why garbage-collected languages like Python, Go, and Java remain popular. For that class of company, sticking with Python, Javascript, etc. will serve them well. On the other side of the spectrum are a class of companies where scaling out has significant impacts on costs or in some cases all out performance matters to them. Making use of all your cores in Rust is a much lower-risk activity compared to C/C++. A real-world example of this was Cloudflare's rewriting of an [NGINX module in Rust](https://blog.cloudflare.com/rust-nginx-module/).

[The source code can be found here. ](https://github.com/kfehlhauer/my-first-rocket/tree/main)     