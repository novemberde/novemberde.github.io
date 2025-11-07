---
title: "How to Use Apache Beam's MultiProcessShared (and Why You Need It)"
tags: ["Apache Beam", "Python", "Machine Learning", "GPU", "Memory Management", "Data Pipeline", "Distributed Computing", "Performance Optimization", "ML Infrastructure", "Google Cloud Dataflow"]
date: "2025-11-05T00:00:00+00:00"
ShowBreadCrumbs: true
ShowReadingTime: true
ShowPostNavLinks: true
---

## Overview

If you're running Apache Beam pipelines with GPUs or large ML models, you've probably hit "CUDA out of memory" errors. Here's what's happening: each worker process loads its own copy of your model, eating up memory until everything crashes.

Apache Beam 2.49.0 added `MultiProcessShared` to fix this. It lets you share one copy of a resource (like a GPU model) across all processes on a worker, instead of loading it separately in each process. This can drop your memory usage from 24GB to 3GB.

This guide shows you when you need `MultiProcessShared` and how to use it in production.

## Table of Contents

1. **Background: Understanding Apache Beam's Worker Architecture**
   - Process and Thread Model
   - Default Resource Loading Behavior
   - Memory Challenges in ML Pipelines

2. **The Problem: GPU Memory Overload**
   - Why Multiple Processes Create Issues
   - CUDA Out of Memory Errors
   - Real-World Scenario

3. **Solution: Shared vs MultiProcessShared**
   - Shared: Thread-level Resource Sharing
   - MultiProcessShared: Process-level Resource Sharing
   - When to Use Each

4. **Implementation Guide**
   - Basic Usage Pattern
   - GPU Model Loading Example
   - Connection Pool Management
   - Configuration Tips
     - Unique Tags
     - Don't Close in Teardown
     - Thread Safety with Locks
     - Performance Impact of Locking
     - Dataflow GPU Configuration

5. **Performance Considerations**
   - Memory Usage Improvements
   - Throughput Impact
   - Cost Optimization

6. **Common Pitfalls and Solutions**
   - Pitfall 1: Closing Shared Resources in Teardown
   - Pitfall 2: Forgetting Thread Safety (Locking)
   - Pitfall 3: Forgetting the Tag Parameter
   - Pitfall 4: Serialization Issues
   - Pitfall 5: Not Using Context Manager
   - Pitfall 6: Process-Unsafe Operations
   - Pitfall 7: Incorrect GPU Memory Management
   - Debugging Tips

---

## 1. Background: Understanding Apache Beam's Worker Architecture

### Process and Thread Model

Apache Beam Python SDK uses a multi-process, multi-threaded architecture for parallel data processing:

- **Workers**: Virtual machines or containers that execute your pipeline
- **SDK Harness Processes**: Each worker spawns multiple Python processes (typically 1 per CPU core)
- **Worker Threads**: Each process dynamically creates threads to handle bundle processing

```
Worker VM
├── SDK Harness Process 1
│   ├── Thread 1
│   ├── Thread 2
│   └── Thread N
├── SDK Harness Process 2
│   ├── Thread 1
│   ├── Thread 2
│   └── Thread N
└── SDK Harness Process M
    └── ...
```

### Default Resource Loading Behavior

By default, when a DoFn needs a resource (like an ML model):

1. Each thread loads its own copy during `setup()`
2. Multiple threads in a process = multiple copies
3. Multiple processes on a worker = even more copies
4. You end up with **N × M instances** on one worker (N processes × M threads)

This is fine for small resources. For large ML models (especially GPUs), you're going to run out of memory.

### Memory Challenges in ML Pipelines

Let's look at a typical GPU worker:

- **GPU Memory**: 16 GB (e.g., NVIDIA T4)
- **CPU Cores**: 8 cores → 8 SDK Harness processes
- **Threads per Process**: 2-4 threads
- **ML Model Size**: 3 GB per instance

**Without resource sharing:**
```
8 processes × 1 model per process = 24 GB required
Result: CUDA Out of Memory Error
```

