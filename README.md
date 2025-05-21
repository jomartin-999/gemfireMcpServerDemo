# GemFireMCPServer

A Spring Boot–based Model Context Protocol (MCP) server that uses VMware GemFire and Spring AI to enable fast, vector-based search over financial documents.

## Overview

This project demonstrates how an MCP client (like Claude Desktop) can interact with a GemFire-backed server to perform semantic search over documents. Financial files are chunked, embedded into vector representations, and stored in GemFire to support fast, intelligent querying using a Retrieval-Augmented Generation (RAG) workflow.

### Users can:

- **Upload documents** — Files are chunked, embedded, and stored as vectors in GemFire. Metadata (such as file name and size) is stored separately in a dedicated GemFire region.
- **Browse uploaded content** — Retrieve and view metadata for all stored documents.
- **Query documents (RAG)** — Submit a natural language question from an MCP client. The server performs a vector similarity search and passes the top results to the language model for response generation.

## Features

- 🔗 MCP-compliant endpoints for document ingestion, listing, and querying
- 🧠 Local, ONNX-based embeddings with Spring AI
- 💾 Vector and metadata storage in VMware GemFire
- 🧾 File metadata stored in a dedicated GemFire region
- 🔍 Fast, in-memory semantic search

---

## Getting Started

### Prerequisites

- Java 17+
- Gradle (recommended for this demo) or Maven
- A running [VMware GemFire cluster](https://techdocs.broadcom.com/us/en/vmware-tanzu/data-solutions/tanzu-gemfire/10-1/gf/getting_started-15_minute_quickstart_gfsh.html)
- Claude Desktop (or an MCP client)

---

### Setting Up Embedding with ONNX
To generate embeddings for PDF documents, Spring AI requires an embedding model. This demo uses the ONNX-exported version of `sentence-transformers/all-MiniLM-L6-v2` for local inference.

Refer to [Spring AI’s ONNX documentation](https://docs.spring.io/spring-ai/reference/api/embeddings/onnx.html#_prerequisites) for details, or follow the local setup below:

#### Steps

1. Create a virtual environment:
```
python3 -m venv venv
```
2. Activate it:
```
source ./venv/bin/activate
```
3. Upgrade pip and install build tools:
```
pip install --upgrade pip setuptools wheel
```
4. Install required packages:
```
pip install optimum onnx onnxruntime sentence-transformers
```
6. Export the model:
```
optimum-cli export onnx --model sentence-transformers/all-MiniLM-L6-v2 onnx-output-folder
```

This will generate a `model.onnx` and `tokenizer.json` in the `onnx-output-folder`.

### Potential issues 
#### ONNX Export: Missing Dependency

If you see the following error during ONNX export:

```
Weight deduplication check in the ONNX export requires accelerate. Please install accelerate to run it.
```

Install the missing dependency:

```bash
pip install accelerate
```
---

## Clone and Build the Project
### Clone the Repository

```bash
git clone https://github.com/your-org/GemFireMCPServer.git
cd GemFireMCPServer
```

### Configure GemFire Artifact Access
To resolve GemFire dependencies, you’ll need access credentials from Broadcom.

1. Log in to the [Broadcom Customer Support Portal](https://support.broadcom.com/) with your customer credentials. 
2. Go to the [VMware Tanzu GemFire](https://support.broadcom.com/group/ecx/productdownloads?subfamily=VMware%20Tanzu%20GemFire) downloads page, select VMware Tanzu GemFire, click Show All Releases.
3. Find the release named **Click Green Token for Repository Access** and click the **Token Download icon on the RIGHT**. This opens the instructions on how to use the GemFire artifact repository. At the top, the Access Token is provided. Click **Copy to Clipboard**. You will use this Access Token as the password. 

#### If using Gradle
Add credentials to your ~/.gradle/gradle.properties (or project-level gradle.properties):
```
gemfireRepoUsername=EXAMPLE-USERNAME
gemfireRepoPassword=MY-PASSWORD
```
- `EXAMPLE-USERNAME` is your support.broadcom.com username.
- `MY-PASSWORD` is the Access Token you copied in step 3.

#### If using Maven:
Update your `~/.m2/settings.xml`:
```
<settings>
  <servers>
    <server>
      <id>gemfire-release-repo</id>
      <username>EXAMPLE-USERNAME</username>
      <password>MY-PASSWORD</password>
    </server>
  </servers>
</settings>
```
- `EXAMPLE-USERNAME` is your support.broadcom.com username.
- `MY-PASSWORD` is the Access Token you copied in step 3.
    

### Configure `application.properties`

Open the `application.properties` file and configure the following properties:

* `spring.main.web-application-type=none` — This example uses the STDIO connection with the MCP Client (Claude Desktop), so we must disable the embedded web server as it doesn't require handling HTTP requests.
* `spring.main.banner-mode=off` — Disables the application banner displayed on startup.
* `logging.pattern.console=` — Disables formatted log output to reduce console noise.
* `spring.ai.mcp.server.stdio=true` — Enables the MCP STDIO server.
* `spring.ai.vectorstore.gemfire.port=7070` — Port used by the GemFire HTTP service; must match the `--http-service-port` used by your cluster.
* `spring.ai.vectorstore.gemfire.initialize-schema=true` — Automatically creates the vector index and region in GemFire. Set to `false` if you're managing the index manually.
* `spring.ai.vectorstore.gemfire.index-name=financialDocuments` — Name of the vector index in GemFire.
* `spring.ai.embedding.transformer.onnx.model-uri=classpath:onnx/model.onnx` — Path to the ONNX model file created earlier.
* `spring.ai.embedding.transformer.tokenizer.uri=classpath:onnx/tokenizer.json` — Path to the tokenizer file created earlier.
* `spring.ai.embedding.transformer.onnx.modelOutputName=token_embeddings` — Output node name used for embeddings.
* `docs.path=/Path/To/FinancialDocs` — Directory to load financial documents from.

###  Build the MCP Server

```
./gradlew clean build
```


