# bb8-arangodb

ArangoDB support for [bb8] based on the [arangors] crate.
The library supports all authentication methods supported by `arangors`,
defined by `AuthenticationMethod`.

[bb8]: https://crates.io/crates/bb8
[arangors]: https://crates.io/crates/arangors

## Installing

Make sure to add `bb8` and `bb8-arangodb` to your `Cargo.toml`, like:

```toml
[dependencies]
bb8 = "0.8"
bb8-arangodb = "0.1"
arangors = "0.5"
```

## Example

```rust
use bb8::Pool;
use bb8_arangodb::{ArangoConnectionManager, AuthenticationMethod};
use arangors::uclient::reqwest::ReqwestClient;
use futures_util::join_all;

#[tokio::main]
async fn main() {
    let manager = ArangoConnectionManager::<ReqwestClient>::new(
        "http://localhost:8529".to_string(),
        AuthenticationMethod::JWTAuth("root".to_string(), "openSesame".to_string())
    );

    let pool = Pool::builder().max_size(5).build(manager).await.unwrap();

    for _i in 0..10 {
        let pool = pool.clone();

        handles.push(tokio::spawn(async move {
            let conn = pool.get().await.unwrap();
            let db = conn.db("test").await.unwrap();

            let result: Vec<String> = db
                .aql_str("FOR doc IN collection RETURN doc.name")
                .await
                .unwrap();

            println!("{:?}", results);
        }))
    }

    join_all(handles).await;
}
```

## Running tests

To run tests, you'll need ArangoDB running locally. To run tests using docker,
execute the following commands after cloning the repository:

```shell
# Starting ArangoDB in a detached docker container
docker run -d -p 8529:8529 -e ARANGO_ROOT_PASSWORD=openSesame --name arangodb arangodb/arangodb:3.10.0

# Wait some seconds to let ArangoDB start and run tests
cargo test

# Stop and remove the ArangoDB container
docker stop arangodb && docker rm arangodb
```

## Releases

Detailed release notes are available in this repo at [CHANGELOG.md].

[CHANGELOG.md]: CHANGELOG.md

## Reporting issues

Found a bug? We'd love to know about it!

Please report all issues on the GitHub [issue tracker][issues].

[issues]: https://github.com/gabor-boros/bb8-arangodb/issues

## Contributing

See the **[bb8-arangodb Contributor Guide]** for a complete introduction
to contributing to `bb8-arangodb`.

## License

`bb8-arangodb` is primarily distributed under the terms of both the MIT license
and the Apache License (Version 2.0).

See [LICENSE-MIT] and [LICENSE-APACHE] for details.

[LICENSE-MIT]: LICENSE-MIT
[LICENSE-APACHE]: LICENSE-APACHE
