# Before / after

Generic documentation prose rewritten into the airo-mono voice. The examples are
deliberately outside robotics, to show the voice transfers.

---

## Opening a package README

**Before**

> # datastore-client
>
> A powerful and flexible Python client library for interacting with DataStore. This
> library provides a comprehensive, developer-friendly API that makes it easy to build
> robust data pipelines with minimal boilerplate, while offering the flexibility to handle
> advanced use cases.

**After**

> # datastore-client
>
> This package contains a Python client for DataStore, specifically:
> - a synchronous `Client` for one-off reads and writes
> - a batching `BulkWriter` for ingest, which is the only path that gets server-side dedup
> - the row/schema types shared with `datastore-schemas`
>
> Query planning and index management are not covered here — those live in the server
> repo, and this page links to it rather than restating it.

What changed: adjectives out, contents in, scope boundary stated, the one thing a reader
would otherwise get wrong (dedup only on the bulk path) surfaced in the list itself.

---

## A constraint the reader will otherwise hit

**Before**

> Note that it is recommended to ensure that only a single writer instance is active at any
> given time, as concurrent writes may lead to unexpected behaviour.

**After**

> **One writer per table.** A second writer does not error — it wins the last write and
> silently drops the other's buffered rows, so you find out from a row count days later. If
> you need two producers, give them separate staging tables and merge afterwards.

What changed: the rule is bolded and flat; "may lead to unexpected behaviour" is replaced
by what actually happens and how you'd notice; a way forward is offered.

---

## Explaining a design choice

**Before**

> The library uses a custom retry mechanism rather than an existing solution. This was
> chosen for its simplicity and flexibility.

**After**

> ### Why not `tenacity`?
>
> We only need one policy — retry the idempotent verbs on 5xx and on a connection reset,
> with jittered backoff, and never retry a write without an idempotency key. Expressing
> that in `tenacity` meant a decorator per method plus a predicate that reached into the
> response object anyway, so the dependency bought us nothing here. It is about 40 lines in
> [`retry.py`](datastore_client/retry.py).
>
> This is not a provably right choice; if the policy grows a second axis we should revisit
> it.

What changed: gets its own `Why not X?` heading, names the alternative, gives the actual
reasoning with concrete detail, admits the choice is revisitable.

---

## A usage example

**Before**

> ```python
> from datastore_client import Client
>
> client = Client(...)
> # Use the client
> result = client.query(...)
> ```

**After**

> ```python
> from datastore_client import Client
>
> # Construct-and-connect: this blocks until the session is authenticated, or raises with a reason.
> with Client("datastore.internal:7000", token=os.environ["DS_TOKEN"]) as client:
>     rows = client.query("select id, ts from events where ts > ?", since)  # a list of Row, not a cursor
>     print(len(rows))
> ```
>
> Everything this class does not expose stays reachable on `client.session` (raw protocol
> calls, the connection pool, server-side statistics).

What changed: real host, real argument, no `...`; comments carry the two facts the
signature cannot (constructor blocks; `query` materializes); the escape hatch is named.

---

## Wrong-way / right-way

**Before**

> Remember to call `flush()` when you are done writing, otherwise some data may not be
> persisted.

**After**

> Rows are buffered until the buffer fills, so the last partial batch is still in memory
> when your script exits. The following silently loses up to 999 rows:
>
> ```python
> writer = BulkWriter(client, batch_size=1000)
> for row in rows:
>     writer.add(row)
> ```
>
> but this does not:
>
> ```python
> with BulkWriter(client, batch_size=1000) as writer:  # flushes on exit, including on exception
>     for row in rows:
>         writer.add(row)
> ```

What changed: the mechanism is explained first, the loss is quantified, and the fix is the
context manager rather than a reminder the reader has to remember.

---

## A "not supported" table

**Before**

> The following methods are not implemented for the read-only client: `insert`, `update`,
> `delete`, `create_table`.

**After**

> **Not supported** (raises `NotImplementedError`), because the credential cannot carry it:
>
> | Method | Why |
> |---|---|
> | `insert`, `update`, `delete` | the read-only role has no write grant; the server would refuse them anyway, one round-trip later |
> | `create_table` | DDL goes through migrations, never through a client. See [migrations.md](docs/migrations.md) |
> | `query(..., timeout=...)` | the read replica enforces its own statement timeout and ignores the hint |

What changed: a `Why` column, each reason specific, and the alternative named where one
exists.

---

## Troubleshooting entries

**Before**

> ## Troubleshooting
>
> If you experience connection issues, verify your network configuration and credentials. If
> problems persist, check the logs for more information.

**After**

> ## Troubleshooting
>
> - **`AuthError: token rejected`** — the token is fine but scoped to another environment;
>   `DS_TOKEN` from a staging shell is the usual cause. `datastore-client whoami` prints the
>   environment the token resolves to.
> - **Connect hangs for exactly 30 s, then `TimeoutError`** — port 7000 is filtered rather
>   than closed, so nothing comes back. Ping first, and check that the VPN split-tunnel
>   covers `10.20.0.0/16`.
> - **Queries succeed but return zero rows on a table you can see in the UI** — you are on
>   the read replica and the write has not replicated yet. Replication lag is visible in
>   `client.session.stats()["replica_lag_s"]`; it is normally under a second and pathological
>   over ten.

What changed: symptom-first, bolded, one cause and one concrete next step each, with the
command that tells you which case you are in. Nothing is "check the logs".

---

## Ending a setup page

**Before**

> ## Testing
>
> Run the test suite to verify your installation.

**After**

> ## Testing
>
> ```bash
> pytest test/test_client.py           # no network, uses the in-process fake server
> pytest test/ -m integration          # needs a reachable DataStore and DS_TOKEN
> ```
>
> ### Without a server
>
> The package ships an in-process fake that speaks the real protocol, which the unit tests
> run against. To dry-run your own script, serve it on the default port in one terminal and
> point the script at `127.0.0.1` in the other:
>
> ```python
> from datastore_client.testing import FakeServer
>
> server = FakeServer(port=7000, seed_rows=1000)
> server.start()
> input("fake server up, press enter to stop")
> ```
>
> Do this before your first run against production — it catches schema mistakes without
> spending a write quota.
