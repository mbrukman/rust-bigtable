[![](https://travis-ci.org/durch/rust-s3.svg?branch=master)](https://travis-ci.org/durch/rust-google-bigtable)
[![](http://meritbadge.herokuapp.com/rust-bigtable)](https://crates.io/crates/rust-bigtable)
[![MIT licensed](https://img.shields.io/badge/license-MIT-blue.svg)](https://github.com/durch/rust-s3/blob/master/LICENSE.md)
[![Join the chat at https://gitter.im/durch/rust-bigtable](https://badges.gitter.im/durch/rust-bigtable)](https://gitter.im/durch/rust-bigtable?utm_source=badge&utm_medium=badge&utm_campaign=pr-badge&utm_content=badge)
## rust-bigtable [[docs](https://durch.github.io/rust-bigtable/)]

Rust library for working with [Google Bigtable](https://cloud.google.com/bigtable/docs/) [Data API](https://cloud.google.com/bigtable/docs/reference/data/rpc/google.bigtable.v2)

*`requires nightly Rust`*

### Intro
Interface towards Cloud Bigtable, supports all [Data API](https://cloud.google.com/bigtable/docs/reference/data/rpc/google.bigtable.v2) methods. 

+ CheckAndMutateRow
+ MutateRow
+ MutateRows
+ ReadModifyWriteRow
+ ReadRows
+ SampleRowKeys

Includes support for [JWT auth](https://cloud.google.com/docs/authentication):

### How it works

Initial plans was to go full `grpc` over `http/2`, unfortunately Rust support is not there yet, so a middle way was taken :).

Requests objects are `protobuf` messages, generated using `proto` definitions available from [Google](https://github.com/googleapis/googleapis/blob/master/google/bigtable/v2/bigtable.proto). And all configuration is done through very nice interfaces generated in this way. These messages are than transparently converted to `json`, and sent to predefined `google.api.http` endpoints, also defined [here](https://github.com/googleapis/googleapis/blob/master/google/bigtable/v2/bigtable.proto). Responses are returned as [serde_json::Value](https://docs.serde.rs/serde_json/value/index.html).

In theory this should enable easy upgrade to full `grpc` over `http/2` as soon as it becomes viable, the only remaining work would be utilising proper return types, also available as `protobuf` messages.

### Configuration

You'll need to provide you private key in `pem` (see `random_rsa_for_testing` for proper format) format as well as Google Cloud service account with proper scopes (scopes are handled by [goauth](https://crates.io/crates/goauth), as part of authentication), 

### Usage

*In your Cargo.toml*

```
[dependencies]
bigtable = '0.1.3'
```

#### Examples

##### CheckAndMutateRow

```
# #![allow(unused_imports)]
extern crate bigtable as bt;
extern crate serde_json;
extern crate protobuf;

use protobuf::RepeatedField;

use bt::request::BTRequest;
use bt::utils::*;
use bt::method::{BigTable, CheckAndMutateRow};
use bt::data::{RowFilter, Mutation_DeleteFromRow, Mutation};
use bt::bigtable::MutateRowsRequest_Entry;
# use bt::error::BTErr;

const TOKEN_URL: &'static str = "https://www.googleapis.com/oauth2/v4/token";
const ISS: &'static str = "service_acc@developer.gserviceaccount.com";
const PK: &'static str = "random_rsa_for_tests";

fn main() {
# #[allow(dead_code)]
# fn wrapper() -> Result<(), BTErr> {
    let mut req = BTRequest {
                          base: None,
                          table: Default::default(),
                          method: CheckAndMutateRow::new()
                    };

    let row_key = row_key_from_str("r1");
        
    let mut predicate_filter = RowFilter::new();
    predicate_filter.set_pass_all_filter(true);
        
    let mut mutations: Vec<Mutation> = Vec::new();
    let mut delete_row_mutation = Mutation::new();
    delete_row_mutation.set_delete_from_row(Mutation_DeleteFromRow::new());
    mutations.push(delete_row_mutation);
        
    req.method.payload_mut().set_row_key(row_key);
    req.method.payload_mut().set_predicate_filter(predicate_filter);
    req.method.payload_mut().set_true_mutations(RepeatedField::from_vec(mutations));

    let response = req.execute(&get_auth_token(TOKEN_URL, ISS, PK)?)?;
    println!("{}", serde_json::to_string_pretty(&response)?);
# Ok(())
# }
}
```

##### MutateRow

```
# #![allow(unused_imports)]
extern crate bigtable as bt;
extern crate serde_json;
extern crate protobuf;

use protobuf::RepeatedField;

use bt::request::BTRequest;
use bt::utils::*;
use bt::method::{BigTable, MutateRow};
use bt::data::{Mutation, Mutation_DeleteFromRow};
# use bt::error::BTErr;

const TOKEN_URL: &'static str = "https://www.googleapis.com/oauth2/v4/token";
const ISS: &'static str = "service_acc@developer.gserviceaccount.com";
const PK: &'static str = "random_rsa_for_tests";

fn main() {
# #[allow(dead_code)]
# fn wrapper() -> Result<(), BTErr> {
    let mut req = BTRequest {
                      base: None,
                      table: Default::default(),
                      method: MutateRow::new()
                };
    
    let row_key = row_key_from_str("r1");
    
    let mut mutations: Vec<Mutation> = Vec::new();
    let mut delete_row_mutation = Mutation::new();
    delete_row_mutation.set_delete_from_row(Mutation_DeleteFromRow::new());
    mutations.push(delete_row_mutation);
    
    req.method.payload_mut().set_row_key(row_key);
    req.method.payload_mut().set_mutations(RepeatedField::from_vec(mutations));

    let response = req.execute(&get_auth_token(TOKEN_URL, ISS, PK)?)?;
    println!("{}", serde_json::to_string_pretty(&response)?);
# Ok(())
# }
}
```

##### MutateRows

```
# #![allow(unused_imports)]
extern crate bigtable as bt;
extern crate serde_json;
extern crate protobuf;

use protobuf::RepeatedField;

use bt::request::BTRequest;
use bt::utils::*;
use bt::method::{BigTable, MutateRows};
use bt::data::{Mutation, Mutation_DeleteFromRow};
use bt::bigtable::MutateRowsRequest_Entry;
# use bt::error::BTErr;

const TOKEN_URL: &'static str = "https://www.googleapis.com/oauth2/v4/token";
const ISS: &'static str = "service_acc@developer.gserviceaccount.com";
const PK: &'static str = "random_rsa_for_tests";

fn main() {
# #[allow(dead_code)]
# fn wrapper() -> Result<(), BTErr> {
    let mut req = BTRequest {
                      base: None,
                      table: Default::default(),
                      method: MutateRows::new()
                };
    
    let mut mutate_entries = Vec::new();
    let mut mutate_entry = MutateRowsRequest_Entry::new();
    
    let row_key = row_key_from_str("r1");
    
    let mut mutations: Vec<Mutation> = Vec::new();
    let mut delete_row_mutation = Mutation::new();
    delete_row_mutation.set_delete_from_row(Mutation_DeleteFromRow::new());
    
    mutations.push(delete_row_mutation);
    mutate_entry.set_mutations(RepeatedField::from_vec(mutations));
    mutate_entry.set_row_key(row_key);
    mutate_entries.push(mutate_entry);

    req.method.payload_mut().set_entries(RepeatedField::from_vec(mutate_entries));

    let response = req.execute(&get_auth_token(TOKEN_URL, ISS, PK)?)?;
    println!("{}", serde_json::to_string_pretty(&response)?);
# Ok(())
# }
}
```

##### ReadWriteModifyRow

```
# #![allow(unused_imports)]
extern crate protobuf;
extern crate bigtable as bt;
extern crate serde_json;

use protobuf::RepeatedField;

use bt::request::BTRequest;
use bt::data::ReadModifyWriteRule;
use bt::utils::*;
use bt::method::{BigTable, ReadModifyWriteRow};
# use bt::error::BTErr;

const TOKEN_URL: &'static str = "https://www.googleapis.com/oauth2/v4/token";
const ISS: &'static str = "service_acc@developer.gserviceaccount.com";
const PK: &'static str = "random_rsa_for_tests";

fn main() {
# #[allow(dead_code)]
# fn wrapper() -> Result<(), BTErr> {
    let mut req = BTRequest {
                          base: None,
                          table: Default::default(),
                          method: ReadModifyWriteRow::new()
               };
    
    let token = get_auth_token(TOKEN_URL, ISS, PK)?;
    
    let mut rules: Vec<ReadModifyWriteRule> = Vec::new();
    let mut rule = ReadModifyWriteRule::new();
    rule.set_family_name(String::from("cf1"));
    rule.set_column_qualifier(encode_str("r1"));
    rule.set_append_value(encode_str("test_value"));
    
    rules.push(rule);
    
    req.method.payload_mut().set_row_key(encode_str("r1"));
    req.method.payload_mut().set_rules(RepeatedField::from_vec(rules));
    
    let response = req.execute(&token)?;
    println!("{}", serde_json::to_string_pretty(&response)?);
# Ok(())
# }
}
```

##### ReadRows

```
# #![allow(unused_imports)]
extern crate bigtable as bt;
extern crate serde_json;

use bt::request::BTRequest;
use bt::data::ReadModifyWriteRule;
use bt::utils::*;
use bt::method::{BigTable, ReadRows};
# use bt::error::BTErr;

const TOKEN_URL: &'static str = "https://www.googleapis.com/oauth2/v4/token";
const ISS: &'static str = "service_acc@developer.gserviceaccount.com";
const PK: &'static str = "random_rsa_for_tests";

fn main() {
# #[allow(dead_code)]
# fn wrapper() -> Result<(), BTErr> {
    let req = BTRequest {
                      base: None,
                      table: Default::default(),
                      method: ReadRows::new()
                };
    let response = req.execute(&get_auth_token(TOKEN_URL, ISS, PK)?)?;
    println!("{}", serde_json::to_string_pretty(&response)?);
# Ok(())
# }
}
```

##### SampleRowKeys

```
# #![allow(unused_imports)]
extern crate bigtable as bt;
extern crate serde_json;

use bt::request::BTRequest;
use bt::data::ReadModifyWriteRule;
use bt::utils::*;
use bt::method::{BigTable, SampleRowKeys};
# use bt::error::BTErr;

const TOKEN_URL: &'static str = "https://www.googleapis.com/oauth2/v4/token";
const ISS: &'static str = "service_acc@developer.gserviceaccount.com";
const PK: &'static str = "random_rsa_for_tests";

fn main() {
# #[allow(dead_code)]
# fn wrapper() -> Result<(), BTErr> {
 let req = BTRequest {
                      base: None,
                      table: Default::default(),
                      method: SampleRowKeys::new()
                };
 let response = req.execute(&get_auth_token(TOKEN_URL, ISS, PK)?)?;
 println!("{}", serde_json::to_string_pretty(&response)?);
# Ok(())
# }
}
```
