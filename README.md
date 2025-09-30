# Synthik SDK Client Documentation

Welcome to the official documentation for the Synthik SDK\! This repository provides powerful, well-typed clients for both **Python** and **TypeScript** to seamlessly integrate with the Synthik Labs API for synthetic data generation.

This document is structured to guide you from installation and authentication to advanced usage for each SDK.

-----

## Python SDK 🐍

A thin, well-typed Python client for the Synthik Labs API.

### 1\. Installation

Install the client from PyPI using pip:

```bash
pip install synthik-client
```

### 2\. Available Imports

The `synthik` package exposes the following classes for direct import:

```python
from synthik import (
    SynthikClient,
    ColumnBuilder,
    DatasetGenerationRequest,
    TextDatasetGenerationRequest,
    ColumnDescription
)
```

  * `SynthikClient`: The main entry point for interacting with the API.
  * `ColumnBuilder`: A fluent helper for defining columns with specific types and constraints.
  * `DatasetGenerationRequest`: The request object for generating tabular data.
  * `TextDatasetGenerationRequest`: The request object for generating text-based data.
  * `ColumnDescription`: The underlying data class for a column definition.

### 3\. Initialization and Configuration

First, import and instantiate the client. It is highly recommended to specify `api_version="v2"` to use the latest features, as v1 is deprecated.

```python
from synthik import SynthikClient

# Initialize the client for API v2
client = SynthikClient(api_version="v2")

# If you already have an API key, you can provide it directly
client = SynthikClient(api_version="v2", api_key="YOUR_API_KEY")
```

### 4\. Authentication

The `client.auth` module handles user registration, login, and token management. The typical flow is to register an account and then log in to obtain an API key (token) for subsequent requests.

```python
# Step 1: Register a new user (only needs to be done once)
try:
    client.auth.register("user@example.com", "a-strong-password")
    print("Registration successful!")
except Exception as e:
    print(f"Registration failed: {e}")

# Step 2: Log in to get an API token
try:
    auth_response = client.auth.login("user@example.com", "a-strong-password")
    api_token = auth_response.get("token")
    print(f"Successfully logged in. API Token: {api_token[:10]}...")

    # Step 3: Use this token to initialize a new client for data generation
    authed_client = SynthikClient(api_version="v2", api_key=api_token)
except Exception as e:
    print(f"Login failed: {e}")
```

### 5\. Usage Examples

Once your client is initialized with a valid API key, you can generate synthetic data.

#### Tabular Data Generation

```python
from synthik.types import ColumnBuilder, DatasetGenerationRequest

# Assumes 'authed_client' is an authenticated client instance
tabular_req = DatasetGenerationRequest(
    num_rows=100,
    topic="User profiles for an e-commerce platform",
    columns=[
        ColumnBuilder.string("full_name", description="User's full name").build(),
        ColumnBuilder.int("age", description="Age in years", constraints={"min": 18, "max": 90}).build(),
        ColumnBuilder.email().build(),
    ],
    seed=42,
)

tabular_json = authed_client.tabular.generate(tabular_req, format="json")
print(tabular_json["metadata"])
```

#### Text Data Generation

```python
from synthik.types import TextDatasetGenerationRequest

# Assumes 'authed_client' is an authenticated client instance
text_req = TextDatasetGenerationRequest(
    num_samples=10,
    task_definition="sentiment analysis",
    data_domain="e-commerce",
    data_description="product reviews",
    output_format="instruction",
    sample_examples=[{"instruction": "Classify sentiment", "input": "Love it", "output": "positive"}],
)
text_dataset = authed_client.text.generate(text_req)
print(text_dataset.metadata)
```

### 6\. API Reference

#### Auth Module

| Method | Description |
| :--- | :--- |
| `register(email, password)` | Create a new user account. |
| `login(email, password)` | Authenticate and receive an API token. |
| `validate_token(token)` | Check if a given token is valid. |
| `list_tokens(include_revoked?, include_expired?)`| List all tokens for the current user. |
| `revoke(token)` | Revoke a token by its value. |
| `revoke_by_id(token_id)` | Revoke a token by its numeric ID. |
| `me()` | Retrieve the profile of the current user. |

#### Tabular Module

| Method | Description |
| :--- | :--- |
| `generate(req, ...)` | Generates data. Formats: `json`, `csv`, `parquet`, `arrow`, `excel`. |
| `strategies()` | Lists available data generation strategies. |
| `analyze(req)` | Performs a pre-flight analysis of your generation request. |
| `validate(data, columns)` | Validates that data conforms to a given schema. |

#### Text Module

| Method | Description |
| :--- | :--- |
| `generate(req)` | Generates a dataset of text samples. |
| `info()` | Retrieves capability information about the text models. |
| `validate(req)` | Validates a text generation request without generating. |
| `examples()` | Returns a list of sample tasks to help you get started. |

