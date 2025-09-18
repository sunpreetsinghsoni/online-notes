# 1) Multithreading: the mental model

- **Thread**: a lightweight path of execution inside a process. Threads share the same memory space (heap), file descriptors, and other process resources.

- **Why use threads?**
    - **I/O-bound tasks**: network scans, HTTP requests, reading/writing sockets, log tailing → threads can overlap I/O waits.
    - **Latency hiding**: while one thread blocks on I/O, others run.
- **Why not threads?**
    - **CPU-bound tasks** in CPython are bottlenecked by the **GIL** (Global Interpreter Lock). Use **multiprocessing** or native extensions (C, Numba) for CPU-heavy exploits/crypto/cracking.
- **Context switching**: The OS scheduler swaps running threads; too many threads can thrash caches and reduce performance.
- **Safety**: Threads share memory → race conditions, deadlocks, state corruption, and side-channel/timing leakage are possible.

# 2) The CPython GIL (must know this)

- CPython has a **Global Interpreter Lock** allowing only one thread to execute Python bytecode at a time.
- **I/O-bound**: not a big deal—blocking I/O releases the GIL (e.g., socket, file, `time.sleep`).
- **CPU-bound**: GIL serializes threads → no speedup. Choose:
    - `multiprocessing`
    - `concurrent.futures.ProcessPoolExecutor`
    - vectorized/native libraries (OpenSSL, zlib) that release the GIL internally.

# 3) `threading` building blocks

## 3.1 Thread lifecycle

```python
import threading

def worker(arg):
    print("work", arg)

t = threading.Thread(target=worker, args=(42,), daemon=False, name="scanner-1")
t.start()
t.join(timeout=5)  # wait; returns when done or timeout
```

- `daemon=True`: thread will not block process exit (good for background, **dangerous** if you need cleanup).
- Inspect:

```python
threading.current_thread().name
threading.active_count()
threading.enumerate()
```

# 3.2 Synchronization primitives

- **Lock**: mutual exclusion (mutex).
- **RLock**: re-entrant lock; same thread can `acquire` multiple times (useful when functions that acquire the same lock call each other).
- **Semaphore(n)**: allow up to `n` concurrent entries (rate/connection limiting).
- **BoundedSemaphore**: like `Semaphore` but prevents over-release.
- **Event**: one-to-many signal—useful for cancellation/stop flags.
- **Condition**: lock + wait/notify for complex state coordination.
- **Barrier(parties)**: wait until N threads reach the barrier.
- **Queue.Queue**: thread-safe FIFO (the easiest way to share work).

### 1) `Lock` (mutex)

**When to use:** protect a simple shared resource (counters, file writes, credential cache).

```python
import threading, time

log_lock = threading.Lock()
shared_counter = 0

def worker():
    global shared_counter
    for _ in range(1000):
        # network work simulated
        time.sleep(0.001)
        with log_lock:             # critical section
            shared_counter += 1

threads = [threading.Thread(target=worker) for _ in range(10)]
for t in threads: t.start()
for t in threads: t.join()
print("counter:", shared_counter)
```

**Cybersec use-case:** multiple scanner threads write discovered credentials or scan results to a single file/structure. Wrap writes in a lock to avoid interleaved or corrupted lines that could leak partial secrets.

**Gotcha/security note:** keep critical sections tiny. Never hold a lock during network I/O — attacker-controlled delays could cause DoS by stalling other threads.


### 2) `RLock` (re-entrant lock)

**When to use:** same thread needs to acquire the lock recursively (nested function calls).

```python
import threading

rlock = threading.RLock()
state = {}

def outer():
    with rlock:
        state['outer'] = True
        inner()

def inner():
    with rlock:            # safe: same thread can re-acquire
        state['inner'] = True

t = threading.Thread(target=outer)
t.start(); t.join()
print(state)
```

**Cybersec use-case:** complex agent where top-level code holds a lock and calls library code that also needs to modify the same global state. RLock avoids deadlock from re-acquire.

**Gotcha/security note:** reentrancy can mask design issues. Prefer explicit small locks when possible to reduce chance of programmer error.

### 3) `Semaphore` (counting semaphore)

**When to use:** limit concurrent operations (concurrent connections, concurrent exploit attempts).

```python
import threading, time, socket

sem = threading.Semaphore(20)  # allow 20 concurrent probes

def probe(target):
    with sem:
        s = socket.socket()
        s.settimeout(1)
        try:
            s.connect((target, 80))
            print("open", target)
        except Exception:
            pass
        finally:
            s.close()

targets = ["192.0.2." + str(i) for i in range(1,101)]
threads = [threading.Thread(target=probe, args=(t,)) for t in targets]
[t.start() for t in threads]
[t.join() for t in threads]
```