---

## 2. The Problem: GPU Memory Overload

### Why Multiple Processes Create Issues

Let's examine a real scenario from a production ML inference pipeline:

**Problem Statement:**
- Need to process billions of records using GPU-powered embedding models
- Each embedding model requires 3+ GB of GPU memory
- Dataflow workers have multiple CPU cores
- Each core spawns a separate Python process
- Each process attempts to load the model into GPU memory

**What Happens:**

```python
class EmbeddingPredictor(beam.DoFn):
    def setup(self):
        # This gets called in EVERY process, EVERY thread
        self.model = load_embedding_model()  # 3 GB
        self.model.to('cuda')  # Load to GPU

    def process(self, element):
        embedding = self.model.predict(element)
        yield embedding
```

**Result:**
- Process 1 loads model → 3 GB GPU memory used
- Process 2 loads model → 6 GB GPU memory used
- Process 3 loads model → 9 GB GPU memory used
- ...
- Process 8 loads model → **CUDA Out of Memory Error!**

### CUDA Out of Memory Errors

The error typically looks like:

```
RuntimeError: CUDA out of memory. Tried to allocate 3.00 GiB
(GPU 0; 15.78 GiB total capacity; 12.45 GiB already allocated;
2.89 GiB free; 13.67 GiB reserved in total by PyTorch)
```

### Real-World Example

Here's what happened when we ran embedding pipelines at Karrot on Google Cloud Dataflow:

**Before MultiProcessShared:**
- GPU workers crashed frequently with OOM errors
- Had to limit workers to single-process mode (`number_of_worker_harness_threads=1`)
- **Severely limited throughput**
- Higher costs due to underutilized CPU resources

**After MultiProcessShared:**
- Single model instance shared across all processes
- Full utilization of CPU cores
- 3-4x throughput improvement
- Significant cost reduction

---

## 3. Solution: Shared vs MultiProcessShared

Apache Beam has two ways to share resources:

### Shared: Thread-level Resource Sharing

**Purpose:** Share resources across threads within a single process

**Use Cases:**
- Thread-safe connection pools
- Read-only configuration data
- Lightweight caching

**Scope:** Single process only

**Example:**
```python
from apache_beam.utils.shared import Shared

class ThreadSafeDoFn(beam.DoFn):
    def setup(self):
        # Shared across threads in THIS process only
        self.pool = Shared(lambda: create_connection_pool())

    def process(self, element):
        with self.pool.acquire() as pool:
            result = pool.query(element)
            yield result
```

### MultiProcessShared: Process-level Resource Sharing

**Purpose:** Share resources across multiple processes on the same worker

**Use Cases:**
- Large ML models (especially GPU models)
- Expensive-to-initialize resources
- Memory-intensive caches
- Singleton services (API clients with process-level initialization)

**Scope:** All processes on the same worker VM

**Introduced:** Apache Beam Python 2.49.0

**Example:**
```python
from apache_beam.utils.shared import Shared

class GPUModelDoFn(beam.DoFn):
    def setup(self):
        # Shared across ALL processes on this worker
        self.model = Shared(
            lambda: load_gpu_model(),
            tag='gpu-embedding-model'  # Unique identifier
        )

    def process(self, element):
        with self.model.acquire() as model:
            prediction = model.predict(element)
            yield prediction
```

### When to Use Each

| Scenario | Use Shared | Use MultiProcessShared |
|----------|-----------|------------------------|
| Thread-safe object | ✓ | ✓ |
| Small memory footprint (<100 MB) | ✓ | ✓ |
| Large memory footprint (>1 GB) | ✗ | ✓ |
| GPU resources | ✗ | ✓ |
| Process-unsafe operations | ✓ | ✗ |
| Need process isolation | ✓ | ✗ |

**Key Differences:**

1. **Scope:**
   - `Shared`: Thread-level within a single process
   - `MultiProcessShared`: Process-level across all processes on a worker