-----

## TypeScript SDK 📜

A minimal, fully-typed TypeScript client for the Synthik Labs backend.

### 1\. Installation

Install the client from npm:

```bash
npm install @synthik/client
```

### 2\. Available Imports

The `@synthik/client` package provides a default export for the client, named exports for builders, and type exports for all request/response objects.

```ts
import SynthikClient, {
  ColumnBuilder,
  type DatasetGenerationRequest,
  type TextDatasetGenerationRequest,
  type ColumnDescription,
  type TabularExportFormat
} from "@synthik/client";
```

  * `SynthikClient`: The main client class (default export).
  * `ColumnBuilder`: A fluent helper for defining data schemas.
  * `DatasetGenerationRequest`, `TextDatasetGenerationRequest`, etc.: TypeScript types for full type-safety when building requests.

### 3\. Initialization and Configuration

Import and instantiate the client. Set `apiVersion: 'v2'` in the options to use the latest API, as v1 is deprecated and will trigger a console warning.

```ts
import SynthikClient from "@synthik/client";

// Initialize the client for API v2
const client = new SynthikClient({ apiVersion: 'v2' });

// If you already have an API key, provide it directly
const client = new SynthikClient({ apiVersion: 'v2', apiKey: 'YOUR_API_KEY' });
```

### 4\. Authentication

The `client.auth` module handles user registration and login. You can use these methods to get an API key that you can then use to initialize an authenticated client instance.

```ts
async function getAuthenticatedClient(): Promise<SynthikClient> {
  const client = new SynthikClient({ apiVersion: 'v2' });

  try {
    // Step 1: Register (optional, only needs to be done once)
    await client.auth.register("user@example.com", "a-strong-password");
    console.log("Registration successful!");
  } catch (e) {
    console.error("Registration might have failed (or user already exists)", e);
  }

  // Step 2: Log in to get a token
  const authResponse = await client.auth.login("user@example.com", "a-strong-password");
  const apiToken = authResponse.token;
  console.log("Login successful!");

  // Step 3: Return a new client instance configured with the API token
  return new SynthikClient({ apiVersion: 'v2', apiKey: apiToken });
}

// Usage:
const authedClient = await getAuthenticatedClient();
```

### 5\. Usage Examples

Once you have an authenticated client, you can generate data.

#### Tabular Data Generation

```ts
import { ColumnBuilder, type DatasetGenerationRequest } from "@synthik/client";

// Assumes 'authedClient' is an authenticated client instance from the example above
const tabularReq: DatasetGenerationRequest = {
  num_rows: 50,
  topic: "User profiles",
  columns: [
    ColumnBuilder.string("full_name").build(),
    ColumnBuilder.int("age", { constraints: { min: 18, max: 90 } }).build(),
    ColumnBuilder.email().build(),
  ],
};

const tabularData = await authedClient.tabular.generate(tabularReq);
console.log(tabularData.metadata);
```

#### Text Data Generation

```ts
import { type TextDatasetGenerationRequest } from "@synthik/client";

// Assumes 'authedClient' is an authenticated client instance
const textReq: TextDatasetGenerationRequest = {
  num_samples: 5,
  task_definition: "sentiment analysis of product reviews",
  data_domain: "e-commerce",
  data_description: "Product reviews with positive/negative/neutral",
  output_format: "instruction",
  sample_examples: [
    { instruction: "Analyze sentiment", input: "Great battery life", output: "positive" }
  ]
};
const text = await authedClient.text.generate(textReq);
console.log(text.metadata);
```

### 6\. API Reference

The following tables provide a quick reference for the methods available in each module.

#### Auth Module

| Method | Description |
| :--- | :--- |
| `register(email, password)` | Create a new user account. |
| `login(email, password)` | Authenticate and get an auth token. |
| `validateToken(token?)` | Validate the current or a provided token. |
| `listTokens(includeRevoked?, includeExpired?)` | List all tokens for the user. |
| `revoke(token)` | Revoke a token by its value. |
| `revokeById(token_id)` | Revoke a token by its numeric ID. |
| `me()` | Get the current user's profile. |

#### Tabular Module

| Method | Description |
| :--- | :--- |
| `generate(req, opts?)` | Unified generate method. Opts: `{ strategy?, format?, batch_size? }`. |
| `strategies()` | Retrieve available generation strategies. |
| `analyze(req)` | Pre-flight analysis of a generation request. |
| `validate({ data, schema })`| Validate data rows against a schema. |

#### Text Module

| Method | Description |
| :--- | :--- |
| `generate(req)` | Unified text generation method. |
| `info()` | Get model and capability information. |
| `validate(req)` | Validate a text generation request. |
| `examples()` | Retrieve sample example tasks. |