**Cybersec use-case:** avoid overwhelming a target or your own outbound NAT by capping concurrent probes.

**Gotcha/security note:** Semantics are “permit up to N at once.” Misconfigured semaphore may still allow too many connections if other parts create sockets outside the scope.

### 4) `BoundedSemaphore`

**When to use:** same as `Semaphore` but catches programming errors (over-release).

```python
from threading import BoundedSemaphore

bsem = BoundedSemaphore(3)

def task():
    bsem.acquire()
    try:
        # do work
        pass
    finally:
        bsem.release()
```

**Cybersec use-case:** protecting a limited pool of resources (e.g., limited set of TLS contexts or hardware tokens) — prevents accidental `release()` calls corrupting accounting.

**Gotcha/security note:** `release()` more times than `acquire()` raises `ValueError` — good for catching bugs early.

### 5) `Event` (one-to-many flag)

**When to use:** broadcast a shutdown or alert to worker threads.

```python
import threading, time

stop_event = threading.Event()

def worker(i):
    while not stop_event.is_set():
        # perform periodic work
        print("worker", i, "tick")
        time.sleep(1)

threads = [threading.Thread(target=worker, args=(i,)) for i in range(3)]
for t in threads: t.start()

# run for 3 seconds then signal stop
time.sleep(3)
stop_event.set()   # wakes all workers
for t in threads: t.join()
```

**Cybersec use-case:** main controller detects compromise or operator interrupt and signals all monitoring threads to stop collecting sensitive artifacts and securely flush logs.

**Gotcha/security note:** `Event` is cooperative. Threads must check `is_set()` regularly — long blocking operations may delay shutdown.

### 6) `Condition`

**When to use:** producer/consumer with predicates, or complex state coordination (e.g., when one thread must wait for a certain security token to be issued).

```python
import threading, time
cond = threading.Condition()
token = None

def producer():
    global token
    time.sleep(1)
    with cond:
        token = "SECRET-TOKEN"
        cond.notify_all()   # wake waiting consumers

def consumer():
    with cond:
        while token is None:
            cond.wait(timeout=5)
        print("got token:", token)

t1 = threading.Thread(target=consumer)
t2 = threading.Thread(target=producer)
t1.start(); t2.start()
t1.join(); t2.join()
```

**Cybersec use-case:** credential rotation: background thread fetches new API tokens and notifies refreshable worker threads to swap credentials atomically.

**Gotcha/security note:** Always check predicate in a loop. Spurious wakeups happen; re-check conditions to avoid using stale/invalid tokens.

### 7) `Barrier`

**When to use:** synchronize threads at phase boundaries (e.g., simultaneously start a coordinated attack or start logging at same time).

```python
import threading, time
bar = threading.Barrier(3)

def phase(i):
    print(f"worker {i} ready")
    bar.wait()   # block until all 3 reach here
    print(f"worker {i} proceed")
    # do phase work

threads = [threading.Thread(target=phase, args=(i,)) for i in range(3)]
[t.start() for t in threads]
[t.join() for t in threads]
```

**Cybersec use-case:** test harness where multiple simulated clients must reach a target at the exact time (load-testing a server).

**Gotcha/security note:** If any thread fails to reach the barrier, others block forever; use `timeout` or handle `BrokenBarrierError`.

### 8) `Queue` (thread-safe FIFO) — the most common primitive

**When to use:** producer-consumer pipelines, backpressure.

```python
import threading, queue, time

q = queue.Queue(maxsize=500)

def producer():
    for i in range(1000):
        q.put(i)   # blocks when full
    # signal done
    for _ in range(3): q.put(None)

def consumer(i):
    while True:
        item = q.get()
        if item is None:
            q.task_done()
            break
        # process item (e.g., analyze packet)
        print(f"consumer-{i} got {item}")
        q.task_done()

threads = [threading.Thread(target=consumer, args=(i,)) for i in range(3)]
for t in threads: t.start()
producer_thread = threading.Thread(target=producer)
producer_thread.start()

q.join()
for t in threads: t.join()
producer_thread.join()
```

**Cybersec use-case:** pipeline: packet capture → parsing → signature matching → storage. Use multiple stages with bounded queues to provide backpressure so a burst of traffic doesn’t exhaust memory.

**Gotcha/security note:** Always call `task_done()` after `get()` if you use `join()`. Use `maxsize` to avoid unbounded memory growth when the consumer lags.

