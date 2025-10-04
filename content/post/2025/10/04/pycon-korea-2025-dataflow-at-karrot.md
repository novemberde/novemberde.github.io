---
title: "AI Platform with Python at Karrot - Inference Pipeline Part(Pycon Korea 2025)"
tags: ["Python", "Apache Beam", "Google Cloud Dataflow", "ML Infrastructure", "PyConKR 2025", "Machine Learning", "Data Pipeline", "GPU Computing", "Embedding", "Karrot", "Kubeflow", "TFX", "Vertex AI", "GCP", "Protobuf", "LLM", "vLLM", "Kafka", "ANN", "Vector Search", "MLOps", "DevOps", "Monitoring", "Alerting", "SDK Development"]
date: "2025-10-04T00:30:00+00:00"
ShowBreadCrumbs: true
ShowReadingTime: true
ShowPostNavLinks: true
---

## Overview

This article covers our journey building a large-scale ML inference pipeline at Karrot (당근) using Python, Apache Beam, and Google Cloud Dataflow. The presentation was given at PyConKR 2025 by Park JunSeong and Byun Kyuhyun from the ML Infrastructure and ML Data Platform teams.

[AI Platform with Python (PDF)](/pdfs/AI%20Platform%20with%20Python.pdf)

<iframe src="/pdfs/AI%20Platform%20with%20Python.pdf" width="100%" height="600px" style="border: none;">
    This browser does not support PDFs. Please download the PDF to view it: <a href="/pdfs/AI%20Platform%20with%20Python.pdf">Download PDF</a>
</iframe>

## Table of Contents

### Part 1: ML Infrastructure with Python (by Park JunSeong)

1. **Service Growth with AI Models**
   - Service Growth in 2024
   - Increasing Number of Training Pipelines

2. **Operating ML Infrastructure**
   - Limited Team Capacity Challenges
   - Managing Training Pipelines with Kubeflow + TFX
   - Configuration with Protobuf
   - Migration to GCP Vertex AI Pipelines
   - Monitoring and Alerting System
   - Internal SDK Development

3. **More Time, More Projects**
   - LLM Router
   - Prompt Studio
   - Custom Builds
   - What's Next

### Part 2: Inference Pipeline with Apache Beam Python (by Byun Kyuhyun)

1. **The Shift to Embedding-based Systems**
   - Why We Need Embedding Data
   - How Embeddings Change Data Handling
   - From Traditional Features to ANN-based Recommendations

2. **Story of the Inference Pipeline**
   - Product Requirements
   - Solution Candidates
   - Introduction to Apache Beam
   - Introduction to Google Cloud Dataflow
   - Pipeline Execution Flow
   - Code Architecture

3. **Practical Performance Tips**
   - Diagnosing Network-Bound Stages
   - GPU Memory Overload Solutions
   - Pipeline Consolidation for Cost Efficiency

---

# Part 1: ML Infrastructure with Python

*By Park JunSeong (박준성), ML Infrastructure Team*

## Background: Karrot's Explosive Growth in 2024

### Service Growth Metrics

In 2024, Karrot achieved remarkable growth:
- **3.8x YoY Operating Profit Growth**
- **189.1 billion KRW in revenue** (up from 127 billion in 2023)
- **43 million ARU** (Annual Registered Users), with **14 million WAU**
- **Advertising revenue up 48%**
- **Service expansion** to Canada, the United States, the United Kingdom, and Japan

