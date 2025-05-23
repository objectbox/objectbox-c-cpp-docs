---
description: >-
  The "Store" is typically your first touching point with the ObjectBox API. It
  represents a database you can open and get further API object to interact with
  it.
---

# Store

## Store Options

When creating a store, you can provide additional options like the database model aka schema, the location of the database or a maximum size. In the following, we'll show some of the most important options (check the API docs for a complete list).

### Model

The model specifies available entities in an ObjectBox database. It is usually specified using FlatBuffer IDL files processed by the ObjectBox Generator to output a `objectbox-model.h` header file for C and C++ users.&#x20;

{% tabs %}
{% tab title="C++" %}
```cpp
#include "objectbox-model.h"
OBX_model* model = create_obx_model();

obx::Options options(model);
obx::Store store(options);
```
{% endtab %}

{% tab title="C" %}
```c
#include "objectbox-model.h"
OBX_model* model = create_obx_model();

OBX_store_options* opt = obx_opt();
obx_opt_model(opt, model);
obx_store_open(opt);
```
{% endtab %}
{% endtabs %}

### Storage Location

ObjectBox persists data per default to files in a directory named "`objectbox`". Use the following option to configure the directory to your application needs.

{% tabs %}
{% tab title="C++" %}
```cpp
obx::Options options(model);
options.directory("resources/mydb");
obx::Store store(options);
```
{% endtab %}

{% tab title="C" %}
```c
OBX_store_options* opt = obx_opt();
obx_opt_directory(opt, "resources/mydb");
obx_store_open(opt);
```
{% endtab %}
{% endtabs %}

#### In-Memory Databases

{% hint style="info" %}
Available since v0.21.0
{% endhint %}

You can use ObjectBox alsofor non-persistent in-memory databases. To create a memory-backed store, use the directory prefix "`memory:`" and pick a name for the database to address it, e.g. “memory:myApp”. Apart from the special prefix, it's using the same "directory" option call:

{% tabs %}
{% tab title="C++" %}
```cpp
obx::Options options(model);
options.directory("memory:myApp");
obx::Store store(options);
```
{% endtab %}

{% tab title="C" %}
```c
OBX_store_options* opt = obx_opt();
obx_opt_directory(opt, "memory:myApp");
obx_store_open(opt);
```
{% endtab %}
{% endtabs %}

**Note:** in-memory databases are kept after closing a store; they have to be explicitly deleted or are automatically deleted if the creating process exists.