### 9) Thread-local storage (`threading.local`)

**When to use:** per-thread context (sessions, TLS, per-connection state) without passing objects around.

```python
import threading
import requests
tls = threading.local()

def get_session():
    if not hasattr(tls, "session"):
        tls.session = requests.Session()   # one session per thread
    return tls.session

def worker(url):
    s = get_session()
    r = s.get(url, timeout=3)
    print(threading.current_thread().name, r.status_code)

threads = [threading.Thread(target=worker, args=("https://example.com",)) for _ in range(5)]
[t.start() for t in threads]
[t.join() for t in threads]
```

**Cybersec use-case:** ensure each scanner thread uses its own connection/session to avoid sharing cookies/credentials that could leak between threads.

**Gotcha/security note:** thread-local objects are not automatically cleaned up until thread exit — long-lived threads keep resources; close/cleanup explicitly when done.

### 10) Combining primitives — common patterns

### Producer/consumer with backpressure + graceful shutdown

```python
import threading, queue, time

q = queue.Queue(maxsize=200)
stop = threading.Event()

def producer():
    i = 0
    while not stop.is_set():
        q.put(i, timeout=1)
        i += 1
    # signal consumers
    for _ in range(4): q.put(None)

def consumer():
    while True:
        item = q.get()
        if item is None:
            q.task_done(); break
        try:
            # process
            pass
        finally:
            q.task_done()

# start threads...
```

**Cybersec use-case:** capture pipeline: live packet capture may be bursty — a bounded queue prevents memory blowup and `Event` allows CTRL-C to stop producers gracefully.

### 11) Security-specific race example: TOCTOU protection

**Problem:** multiple threads validate file path then open file — attacker may swap symlink between check and open.

```python
import os, threading

def safe_write(path, data):
    # open with O_CREAT|O_EXCL to avoid TOCTOU when creating; use os.replace for atomic replace
    fd = os.open(path + ".tmp", os.O_WRONLY | os.O_CREAT | os.O_TRUNC, 0o600)
    try:
        os.write(fd, data)
    finally:
        os.close(fd)
    os.replace(path + ".tmp", path)  # atomic rename

lock = threading.Lock()
def worker(path, data):
    with lock:              # coarse but safe: serializes writers
        safe_write(path, data)
```

**Cybersec note:** application-level locks help but are not sufficient across processes. Use OS-level atomic ops (`os.replace`, `O_EXCL`) to protect file integrity across processes.

### 12) Debugging & safe practices (security checklist)

- Use `Queue` instead of manual locks where possible.
- Timeouts on waits, `acquire(timeout=...)`, `Barrier.wait(timeout=...)` — avoid forever-blocking.
- Name threads and include thread name in logs for incident analysis.
- Avoid storing secrets in global shared variables; if necessary, protect with locks and use in-memory secure erase if possible.
- Reduce attack surface: restrict number of threads handling incoming untrusted connections (Semaphore).
- For networking, always set socket timeouts and use per-thread SSL contexts when needed.
- Validate and sanitize inputs before passing to worker threads to avoid race-based exploits.
- Prefer immutable shared data where possible to avoid races.

### Lock example (race vs. fixed)

```python
# RACE CONDITION
count = 0

def inc():
    global count
    for _ in range(100000):
        count += 1  # not atomic

# FIX
lock = threading.Lock()
def inc_safe():
    global count
    for _ in range(100000):
        with lock:
            count += 1
```

## 3.3 Thread-local storage

```python
tls = threading.local()

def worker():
    tls.request_id = gen_id()
    handle(tls.request_id)  # isolated per thread
```

Use for per-connection state without sharing globals.

## 3.4 Exceptions in threads

- Exceptions inside a `Thread` **don’t propagate** to the main thread by default. You must capture and communicate them (Queue/Event) or use `ThreadPoolExecutor` (which surfaces exceptions via `future.result()`).

# 4) Higher-level: `concurrent.futures.ThreadPoolExecutor`

The ergonomic way to run many tasks:

```python
from concurrent.futures import ThreadPoolExecutor, as_completed
import requests

urls = [...]
def fetch(u):
    r = requests.get(u, timeout=5)
    return u, r.status_code, len(r.content)

with ThreadPoolExecutor(max_workers=50) as pool:
    futures = [pool.submit(fetch, u) for u in urls]
    for fut in as_completed(futures, timeout=60):
        u, code, size = fut.result()    # raises if fetch() failed
```

- Great for scanners/crawlers: bound `max_workers`, get results & exceptions, easy timeouts.