2. **Memory Impact:**
   - `Shared`: Reduces memory by N (threads per process)
   - `MultiProcessShared`: Reduces memory by N × M (processes × threads)

3. **Serialization:**
   - `Shared`: No special serialization needed
   - `MultiProcessShared`: Resources must be serializable (or use tag-based identification)

---

## 4. Implementation Guide

### Basic Usage Pattern

```python
from apache_beam.utils.shared import Shared

class MyDoFn(beam.DoFn):
    def setup(self):
        self.resource = Shared(self._create_resource, tag='my-resource')

    def _create_resource(self):
        return create_expensive_resource()  # Called once per worker

    def process(self, element):
        with self.resource.acquire() as res:
            yield res.process(element)
```

**Key points:**
- Factory function runs **once** per worker (all processes share it)
- The `tag` makes it share across processes
- Don't close resources in `teardown()` - see Configuration Tips

### GPU Model Loading Example

```python
import threading
from apache_beam.utils.shared import Shared
import torch
from transformers import AutoModel, AutoTokenizer

class GPUEmbeddingDoFn(beam.DoFn):
    def __init__(self, model_name: str):
        self.model_name = model_name

    def setup(self):
        # Share model across processes
        self.model = Shared(self._load_model, tag=f'model-{self.model_name}')
        # Share lock across threads in this process
        self.lock = Shared(lambda: threading.Lock(), tag=f'lock-{self.model_name}')
        # Tokenizer is thread-safe
        self.tokenizer = AutoTokenizer.from_pretrained(self.model_name)

    def _load_model(self):
        model = AutoModel.from_pretrained(self.model_name)
        return model.to('cuda').eval()

    def process(self, text):
        inputs = self.tokenizer(text, return_tensors='pt', truncation=True)
        inputs = {k: v.to('cuda') for k, v in inputs.items()}

        # Acquire lock first, then model
        with self.lock.acquire() as lock:
            with lock:  # Prevent concurrent access
                with self.model.acquire() as model:
                    with torch.no_grad():
                        embeddings = model(**inputs).last_hidden_state[:, 0, :]

        yield embeddings.cpu().numpy().tolist()
```

### Connection Pool Management

```python
from apache_beam.utils.shared import Shared
import httpx

class APIClient(beam.DoFn):
    def setup(self):
        self.client = Shared(self._create_client, tag='http-client')

    def _create_client(self):
        return httpx.Client(timeout=30.0, limits=httpx.Limits(max_connections=100))

    def process(self, item):
        with self.client.acquire() as client:
            response = client.post(API_URL, json=item)
            yield response.json()
```

### Configuration Tips

#### 1. Choose Unique Tags

```python
# ✓ Good: Unique, descriptive
self.model = Shared(load_model, tag=f'model-{model_name}-{version}')

# ❌ Bad: Too generic, might conflict
self.model = Shared(load_model, tag='model')
```

#### 2. Don't Close Shared Resources in Teardown

**CRITICAL:** Don't close shared resources in `teardown()` - other threads/processes are still using them!

```python
# ❌ Wrong - closes resource while others use it
def teardown(self):
    with self.resource.acquire() as res:
        res.close()  # Other DoFn instances will crash!

# ✓ Correct - let process cleanup handle it
def teardown(self):
    pass
```

**Why:** Each DoFn has its own `Shared` wrapper, but they share the **same underlying resource**. If one closes it, others crash when calling `acquire()`.

#### 3. Thread Safety: Most ML Models Need Locking

**IMPORTANT:** `MultiProcessShared` shares resources across processes, but multiple **threads** within each process can still access the resource concurrently. Many ML models are **not thread-safe** and will crash or return incorrect results without locking.

**Thread-safe ML frameworks:**
- ✅ TensorFlow (with default settings)
- ✅ ONNX Runtime (explicitly thread-safe)
- ✅ Some scikit-learn models (check docs)

