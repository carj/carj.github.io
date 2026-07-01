---
layout: post
title: Rate Limiting Uploads in pyPreservica
---

[pyPreservica](https://pypreservica.readthedocs.io/) provides functionality to upload packages directly to Preservica using the
[upload_zip_package](https://pypreservica.readthedocs.io/en/latest/api.html#pyPreservica.UploadAPI.upload_zip_package) method in the Upload API.

This allows users to upload packages as quickly as they can be created. This is not necessarily a good thing — 
submitting packages faster than the ingest pipeline can process them can cause queues to back up, and put unnecessary
load on the server, Preservica will queue packages on the server which cannot be processed immediately so 
packages will never be rejected, but this can cause the ingest queue to grow.

To give users better control over their upload throughput, `upload_zip_package` now accepts a `limit_per_minute` 
parameter that caps how many packages are submitted per minute.

### How it works

Rate limiting is implemented using the [pyrate-limiter](https://github.com/vutran1710/PyrateLimiter) library,
which provides a robust leaky-bucket algorithm for controlling request rates in Python applications.

### The leaky bucket algorithm

The leaky bucket algorithm is a common approach to rate limiting. The "bucket" has a fixed capacity
set by `limit_per_minute`. Each upload consumes one slot in the bucket. pyrate-limiter tracks how
many requests have been made in the last 60 seconds using a rolling window — once the bucket is full,
any further upload calls block until enough time has passed for older requests to fall outside that window.

```
  Uploads arriving
  at any rate
       │  │  │
       ▼  ▼  ▼
  ┌───────────────┐
  │  ● ● ● ● ●    │  ← bucket fills up (limit_per_minute slots)
  │               │
  └───────┬───────┘
          │  blocks when full;
          │  unblocks as slots expire
          ▼  after 60 seconds
   Preservica ingest
```

Importantly, the limiter does not space uploads evenly across the minute. If `limit_per_minute=60`,
your script can still fire all 60 uploads in rapid succession — it will then block until the oldest
requests are more than 60 seconds old and slots become available again. The constraint is on
*how many* uploads happen per minute, not *when within* that minute they happen.

The rate limiting does not impact the upload speed to Preservica, this still happens as fast as your
network allows, the rate limiting is only impacting the number of uploads per minute.

The new function signature is:

```python
upload_zip_package(path_to_zip_package, folder=None, callback=None, delete_after_upload=False, limit_per_minute=180)
```

The `limit_per_minute` parameter defaults to `180`, which corresponds to three uploads per second — 
a rate faster than most packages can realistically be created, this is effectively the unlimited rate. 
Reducing this value slows down the upload rate.


### Basic usage

A simple upload with the default rate limit requires no changes to existing code:

```python
from pyPreservica import *

client = UploadAPI()
folder = "9fd239eb-19a3-4a46-9495-40fd9a5d8f93"

package = simple_asset_package("my-document.pdf", parent_folder=folder)

client.upload_zip_package(package)
```

### Reducing the upload rate

If you are running a long bulk ingest with existing packages which already exist and want to avoid placing heavy load on the server,
lower `limit_per_minute` to a more conservative value:

```python
from pyPreservica import *

client = UploadAPI()
folder = "9fd239eb-19a3-4a46-9495-40fd9a5d8f93"

for file in Path("packages").glob("*.zip"):
    client.upload_zip_package(str(file), folder=folder, limit_per_minute=15)
```

Here uploads are capped at 15 per minute, giving the ingest pipeline time to 
process each package before the next one arrives.

### Increasing the upload rate

On a lightly loaded system, or when running an urgent migration, you can raise the limit to 
push packages through more quickly:

```python
from pyPreservica import *

client = UploadAPI()
folder = "9fd239eb-19a3-4a46-9495-40fd9a5d8f93"

for file in Path("packages").glob("*.zip"):
    client.upload_zip_package(str(file), folder=folder, delete_after_upload=True, limit_per_minute=60)
```

> **_NOTE:_** Setting `limit_per_minute` very high will not necessarily make uploads faster overall — 
> especially if you need to create the packages locally, packages will still queue server-side regardless. 
> The default of 180 is a reasonable starting point for most deployments.

### Combining rate limiting with a progress callback

`limit_per_minute` works alongside the existing `callback` parameter, so you can track progress 
while still controlling throughput:

```python
from pyPreservica import *

client = UploadAPI()
folder = "9fd239eb-19a3-4a46-9495-40fd9a5d8f93"

packages = list(Path("packages").glob("*.zip"))

for i, file in enumerate(packages, 1):
    print(f"Uploading {file.name} ({i}/{len(packages)})")
    client.upload_zip_package(str(file), folder=folder, callback=UploadProgressConsoleCallback(file), limit_per_minute=60)
```

### Summary

The `limit_per_minute` parameter gives you straightforward control over upload throughput without
requiring any changes to how packages are created. For most bulk ingest workflows the default of 
180 packages per minute is appropriate, but lowering it to 60 or below is recommended when running 
large overnight migrations where steady, sustained throughput matters more than raw speed.