# 5) Patterns you’ll actually use in security work

## 5.1 Producer–consumer work queue (port scan / URL crawl)

```python
import threading, queue, socket

q = queue.Queue()
results = []
lock = threading.Lock()

def scan(host, port, timeout=0.5):
    with socket.socket() as s:
        s.settimeout(timeout)
        try:
            s.connect((host, port))
            return True
        except:
            return False

def worker():
    while True:
        item = q.get()
        if item is None:  # poison pill
            q.task_done()
            break
        host, port = item
        openp = scan(host, port)
        with lock:
            results.append((port, openp))
        q.task_done()

# enqueue work
host = "192.0.2.10"
for port in range(1, 1025):
    q.put((host, port))

# start pool
threads = [threading.Thread(target=worker, daemon=False) for _ in range(200)]
for t in threads: t.start()

q.join()  # wait until all tasks processed
for _ in threads: q.put(None)  # stop
for t in threads: t.join()

print(sum(1 for _,openp in results if openp), "open ports")
```

Notes:

- `Queue` guarantees thread safety.
- “Poison pill” pattern for clean shutdown.
- Use `lock` only around shared writes (narrow critical sections).

## 5.2 Bounded concurrency (don’t DOS yourself or the target)

```python
sem = threading.Semaphore(50)

def probe(target):
    with sem:
        # do network request / handshake
        pass
```

## 5.3 Coordinated cancellation with `Event`

```python
stop = threading.Event()

def worker():
    while not stop.is_set():
        do_step()
        if error_condition:
            stop.set()

# start threads...
# later
stop.set()
```

## 5.4 Barriers for multi-phase tasks

```python
bar = threading.Barrier(5)

def phase_worker(i):
    phase1(i)
    bar.wait()  # all reach here
    phase2(i)
```

## 5.5 Thread-safe logging

```python
import logging, sys
logging.basicConfig(stream=sys.stdout, level=logging.INFO)
log = logging.getLogger("scanner")
log.info("thread-safe")  # Python logging uses internal locks
```

# 6) Performance tuning (with security sense)

- **Right size pools**: For I/O, try `min(32, 5 * cpu_count)` to start; adjust after measuring.
- **Pin timeouts everywhere**: sockets, HTTP clients → prevents thread pileups / resource exhaustion.
- **Limit in-flight work**: bounded `Queue(maxsize=N)` so producers can’t flood memory.
- **Batch & reuse**: connection reuse (HTTP keep-alive), session pools.
- **Avoid GIL fights**: never use threads for CPU-bound crypto/cracking; use `multiprocessing` or tools built in C.

# 7) Debugging, profiling, and observability

- **Deadlocks**: if it hangs, dump stacks:

```python
import sys, threading, traceback
for t in threading.enumerate():
	print(t, ''.join(traceback.format_stack(sys._current_frames()[t.ident])))
```

- **`faulthandler`**:

```python
import faulthandler; faulthandler.dump_traceback_later(10, repeat=True)
```

- **Log thread names**:

```python
logging.basicConfig(format='%(asctime)s %(threadName)s %(levelname)s: %(message)s')
```

- **Resource usage**: watch file descriptors (`lsof`), sockets, memory; leaks in threads often come from unjoined threads or unbounded queues.

# 8) Concurrency bugs you must recognize (security angle)

- **Race conditions** (TOCTOU): Check-then-use on files, permissions, or existence can be exploited if another thread/process changes state between steps. Use atomic operations, locks, or OS primitives (`os.replace`, `open(..., O_EXCL)` via `os.open`).
- **Deadlocks**: A waits on B’s lock while B waits on A. Prevent by fixed **lock ordering**, prefer high-level primitives (Queue), and use timeouts on `acquire`.
- **Livelock**: threads run but make no progress (e.g., back off/retry simultaneously). Randomized backoff helps.
- **Starvation**: a thread never gets the lock. Keep critical sections minimal; avoid long I/O while holding locks.
- **Data races leading to info leaks**: partial writes/reads of shared buffers can expose internal state or keys.
- **Timing side channels**: thread scheduling can change timing; if you measure timings (e.g., crypto or auth), isolate from noise or use constant-time ops.
- **DoS risks**: unbounded thread creation or unbounded queues → memory/FD exhaustion. Always cap threads and queue sizes.
- **Non thread-safe libs**: some libraries aren’t thread-safe (or only if each thread has its own session). Example: SQLite default connection isn’t threadsafe unless configured; use per-thread connections or `check_same_thread=False` and your own locks.