**NOT thread-safe:**
- ❌ PyTorch models (most operations)
- ❌ Transformers models built on PyTorch
- ❌ TensorRT
- ❌ Some custom CUDA kernels

**How to check if your model is thread-safe:**
1. Check the framework documentation
2. Look for "GIL release" or "thread-safe" mentions
3. When in doubt, assume it's NOT thread-safe and use a lock

**How to add thread safety:**

```python
import threading
from apache_beam.utils.shared import Shared

class SafePyTorchDoFn(beam.DoFn):
    def setup(self):
        # Share model across processes
        self.model = Shared(self._load_model, tag='model')
        # Share lock across threads (prevents concurrent access)
        self.lock = Shared(lambda: threading.Lock(), tag='model-lock')

    def _load_model(self):
        return load_pytorch_model().to('cuda').eval()

    def process(self, element):
        with self.lock.acquire() as lock:
            with lock:  # Only one thread can use model at a time
                with self.model.acquire() as model:
                    result = model.predict(element)
        yield result
```

**Important:** The lock must also use `Shared()` with a tag!

```python
# ❌ Wrong - each DoFn instance gets its own lock
def setup(self):
    self.lock = threading.Lock()  # Different lock per instance!
    self.model = Shared(load_model, tag='model')

# ✓ Correct - all instances share the same lock
def setup(self):
    self.lock = Shared(lambda: threading.Lock(), tag='model-lock')
    self.model = Shared(load_model, tag='model')
```

If you do `self.lock = threading.Lock()`, each DoFn instance gets a different lock object, so threads won't coordinate access. Both the model AND lock need `Shared()` with tags.

**Why this matters:**

Without locking, you'll see intermittent failures:
```
RuntimeError: CUDA error: an illegal memory access was encountered
Segmentation fault (core dumped)
```

These are **race conditions** from multiple threads using the model simultaneously.

#### 4. Performance Impact of Locking

**Question:** Won't locking kill my throughput?

**Answer:** It depends on your model inference time:

- **Fast models (<10ms):** Lock contention can reduce throughput by 20-30%
- **Medium models (10-50ms):** Lock contention reduces throughput by 5-15%
- **Slow models (>50ms):** Lock contention is negligible (<5%)

Most GPU models are slow enough that locking overhead is minimal. The memory savings from `MultiProcessShared` far outweigh the lock overhead.

**Example throughput comparison:**

| Configuration | Throughput | Memory |
|--------------|-----------|---------|
| No sharing (crashes) | 0 records/sec | OOM ❌ |
| MultiProcessShared + Lock | 4,200 records/sec | 3 GB ✅ |
| Ideal (no lock needed) | 4,500 records/sec | 3 GB ✅ |

The 7% overhead from locking is acceptable compared to not working at all.

#### 5. Dataflow GPU Configuration

When using GPU workers with MultiProcessShared:

```python
# Pipeline options for GPU workers
pipeline_options = PipelineOptions([
    '--runner=DataflowRunner',
    '--project=my-project',
    '--region=us-central1',
    '--worker_machine_type=n1-standard-8',
    '--disk_size_gb=50',
    '--experiments=use_runner_v2',
    '--experiments=enable_gpu',
    '--worker_accelerator=type:nvidia-tesla-t4;count:1;install-nvidia-driver',
    '--number_of_worker_harness_threads=4',  # Can use multiple threads now!
    '--sdk_container_image=gcr.io/project/beam-gpu:latest'
])
```

---

## 5. Performance Considerations

### Memory Usage Improvements

**Before MultiProcessShared:**
```
8 processes × 3 GB per model = 24 GB GPU memory required
Result: Pipeline fails with OOM
```

**After MultiProcessShared:**
```
1 shared model × 3 GB = 3 GB GPU memory required
Result: Pipeline runs successfully with 13 GB headroom
```