[Source: Karrot 2024 Annual Results](https://about.daangn.com/company/pr/archive/당근-2024년-매출-1891억-원-영업이익-376억-원-기록/)

### Training Pipeline Explosion

Along with this business growth, our ML training pipelines grew dramatically:
- **Daily pipeline executions increased by 3-4x**
- From hundreds to **thousands of daily pipeline runs**
- This surge directly correlated with our operating profit growth

---

## The Challenge: Growing Fast with Limited Team Capacity

### Team Capacity Reality

Here's what made 2024 particularly challenging:
- Started the year with **3 team members**
- In **June 2025** (during peak pipeline growth), one team member left
- At our peak demand period, we were down to **2 engineers**
- Training pipelines were growing exponentially, but our team was shrinking

As one team member put it: *"Like... We're in the endgame now."* (Referencing the famous Avengers scene)

### The Mission

With limited resources, we needed to find efficient ways to:
1. **Maintain stability** of existing infrastructure
2. **Support rapid experimentation** by ML teams
3. **Minimize operational burden** on our small team
4. **Enable self-service** for ML engineers

---

## Solution 1: Managing Training Pipelines with Kubeflow + TFX

### Why Kubeflow Pipelines?

We chose Kubeflow as our ML pipeline orchestrator for several reasons:

**Kubeflow Pipelines Features:**
- ML Pipeline Orchestrator based on **Argo Workflow**
- **Extensibility and flexibility with Python** - crucial for our ML teams
- **Reusable components** - write once, use everywhere
- **Supports TensorFlow, PyTorch, XGBoost**, and more

The key advantages we experienced:
1. **Component Reusability** - ML pipeline components can be shared across teams
2. **Python-based Flexibility** - High degree of freedom in component development
3. **Kubernetes-native** - Runs on our existing K8s infrastructure

**However**, this high degree of freedom came with risks:
- Inconsistent implementation styles across teams
- Unpredictable issues in areas outside our coverage
- Debugging became difficult with diverse code patterns

### Adding TFX (TensorFlow Extended)

To mitigate these risks while maintaining Python's ML ecosystem, we adopted **TFX**:

**TFX Benefits:**
- **End-to-end platform** for ML pipelines
- **Comprehensive component set** for various ML workflow stages
- **ML Metadata support** for Kubeflow
- **Apache Beam integration** for distributed data processing and scalable workloads

**Key Advantages:**
- **Proven structure and interfaces** reduce implementation risks
- **Code consistency and reusability** through standardized patterns
- **Predictable debugging scope** - errors occur in known areas
- **Easy integration** with BigQuery and Dataflow (Google ecosystem)

### Training Pipeline Architecture

Our training pipeline consists of three main stages:

#### 1. Data Processing (Blue Section)
- **BigQuery Data Ingestion** - SQL-based data extraction
- **Data Validation** - Ensure data quality with TFX validators
- **Transform** - Convert data into training-ready format
- **Statistics Visualization** - Understand data distributions

**Usage Patterns:**
- **Experimentation**: Run data processing with each training iteration (frequent feature changes)
- **Production**: Pre-ingest data into feature platform, reuse across multiple pipelines

#### 2. Model Training (Green Section)
- **Trainer** - Execute model training with TensorFlow/PyTorch
- **Evaluator** - Validate model performance
- **Evaluation Visualization** - Visualize metrics and results

#### 3. Model Deployment (Red Section)
- **Pusher** - Upload trained model to cloud storage (GCS/S3)
- **Inference Service Restart** - Deploy new model to serving infrastructure
- **Model Card Generator** - Document model metadata
- **Metadata Pusher** - Store ML metadata for tracking

### Custom Components Extension

While TFX provides the baseline, we can easily add custom components:
- **Anomaly Checker** - Detect data anomalies before training
- **Optimized BigQuery ExampleGen** - Faster data ingestion
- **Kontrol Pusher** - Custom deployment to our serving infrastructure
- **Evaluation Publisher** - Share results with stakeholders

The beauty of this approach: **ML engineers can trust and utilize components** without worrying about underlying implementation details.

---

## Solution 2: Configuration with Protobuf

### The Dynamic Typing Challenge

Python's dynamic typing is great for rapid prototyping but problematic at scale:
- Type-related bugs appear at runtime
- Data integrity issues are hard to catch early
- Uncertainty about what values are valid

Popular solutions in the community:
- **FastAPI** uses Pydantic for request validation
- **Flask** uses Marshmallow for serialization

### Why Protobuf for ML Pipelines?

We adopted **Protocol Buffers (Protobuf)** for configuration management:

**Advantages:**
1. **Type Safety** - Prevent type-related bugs and runtime errors from dynamic typing
2. **Built-in Validation** - Eliminates additional checking code
3. **Self-Documenting** - Configuration behavior understood through Protobuf specs without code analysis
4. **Backward Compatibility** - Field addition/deletion on schema changes
5. **Human Readable** - .pbtxt format is easy to read and edit

### Example: Sampling Method Configuration

**Before (YAML/TOML) - The Problem:**
```yaml
# config.yaml
sampling_method:
  type: "filter_duplicate"
  params:
    threshold: 0.8
    enable_caching: true

# or
sampling_method:
  type: "filter_uniform_random"
  sample_rate: 0.3
  seed: 42
```

**Questions arise:**
- 👀 What types are available for `sampling_method`?
- What parameters does each type support?
- Which parameters are required vs optional?

To answer these, you'd need to **read the implementation code** every time.

**After (Protobuf) - The Solution:**
```protobuf
message FilterDuplicate {
  repeated string identifiers = 1;
  bool keep_first = 2;
  bool enable_caching = 3;
}

message FilterUniformRandom {
  float sample_rate = 1;
  int32 seed = 2;
}

message SamplingMethod {
  oneof method {
    FilterDuplicate filter_duplicate = 1;
    FilterUniformRandom filter_uniform_random = 2;
    // ... Other filters
  }
}
```

**Benefits:**
- **All filtering method types are in the schema** - no code reading needed
- **Required fields are explicit** - `sample_rate` is required for uniform random
- **IDE support** - Autocomplete and validation through Python classes
- **Minimal validation logic** - Protobuf handles it

### Validation Example

With Protobuf, validation becomes elegant:

```python
def get_sampling_config(config):
    method_type = config.get("sampling_method", {}).get("type")

    if method_type == "filter_duplicate":
        if "threshold" not in config["sampling_method"]["params"]:
            raise ValueError("threshold required for filter_duplicate")
        if not isinstance(config["sampling_method"]["params"]["threshold"], float):
            raise TypeError("threshold must be float")

    elif method_type == "filter_uniform_random":
        if "sample_rate" not in config["sampling_method"]:
            raise ValueError("sample_rate required for filter_uniform_random")
        rate = config["sampling_method"]["sample_rate"]
        if not 0.0 <= rate <= 1.0:
            raise ValueError("sample_rate must be between 0.0 and 1.0")

    elif method_type == "filter_future_context":
        # More options...
        pass
    else:
        raise ValueError(f"Unknown sampling method: {method_type}")
```

This becomes just:
```python
# Protobuf handles all validation automatically!
config = SamplingMethod()
text_format.Parse(config_text, config)
```

### Real-World Impact: Experiments with Protobuf

Our experience using Protobuf in production:

**Benefits Realized:**
1. **Reliable Development** - Protobuf specifications reduce runtime errors significantly
2. **Single Repository Collaboration** - Enables reusability and knowledge sharing across teams
3. **Accelerated Iteration Cycles** - Faster experimentation and deployment with confidence
4. **Reduced ML Infra Operational Burden** - Standardized patterns require less maintenance

**Repository Structure:**
```
config/
├── env/                    # Environment configs
├── pipeline/              # Pipeline configurations
    ├── ads_conversion/
    ├── ads_conversion_coefficient/
    ├── community_feed_ranking/
    ├── home_feed_ranking/  ← Each pipeline has its own config
        ├── baseline/
            ├── deploy.pbtxt
            ├── model.pbtxt
            ├── model_card.md
            ├── pipeline.pbtxt
            └── schema.pbtxt
    ├── baseline_ingestion/
    └── distance-only/
```

**Key Insight:** Similar pipeline configurations in one repository means:
- **Knowledge sharing** - ML engineers learn from each other's configurations
- **Easy onboarding** - New team members understand one structure, apply everywhere
- **Confident experimentation** - Structure prevents bugs, enables faster iteration
- **Safe production deployment** - Validation catches issues before runtime

As pipelines grew (3-4x increase), our **operational burden didn't scale linearly** thanks to this stability.

---

## Solution 3: Migration to GCP Vertex AI Pipelines

### Remaining Infrastructure Challenges

While Protobuf and TFX solved user-code stability, we still faced infrastructure issues:

**Operational Burden:**
- Autoscaler configuration and tuning
- Monitoring setup and maintenance
- Alert management and on-call rotation
- Kubernetes cluster problems (upgrades, networking, etc.)
- Kubeflow-specific issues
- Quota management across clusters
- Network failures and debugging

### The Alert Problem

Before better systems, our alerting was chaotic:

**Previous State:**
- All pipeline failures triggered **ML Infra team callouts**
- User code errors? → ML Infra paged
- Cluster issues? → ML Infra paged
- Network problems? → ML Infra paged
- **Limited development time** because of constant firefighting

**Root Cause:**
- Had to manually collect Pod logs from Kubernetes
- No automatic categorization of error types
- Couldn't distinguish user code errors from infrastructure errors
- With limited team capacity, we couldn't build proper log classification

### Enter Vertex AI Pipelines

We discovered **GCP Vertex AI Pipelines** - a serverless solution:

**Key Features:**
- **Serverless ML Workflows** - No cluster management needed
- **Supports Kubeflow Pipelines and TFX** - Easy migration from our existing setup
- **Reduced ML Infra callouts** on pipeline failures
- **Eliminates operational burden:**
  - No cluster management & upgrades
  - Auto-scaling within quotas
  - **Easy differentiation** between user code errors and infrastructure errors

**Migration Benefits:**
- **Automatic error categorization** through GCP Cloud Logging labels
- **User code errors** → Route to pipeline owner
- **Infrastructure errors** → Route to ML Infra team (much less frequent)
- **Reduced on-call burden dramatically**

**But monitoring and alerting still needed work...**

---

## Solution 4: Python-based Monitoring and Alerting

### Building Custom Alert System

GCP provided some tools, but we needed more:

**What GCP Offered:**
- Log-based filtering and collection
- Vertex AI Pipelines metrics
- Basic alerting through GCP Monitoring

**What We Built (Python-based):**

**Architecture:**
1. **Log Collection** - Collect ML pipeline logs from multiple GCP projects via log-based filtering and Vertex AI Pipelines metrics
2. **Alert Policies** - Create alert policies from collected logs
3. **Slack Integration** - Generate Slack alerts via GCP Cloud Run using Alert Policies and ML Metadata
4. **Auto-mention** - Identify responsible parties and users through SDK and user group lists

**Why Python?**
- Easy integration with GCP APIs
- ML Metadata manipulation for context enrichment
- Rapid development and iteration
- Team expertise (we're a Python-first team)

### Alert Evolution

**Before (Basic GCP Alerts):**
```
❌ Pipeline Failed
Pipeline: home_feed_ranking_baseline_20240625
Status: FAILED
```

**After (Enriched Python Alerts):**
```
🚨 base-example-pipeline-johan-test-20241219022924 Failed

Pipeline State: PIPELINE_STATE_FAILED
Project: ml-training-prod

Create Time: 2024-12-19 14:38:50
Assignee: @Johan (요한)

Error:
The DAG failed because some tasks failed. The failed tasks are:
[ImportSchemaGen, BigQueryExampleGen].

Job (project_id = ..., job_id = ...) is failed
due to the above error.
Failed to handle the job: {project_number = ..., job_id = ...}

Updated at 2024-12-19 14:38:50
🔗 [Pipeline Page Link]

---
BigQueryExampleGen 컴포넌트의 에러 로그 확인해주세요.
(로그는 최근 150줄 제한이 있습니다)

[Error Log showing BigQuery ExampleGen CUDA registration error details...]
```

**Key Improvements:**
- **Rich context** - Project, create time, assignee
- **Specific errors** - Which components failed
- **Auto-mention** - Directly notify responsible engineer (@Johan)
- **Actionable info** - Error logs, links to pipeline UI
- **Korean language support** - For our local team

**Result:**
- **Pipeline owners self-diagnose** issues
- **ML Infra team rarely involved** unless infrastructure problem
- **Faster resolution** - Right person notified immediately

---

## Solution 5: Internal SDK for Reusability

### SDK Strategy

We packaged frequently used pipeline elements as an SDK:

**Contents:**
- **TFX custom components** - Our extensions to standard TFX
- **ML pipeline utilities** - Common helper functions
- **Shared business logic** - Cross-team functionality

**Version Management:**
- **CalVer versioning** (`YYYY.MM.DD.timestamp`) - Time-based releases
- **`.dev` suffix** for development versions
- **Modern Python packaging:**
  - `pyproject.toml` instead of `setup.py`
  - `uv` package manager for fast, reliable installs

**Directory Structure:**
```
daangn/
├── kfp_addons/           # Kubeflow Pipelines add-ons
├── tf_addons/            # TensorFlow add-ons
├── tfrs_addons/layers/   # TensorFlow Recommenders add-ons
├── tfx_addons/           # TFX custom components
├── vertex_ai_addons/     # Vertex AI utilities
└── utils/                # Common utilities
```

### Multi-Cloud Deployment

**Package Distribution:**
- **GCP Artifact Registry** - For GCP-based pipelines
- **AWS CodeArtifact** - For AWS-based pipelines
- **Single source, multi-destination** - CI/CD handles deployment

**Benefits:**
- Engineers just run `pip install` from their environment
- Automatic selection of appropriate registry
- Version consistency across clouds

### Central Configuration with Central Dogma

**Problem:** How do you update configurations without requiring SDK version upgrades?

**Solution:** Central Dogma integration
- **Operational configurations** managed through Central Dogma (LINE's configuration management system)
- **SDK fetches configs at runtime** from Central Dogma
- **No redeployment needed** for config changes

**Example Use Cases:**
- Update resource quotas across all pipelines
- Change default ML metadata settings
- Adjust retry policies and timeouts
- Update service endpoints

**Result:**
- **Immediate config propagation** - No waiting for SDK updates
- **Operational flexibility** - ML Infra team can adjust settings quickly
- **Reduced deployment burden** - Users don't need to update SDK versions frequently

---

## More Time, More Projects

With efficient pipeline operations established, we gained time for new initiatives:

### 1. LLM Router

**Problem:** Multiple teams need different LLM models (self-hosted and external APIs)

**Solution:** Internal LLM Router (like Open Router, but internal)

**Features:**
- **vLLM-based serving** for self-hosted models (including GPT-120B after open-source release)
- **External API routing** - OpenAI, Anthropic, Google, etc.
- **Unified interface** - OpenAI-compatible API for all models
- **Cost tracking** - Monitor usage, token counts, and costs per team
- **Transparent model selection** - Users don't need to know where models are hosted

**Benefits:**
- **Easy model switching** - Change endpoints without code changes
- **Cost optimization** - Visibility into usage patterns
- **Operational efficiency** - Single system to maintain

### 2. Prompt Studio

**Built on top of LLM Router** to manage prompts for production:

**Features:**
- **Dynamic template support** - Variables and placeholders in prompts
- **MCP Agent configuration** - Access internal data sources
- **BigQuery integration** - Import test datasets for evaluation
- **Batch testing** - Validate prompts against large test sets at once
- **API monitoring** - Track prompt performance in production
- **Version management** - Easy rollback to previous prompt versions
- **AI Assistant** - Help with prompt writing and optimization

**Workflow:**
1. Design prompt with template
2. Configure MCP agents for data access
3. Test with BigQuery datasets
4. Deploy to production via API
5. Monitor performance
6. Iterate or rollback as needed

**Result:** Prompt Studio + LLM Router = **Powerful hub for LLM experimentation** that easily connects to production

### 3. Custom Builds

To support internal needs, we maintain custom builds of key libraries:

**TensorFlow IO:**
- S3 Native integration with `aws-sdk-cpp`
- Custom file system implementations

**TensorFlow Serving:**
- TensorFlow Runtime support
- ARM architecture compatibility patches
- S3 file system integration

**ScaNN (Scalable Nearest Neighbors):**
- gcc-10 compatibility patches
- Supports TensorFlow Serving 2.17 version

**Optimized TFX Components:**
- Optimized components for Apache Beam
- Performance improvements for our use cases

**Why Custom Builds?**
- **Missing features** - Add S3 support where it doesn't exist
- **Performance** - Optimization runtimes for faster inference
- **Architecture support** - ARM compatibility for cost savings
- **Bug fixes** - Don't wait for upstream releases

---

# Part 2: Inference Pipeline with Apache Beam Python

*By Byun Kyuhyun (변규현), ML Data Platform Team / AWS Serverless HERO*



## 1. The Shift to Embedding-based Systems

### Why We Need Embedding Data

Previously, our recommendations were driven by a keyword-based approach. The system relied on:
- Manually engineered features (category IDs, keyword tags, numerical scores)
- Rule-based filtering or exact matching in structured fields
- Similarity limited to predefined attributes (same category, matching keywords)

With Large Language Models (LLMs), we now generate embeddings for data representation, enabling semantic understanding rather than just keyword matching.

### How Embeddings Change Data Handling

The transformation includes:
- **From keyword/text matching to vector-based matching**
- **Flexible representation** across different modalities
- **Cross-domain usage** for search, recommendation, and classification tasks

### From Traditional Features to ANN-based Recommendations

**After (with Embeddings + ANN):**
- Each item is represented as a **dense vector embedding**, capturing semantic meaning from content, images, or user interactions
- **Approximate Nearest Neighbor (ANN) search** finds items closest in vector space
- Enables recommendations based on **semantic similarity** (visually similar, conceptually related), even when metadata doesn't match exactly

**Capabilities Unlocked:**
- Multi-modal recommendations (text, image, audio, behavior data)
- Cold-start scenario support through embedding similarity
- More flexible and scalable than manual feature engineering

---

## 2. Story of the Inference Pipeline

### Product Requirements

Our key requirements were:
1. **Process billions of records within hours**
2. **GPU-powered inference** with various embedding models
3. **Dynamic scaling** based on data volume
4. **Develop in Python**
5. **Minimal infrastructure management**
6. **Utilize BigQuery datasets and GCS images**

### The Problem with Separate Inference Servers

The old approach had challenges:
- Additional operational overhead from scaling inference servers directly
- Inference servers are auto-scaling, but response doesn't happen immediately when pipeline requests increase
- Sudden large traffic spikes can prevent proper scaling

### Solution Candidates

We evaluated three options:

| Criteria | Beam+Dataflow | Spark+DataProc | Flink |
|----------|---------------|----------------|-------|
| Large-scale batch support | **Fully auto** | Configure algorithm factors | Streaming Focus |
| GPU usage | **Custom container** | Native GPU | Limited GPU |
| Python Support | **Beam SDK** | Pyspark | Limited PyFlink |
| Infra management | **Serverless** | Cluster | Cluster + Complex Config |
| GCP integration | **Native BQ/GCS** | SDK support | Extra setup |

**Winner: Apache Beam + Google Cloud Dataflow**

### Introduction to Apache Beam

Apache Beam provides:
- **Unified programming model** for batch and streaming data processing
- **Write once, run anywhere** - works on Google Dataflow, Apache Spark, Flink
- **Multiple language support** - Python, Java, Go
- **Portable, scalable**, and integrates well with cloud services

### Introduction to Google Cloud Dataflow

Google Cloud Dataflow is:
- **Fully managed, serverless** data processing service on Google Cloud
- Runs Apache Beam pipelines for both batch and streaming workloads
- **Automatically handles** resource provisioning, scaling, and optimization
- **Seamlessly integrates** with BigQuery, Cloud Storage, Pub/Sub, and more
- Supports multiple languages via Apache Beam SDK (Python, Java, Go)

### Pipeline Execution Flow of Dataflow

The end-to-end execution path:

1. **Development**
   - Build & push custom container (Docker, dependencies)

2. **Job Submission**
   - Submit job to Dataflow (GPU, config)

3. **Google Cloud Infra**
   - Provision GPU-enabled VMs & run containers

4. **Pipeline Execution**
   - Run Beam SDK Harness & ML inference engine
   - Read/Write: BigQuery, GCS, Pub/Sub, Kafka

### Code Architecture

Our pipeline follows a modular pattern:

```
├── client/          # External service clients (GCS, Redis, BigPicture)
├── inputfilter/     # Data source filtering and validation
├── outputconverter/ # Prediction result format conversion
├── pipelines/       # Actual pipeline implementations
├── postprocessor/   # Post-output processing logic
├── predictor/       # ML model prediction execution
├── preprocessor/    # Data preprocessing and refinement
├── record/          # Data model definitions
├── scheme/          # BigQuery schema definitions
├── sink/            # Data output destinations
├── source/          # Data input sources
└── util/            # Common utilities
```

**Pipeline Flow:**
```
Data Sources (Kafka/PubSub/BigQuery)
  → Input Filter
  → Preprocessor (fetch images, resize, build prompt)
  → Predictor (run embedding inference)
  → Output Converter
  → Data Sinks (Kafka/PubSub/BigQuery)
```

**Example Pipeline Code:**
```python
input_collection = pipeline | "Read from Kafka" >> ReadFromKafka(
    topics=["my_topic"],
    consumer_config={...},
)

image_processed_collection = input_collection | "Image Process" >> ParDo(ImageProcessor(...))
prompt_processed_collection = image_processed_collection | "Prompt Process" >> ParDo(PromptProcessor())
predicted_collection = prompt_processed_collection | "Predict" >> ParDo(Predictor(...))
postprocessed_collection = predicted_collection | "Postprocess" >> ParDo(Postprocessor(...))

postprocessed_collection | "Converter 1" >> ParDo(...) | "Write to BigQuery" >> WriteToBigQuery(...)
postprocessed_collection | "Converter 2" >> ParDo(...) | "Write to Kafka" >> WriteToKafka(...)
```

---

## 3. Practical Performance Tips

### Diagnosing a Network-Bound Stage

**Symptom:** Workers show low CPU usage, yet throughput remains flat. Autoscaler sees backlog growth → adds more workers → Result: More workers, small throughput, higher cost.

**Root Causes:**
- **Per-element synchronous calls** - Blocking HTTP calls stall threads
- **Low concurrency within a worker** - Limited SDK harness threads; blocking I/O kills parallelism
- **Pipeline fusion & tiny bundles** - Small bundles → low in-flight concurrency
- **External system quotas** - No pooling → QPS ceiling
- **Retry/backoff stalls** - Rate-limits + exponential backoff = long idle times
- **Network plumbing constraints** - Latency, port limits, DNS throttling, disabled keep-alive

**Fixes:**

1. **Make I/O concurrent & non-blocking:**
   - Async client + connection pool + concurrency limits
   - Batch elements before API calls

2. **Break fusion before I/O:**
   - Use `beam.Reshuffle()` to get larger bundles into the I/O stage

**Example Code:**
```python
class AsyncHTTPDoFn(beam.DoFn):
    def setup(self):
        self.sem = asyncio.Semaphore(128)
        self.client = httpx.AsyncClient(http2=True)

    async def _call_one(self, item):
        async with self.sem:
            r = await self.client.post(URL, json=item)
            return r.json()

    async def _call_batch(self, batch):
        return await asyncio.gather(*(self._call_one(it) for it in batch))

    def process(self, batch):
        yield from asyncio.run(self._call_batch(batch))

input_collection
| beam.Reshuffle()
| beam.BatchElements(min_batch_size=32, max_batch_size=256)
| beam.ParDo(AsyncHTTPDoFn())
```

### Problem: GPU Memory Overload

**The Issue:**
- Default behavior: Beam spawns 1 process per CPU core
- Each process dynamically creates worker threads
- Each thread loads the model for its step
- GPU memory is limited (~16 GB)
- Model load consumes at least 3GB per thread
- Result: **CUDA Out of Memory**

**Solution: Using Shared and MultiprocessShared**

**Shared (Thread-level):**
- Allows multiple threads within a single process to share an instance
- Reduces memory duplication for expensive objects in multi-threaded workers

**MultiprocessShared (Process-level):**
- Allows multiple processes on the same worker to share a single instance
- Greatly reduces memory footprint for large models
- Added in Beam Python 2.49.0

### Pipeline Consolidation for Cost Efficiency

**Strategy:**
- Running all pipelines separately → High baseline cost
- Identified low-traffic pipelines with underutilized resources
- Consolidated these into shared pipelines
- Reduced idle resource usage without impacting performance

---

## What's Next?

Our roadmap includes:

1. **Expand embedding-based pipelines to more products**
   - Deploy the current embedding-powered architecture beyond the initial use case
   - Enable search, recommendations, and personalization features across multiple services

2. **Improve customer experience with more models**
   - Integrate additional ML/LLM models to enhance relevance, accuracy, and responsiveness
   - Focus on multi-modal support (text, image, and video) for richer user interactions

---

## About the Authors

**박준성 (Park JunSeong)**
- ML Infrastructure Team, Software Engineer
- LinkedIn: [linkedin.com/in/johan-park/](https://linkedin.com/in/johan-park/)
- GitHub: [github.com/Writtic](https://github.com/Writtic)

**변규현 (Byun Kyuhyun)**
- ML Data Platform Team, Software Engineer
- AWS Serverless HERO
- LinkedIn: [linkedin.com/in/novemberde/](https://linkedin.com/in/novemberde/)
- Blog: [novemberde.github.io](https://novemberde.github.io)

---

## Conclusion

Building a scalable inference pipeline for embedding-based recommendations required careful consideration of:
- **Architecture**: Moving from separate inference servers to integrated pipelines
- **Technology**: Leveraging Apache Beam + Google Cloud Dataflow for serverless, scalable processing
- **Performance**: Addressing network bottlenecks and GPU memory constraints
- **Cost**: Consolidating low-traffic pipelines to reduce baseline costs

The result is a robust system capable of processing billions of records within hours, supporting Karrot's rapid growth and AI-powered features.