Typical applications are caching and using ObjectBox as an advanced in-memory data structure with all the benefits of using FlatBuffer objects, transactions, relations, querying, etc.... For more details, check out our [blog post on in-memory use cases](https://objectbox.io/in-memory-database-use-cases/).

### Read-Only Access

Setting a database to read-only access only.

{% hint style="info" %}
Advanced Setting: Use previous commit

Recommended to be used together with read-only mode to ensure no data is lost. Ignores the latest data snapshot (committed transaction state) and uses the previous snapshot instead. When used with care (e.g. backup the DB files first), this option may also recover data removed by the latest transaction. Defaults to false.
{% endhint %}

{% tabs %}
{% tab title="C++" %}
```cpp
obx::Options options(model);
options.readOnly(true)
       .usePreviousCommit(true); 
obx::Store store(options);
```
{% endtab %}

{% tab title="C" %}
```c
OBX_store_options* opt = obx_opt();
obx_opt_readonly(opt, true);
obx_opt_use_previous_commit(opt, true);
obx_store_open(opt);
```
{% endtab %}
{% endtabs %}

### File Mode

Specify unix-style file permissions for database files. This parameter is ignored on Windows platforms.

{% tabs %}
{% tab title="C++" %}
```cpp
obx::Options options(model);
options.fileMode(0640); 
obx::Store store(options);
```
{% endtab %}

{% tab title="C" %}
```c
OBX_store_options* opt = obx_opt();
obx_opt_file_mode(opt, 0640);
obx_store_open(opt);
```
{% endtab %}
{% endtabs %}

### Maximum Resource Limits

Several limitations regarding the resources of a Store can be specified.

{% tabs %}
{% tab title="C++" %}
```cpp
obx::Options options(model);
options
  .maxDbSizeInKb(4*1024*1024)
  .maxDataSizeInKb(4*1024*1024)
  .maxReaders(256);
obx::Store store(options);
```
{% endtab %}

{% tab title="C" %}
```c
OBX_store_options* opt = obx_opt();
obx_opt_max_db_size_in_kb(opt,4*1024*1024)
obx_opt_max_data_size_in_kb(opt,3*1024*1024)
obx_opt_max_readers(opt,256)
```
{% endtab %}
{% endtabs %}

#### Maximum Database Size

`maxDbSizeInKb` sets the maximum size the database file can grow to. When applying a transaction (e.g. putting an object) would exceed, it an exception is thrown (C++) or error is returned (C).\
By default, this setting is 1 GB, which should be sufficient for most applications. In general, a maximum size prevents the database from growing indefinitely when something goes wrong (for example data is put in an infinite loop).\
This value can be changed, so increased or also decreased, each time when opening a store.

#### Maximum Data Size

`maxDataSizeInKB` can be set to limit the actual storage of data, excluding system and metadata, in contrast to `maxDbSizeInKB`. Thus, it must be less than the maximum database size.&#x20;

{% hint style="info" %}
This resource limitation tracker is more involved.  Only use this if a stricter, more accurate limit is required.
{% endhint %}

#### Maximum Number of Readers

A "Reader" is short for a thread involved in a read transaction. Readers are an finite resource for which we need to define a maximum number upfront. `maxReaders` sets the maximum number of concurrent readers. The default value (about 126 readers) is enough for most apps and usually you can ignore it completely. However, if the maximum is exceeded the store throws an exception or reports an error (`OBX_ERROR_MAX_READERS_EXCEEDED`). In this case check that your code only uses a reasonable amount of threads.

For highly concurrent setups (e.g. you are using ObjectBox on the server side) it may make sense to increase the number.

{% hint style="info" %}
The internal default is currently 126. So when hitting this limit, try values around 200-500.\
\
Each thread that performed a read transaction and is still alive holds on to a reader slot. These slots only get vacated when the thread ends. Thus, be mindful with the number of active threads.
{% endhint %}

### Consistency and Validation&#x20;

ObjectBox offers several maintenance checks for validation and consistency checking. When the DB is opened initially, ObjectBox can do a&#x20;

* consistency check on (branch and leaf) database pages
* validation over the key/value pairs to check

{% tabs %}
{% tab title="C++" %}
```cilkcpp
obx::Options options(model);
options
  .validateOnOpenPages(1, OBXValidateOnOpenPagesFlags_VisitLeafPages)
  .validateOnKV();
obx::Store store(options);
```
{% endtab %}

{% tab title="C" %}
```c
OBX_store_options* opt = obx_opt();
obx_opt_validate_on_open_pages(opt, 1, OBXValidateOnOpenPagesFlags_VisitLeafPages); 
obx_opt_validate_on_open_kv(opt, OBXValidateOnOpenKvFlags_None);
obx_store_open(opt);
```
{% endtab %}
{% endtabs %}

`validateOnOpenPages` enables a consistency check on the given amount of pages.  To completely disable this test you can pass 0, but we recommend a setting of at least 1. Optional flag `VisitLeafPages` enable validation of leaf pages (by default only branch pages are validated).&#x20;

{% hint style="info" %}
Reliable file systems already guarantee consistency, so this is primarily meant to deal with unreliable OSes, file systems, or hardware. Thus, usually a low number (e.g. 1-20) is sufficient and does not impact startup performance significantly.\
\
**Note:** ObjectBox builds upon ACID storage, which guarantees consistency given that the file system is working correctly (in particular fsync).
{% endhint %}

`validateOnKV` enables validation over the key/value pairs to check, for example, whether they're consistent towards our internal specification. The flags argument is currently an empty enum (except None) for future improvments. &#x20;

### Tuning Asynchronous Operations

Fine-tune asynchronous operations using functions prefixed with `obx_opt_async_` (C) and `Options::async` (C++).

{% tabs %}
{% tab title="C++" %}
```cpp
obx::Options options(model);
options.asyncMaxInTxDuration(1000);
obx::Store store(options);
```
{% endtab %}

{% tab title="C" %}
```
OBX_store_options* opt = obx_opt();
obx_opt_async_max_in_tx_duration(opt, 1000);
obx_store_open(opt);
```
{% endtab %}
{% endtabs %}

Available options for fine-tuning asynchronous operations:

* `MaxInTxDuration` (micros): Max. duration spent in a transaction before queue enforces a commit.
* `MaxInTxOperations` (number of ops): Max. Ops performed in a transaction before queue enforces a commit. This becomes relevant if the queue is constantly populated at a high rate.&#x20;
* `MaxQueueLength` (size): Max. size of queue.
* `MaxTxPoolSize` (size): Default 10000, set to 0 to deactivate pooling.
* `MinorRefillMaxCount` (size): If non-zero, this allows "minor refills" with small batches that came in (off by default).
* `MinorRefillThreshold` (size): Numbers of operations below this value are considered "minor refills".
* `ObjectBytesMaxCacheSize` (size): Total cache size; default: \~ 0.5 MB.
* `ObjectBytesMaxSizeToCache` (size): Maximal size for an object to be cached (only cache smaller ones),
* `PreTxnDelay` (micros): Delay before starting transactions when queue is triggered (e.g. gives a newly starting producer some time to produce more than one a single operation before queue starts.) Note: This value should typically be low to keep latency low and prevent accumulating too much operations.
* `PostTxnDelay` (micros): Similar to `PreTxDelay` but after a transaction was committed. One of the purposes is to give other transactions some time to execute. In combination with \
  `PreTxDelay` this can prolong non-TX batching time if only a few operations are around.
* `ThrottleAtQueueLength`(size): Producers (AsyncTx submitter) is throttled when the queue size hits this.
* `ThrottleMicros` (micros): Sleeping time for throttled producers on each submission.

### Debug Flags

When debugging or fine-tuning performance you can enable [logging](dev-tools-and-debugging.md#logging) of various events at the info level.

{% tabs %}
{% tab title="C++" %}
```cpp
obx::Options options(model);
options.addDebugFlags(
  OBXDebugFlags_LOG_TRANSACTIONS_READ
| OBXDebugFlags_LOG_TRANSACTIONS_WRITE
);
obx::Store store(options);
```
{% endtab %}

{% tab title="C" %}
```c
OBX_store_options* opt = obx_opt();
obx_opt_add_debug_flags(opt, OBXDebugFlags_LOG_TRANSACTIONS_READ
| OBXDebugFlags_LOG_TRANSACTIONS_WRITE);
obx_store_open(opt);
```
{% endtab %}
{% endtabs %}

## Using the Store

Once you created the store, it gives you access to several ways to interact with the data you store in it; aka the database:

* [**Box API**](getting-started.md#working-with-object-boxes)**:** easy to use object-based API with implicit transactions. It is used in combination with the ObjectBox Generator, which generates the source code for data and Box classes. Check the getting started track for examples.
* [**Query API**](queries.md)**:** In combination with the Box API, you can build powerful queries using the query builder. Once built, query instances can be executed multiple times to retrieve data.&#x20;
* [**Explicit Transactions**](transactions.md)**:** Often it's a good idea to group several operations into a single "batch", or more precisely, one transaction. Batched transactions are not only faster, but also consider safe state changes for your data.