**Measurement:**
```python
import torch

class ModelDoFn(beam.DoFn):
    def setup(self):
        self.model = Shared(
            factory_function=self._load_model,
            tag='shared-gpu-model'
        )

    def _load_model(self):
        model = load_model()
        print(f"GPU Memory Allocated: {torch.cuda.memory_allocated() / 1e9:.2f} GB")
        print(f"GPU Memory Cached: {torch.cuda.memory_reserved() / 1e9:.2f} GB")
        return model
```

### Throughput Impact

Our production metrics at Karrot:

| Metric | Before | After | Improvement |
|--------|--------|-------|-------------|
| Worker Processes | 1 | 8 | 8x |
| Throughput (records/sec) | 1,200 | 4,500 | 3.75x |
| GPU Utilization | 45% | 85% | 1.9x |
| Cost per Million Records | $12 | $4 | 3x reduction |

**Why only 3.75x instead of 8x?**
- GPU becomes the bottleneck (which is what you want)
- Network I/O overhead
- Beam bundle processing overhead
- Still a huge win

### Cost Optimization

**Strategy: Consolidating Pipelines**

Before MultiProcessShared, we had to run separate pipelines for different models:

```
Pipeline A: Model A (1 GPU worker)
Pipeline B: Model B (1 GPU worker)
Pipeline C: Model C (1 GPU worker)
Total: 3 GPU workers × $2.50/hour = $7.50/hour baseline
```

After MultiProcessShared, we can consolidate:

```python
class MultiModelDoFn(beam.DoFn):
    def setup(self):
        self.model_a = Shared(lambda: load_model('A'), tag='model-a')
        self.model_b = Shared(lambda: load_model('B'), tag='model-b')
        self.model_c = Shared(lambda: load_model('C'), tag='model-c')

    def process(self, element):
        model_type = element['model_type']

        if model_type == 'A':
            with self.model_a.acquire() as model:
                yield model.predict(element)
        elif model_type == 'B':
            with self.model_b.acquire() as model:
                yield model.predict(element)
        elif model_type == 'C':
            with self.model_c.acquire() as model:
                yield model.predict(element)
```

**Result:**
- 1 GPU worker handles all three models
- Cost: $2.50/hour baseline
- **Savings: 66% reduction in baseline costs**

---

## 6. Common Pitfalls and Solutions

### Pitfall 1: Closing Shared Resources in Teardown

```python
# ❌ Wrong
def teardown(self):
    with self.model.acquire() as model:
        model.close()  # Breaks other DoFn instances!

# ✓ Correct
def teardown(self):
    pass  # Let process cleanup handle it
```

**What happens:** Thread 1 closes model → Threads 2-N crash with `RuntimeError` or `NoneType`

### Pitfall 2: Forgetting Thread Safety (Locking)

```python
# ❌ Wrong - PyTorch without lock
def setup(self):
    self.model = Shared(load_pytorch_model, tag='model')

def process(self, element):
    with self.model.acquire() as model:
        result = model.predict(element)  # Multiple threads access concurrently!

# ✓ Correct - with shared lock
def setup(self):
    self.model = Shared(load_pytorch_model, tag='model')
    self.lock = Shared(lambda: threading.Lock(), tag='model-lock')

def process(self, element):
    with self.lock.acquire() as lock:
        with lock:  # Serialize access
            with self.model.acquire() as model:
                result = model.predict(element)
```

**Result without lock:** `RuntimeError: CUDA error` or `Segmentation fault`

**Remember:** PyTorch/Transformers need locks. TensorFlow/ONNX usually don't.

### Pitfall 3: Forgetting the Tag Parameter

```python
# ❌ Wrong - no tag, each process loads separately
self.model = Shared(lambda: load_model())

# ✓ Correct - tag enables cross-process sharing
self.model = Shared(lambda: load_model(), tag='model')
```

### Pitfall 4: Serialization Issues

```python
# ❌ Wrong - model not serializable
def __init__(self, model):
    self.model = model

# ✓ Correct - store path, load in setup()
def __init__(self, model_path):
    self.model_path = model_path

def setup(self):
    self.model = Shared(lambda: load_model(self.model_path), tag='model')
```