# 9) Files, sockets, and HTTP safely

## Sockets (common in scanners)

- Always set `settimeout`.
- Use per-thread sockets; don’t share a socket without a lock.
- Close sockets (`with socket.socket() as s:`).
- For TLS, use `ssl.SSLContext` per thread or a safely shared, read-only context.
## HTTP (requests)

- Use a **Session per thread** or a session pool guarded by a lock; Sessions aren’t strictly thread-safe.

```python
import threading, requests
tls = threading.local()

def get_session():
    if not hasattr(tls, "s"):
        tls.s = requests.Session()
        tls.s.headers.update({"User-Agent": "your-tool/1.0"})
    return tls.s
```

# 10) Threading vs. asyncio vs. multiprocessing

- **Threading**: simple mental model, great for I/O, easy to integrate with blocking libraries, but shared-state hazards + GIL.
- **asyncio**: best when your stack is async-friendly (async sockets/HTTP). One thread, many coroutines, fewer races, but requires async-compatible libs.
- **Multiprocessing**: true parallelism for CPU-bound tasks; heavier IPC & memory.

A hybrid that works well in security tools: **asyncio** for high-scale network I/O, **process pool** for CPU-heavy parsing/crypto, or **thread pool** for blocking 3rd-party libs.

# 11) Clean shutdown patterns (don’t leave messes)

- Use **Events** or **poison pills** to stop workers.
- Always `join()` threads you created (unless daemon by design).
- Timeouts on everything; catch `KeyboardInterrupt` to trigger graceful stop.

```python
stop = threading.Event()

def main():
    threads = [Thread(target=worker) for _ in range(50)]
    for t in threads: t.start()
    try:
        run_cli_loop()
    except KeyboardInterrupt:
        stop.set()
    finally:
        for t in threads: t.join(timeout=2)
```

# 12) Security checklists (practical)

**When building a threaded scanner/agent:**

-  Bound `max_workers`, `Queue(maxsize=…)`
-  Timeouts on sockets/HTTP/joins
-  Use `Queue` to pass results; avoid shared mutable state
-  Wrap critical sections with locks (short!)
-  Handle exceptions in workers; surface them to main
-  Respect target rate limits; use `Semaphore` or token bucket
-  Log thread name & structured fields (target, port, error)
-  Drop privileges if possible before worker threads (Linux)
-  Avoid sensitive data in logs; scrub keys/tokens

**When touching files/resources:**

-  Use atomic writes (`tempfile` + `os.replace`)
-  Validate paths (no traversal), don’t trust external input
-  Beware TOCTOU (re-check after acquiring the lock)

**When mixing CPU and I/O:**

-  Offload CPU heavy parts to processes
-  Keep threads for blocking I/O and glue code

# 13) Mini “gotchas” you’ll hit

- `print` from 200 threads makes jumbled output → use logging or a `print_lock`.
- Holding a lock while doing network I/O → throughput collapses, risk of deadlocks.
- Daemon threads swallow cleanup; the process may exit mid-write/flush.
- `Thread.join()` with no timeout can hang your shutdown sequence.
- `random` is global and not thread-safe for reproducibility; consider per-thread RNG (`random.Random()` instances) if you care.

# 14) Quick reference: common APIs

```python
# Thread
t = threading.Thread(target=fn, args=(...), kwargs={...}, daemon=False, name="t"); t.start(); t.join()

# Locks
lock = threading.Lock()
with lock: critical()

r = lock.acquire(timeout=1.0)
if r:
    try: critical()
    finally: lock.release()

# Semaphore
sem = threading.Semaphore(10)
with sem: do()

# Event
evt = threading.Event()
evt.set(); evt.clear(); evt.wait(timeout=5)

# Condition
cond = threading.Condition()
with cond:
    while not predicate():
        cond.wait(timeout=1)
    # do work
# elsewhere
with cond:
    update_state()
    cond.notify()      # or cond.notify_all()

# Barrier
bar = threading.Barrier(5)
bar.wait(timeout=10)

# Queue
q = queue.Queue(maxsize=100)
q.put(item, timeout=1)
item = q.get(timeout=1); q.task_done()
q.join()
```

# 15) What to practice (cyber-focused)

- Build a threaded TCP port scanner with bounded concurrency and graceful stop.
- Write a threaded HTTP crawler with per-thread sessions and robots/rate-limit compliance.
- Create a log tailer that tails multiple files over the network, aggregates to a central sink.
- Implement a multi-stage pipeline: capture (pcap) → parse → store, using `Queue` stages with backpressure.