### Pitfall 5: Not Using Context Manager

```python
# ❌ Wrong - doesn't release properly
model = self.model.acquire()
result = model.predict(element)

# ✓ Correct - auto-releases with context manager
with self.model.acquire() as model:
    result = model.predict(element)
```

### Pitfall 6: Process-Unsafe Operations

```python
# ❌ Wrong - SQLite is not process-safe
self.db = Shared(lambda: sqlite3.connect('data.db'), tag='db')

# ✓ Correct - use process-safe database
self.db = Shared(lambda: create_postgres_pool(), tag='db')
```

### Pitfall 7: Incorrect GPU Memory Management

```python
# ❌ Wrong - not explicitly moving to GPU
def _load_model(self):
    return load_model()

# ✓ Correct - explicit GPU placement and verification
def _load_model(self):
    torch.cuda.empty_cache()  # Clear GPU first
    model = load_model().to('cuda').eval()
    print(f"GPU Memory: {torch.cuda.memory_allocated() / 1e9:.2f} GB")
    return model
```

### Debugging Tips

**1. Verify resource sharing (check same object ID across processes):**

```python
def _load_model(self):
    model = load_model()
    print(f"[PID {os.getpid()}] Loaded model: {id(model)}")
    return model

def process(self, element):
    with self.model.acquire() as model:
        print(f"[PID {os.getpid()}] Using model: {id(model)}")  # Same ID!
```

**2. Monitor GPU memory:**

```python
def _load_model(self):
    model = load_model().to('cuda')
    gb = torch.cuda.memory_allocated() / 1e9
    print(f"GPU Memory: {gb:.2f} GB")
    return model
```

**3. Profile performance:**

```python
import time

def process(self, element):
    start = time.time()
    with self.model.acquire() as model:
        result = model.predict(element)
    print(f"Inference: {time.time() - start:.3f}s")
    yield result
```

---

## Conclusion

If you're running Apache Beam pipelines with large models or GPU workloads, `MultiProcessShared` fixes the memory problem that forces you to either run single-process (slow) or crash with OOM errors.

The results from our production usage:
- 8-10x reduction in memory per worker
- No more CUDA OOM errors
- 3-4x higher throughput since we can use all CPU cores
- 3x cost reduction from better resource utilization

### Things to remember

- **Don't close shared resources in `teardown()`** - this is the #1 mistake. Each DoFn has its own wrapper but shares the underlying resource. If one closes it, others will crash
- **Add threading.Lock for PyTorch/Transformers models** - these are NOT thread-safe. Create a shared lock with `Shared(lambda: threading.Lock(), tag='lock')` and use `with self.lock.acquire() as lock:` before accessing the model
- Always use a unique `tag` parameter - without it, you're not actually sharing across processes
- Use the `with resource.acquire()` context manager or you'll have resource leaks
- Check if your ML framework is thread-safe (TensorFlow: yes, PyTorch: no, ONNX: yes, TensorRT: no)
- Verify GPU memory allocation with `torch.cuda.memory_allocated()` to confirm sharing works
- If you have multiple models, consider consolidating pipelines to run on the same workers
- Test with realistic load - lock overhead is usually <10% for GPU models

### When to Use MultiProcessShared

| Scenario | Use it? |
|----------|---------|
| GPU models (>1 GB) | Yes, always |
| Large CPU models (>500 MB) | Yes |
| Connection pools | Yes, if you have high traffic |
| Small resources (<100 MB) | Probably not worth the overhead |
| Process-unsafe resources (like SQLite) | No - will cause problems |

### Further Resources

- [Apache Beam Python SDK Documentation](https://beam.apache.org/documentation/sdks/python/)
- [Shared Class Reference](https://beam.apache.org/releases/pydoc/current/apache_beam.utils.shared.html)
- [Google Cloud Dataflow GPU Workers](https://cloud.google.com/dataflow/docs/guides/using-gpus)
- [Apache Beam Performance Guide](https://beam.apache.org/documentation/runtime/performance/)
