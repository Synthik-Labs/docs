# Synthik SDK Client Documentation 

Welcome to the official Synthik SDK\! This repository provides powerful, well-typed clients for both **Python** and **TypeScript** to seamlessly integrate with the Synthik Labs API for synthetic data generation.

Whether you're working in a Python backend or a TypeScript/JavaScript project, our SDKs are designed to be intuitive, robust, and easy to use.

-----

## Python SDK 🐍

A thin, well-typed Python client for the Synthik Labs API.

### 1\. Installation

Install the client from PyPI using pip:

```bash
pip install synthik-client
```

### 2\. Client Initialization

Import and instantiate the client. The client can be configured with your API key, a custom base URL, and other options.

```python
from synthik import SynthikClient

# Initialize with default settings (API key can be set via an environment variable)
client = SynthikClient()
```

### 3\. Available Imports

The SDK exposes several key classes to help you build generation requests:

```python
from synthik import SynthikClient
from synthik.types import (
    ColumnBuilder, # A fluent helper for defining columns
    DatasetGenerationRequest, # Request object for tabular data
    TextDatasetGenerationRequest, # Request object for text data
    ColumnDescription # The underlying data class for a column
)
```

### 4\. Generating Tabular Data

To generate synthetic tabular data, create a `DatasetGenerationRequest` and use the `client.tabular.generate()` method. The **`ColumnBuilder`** provides a convenient, fluent interface for defining your schema.

#### Example: Generating User Profiles

```python
from synthik import SynthikClient
from synthik.types import ColumnBuilder, DatasetGenerationRequest, TextDatasetGenerationRequest

client = SynthikClient()

# Tabular
req = DatasetGenerationRequest(
    num_rows=100,
    topic="User profiles",
    columns=[
        ColumnBuilder.string("full_name", description="User's full name").build(),
        ColumnBuilder.int("age", description="Age in years", constraints={"min": 18, "max": 90}).build(),
        ColumnBuilder.categorical("country", ["US", "CA", "GB"]).build(),
        ColumnBuilder.email().build(),
    ]
)
result = client.tabular.generate(req)
print(result["metadata"])  # when format=json
```

### 5\. Generating Text Data

For NLP tasks, you can generate text data using `client.text.generate()`.

#### Example: Generating Product Reviews

```python
text_req = TextDatasetGenerationRequest(
    num_samples=10,
    task_definition="sentiment analysis",
    data_domain="e-commerce",
    data_description="product reviews",
    output_format="instruction",
)
text_data = client.text.generate(text_req)
print(text_data.metadata)
```

### 6\. Other Endpoints

The client also provides access to other utility endpoints:

  * `client.tabular.strategies()`: Get available generation strategies.
  * `client.tabular.analyze(req)`: Analyze a generation request.
  * `client.tabular.validate(data, columns)`: Validate data against a schema.
  * `client.text.info()`: Get information about the text generation service.

-----

## TypeScript SDK 📜

A minimal, fully-typed TypeScript client for the Synthik Labs API.

### 1\. Installation

Install the client from npm:

```bash
npm install synthik-client
# or
pnpm add synthik-client
# or
yarn add synthik-client
```

### 2\. Client Initialization

Import and instantiate the client. The base URL is pre-configured, but you can provide an API key and other options.

```ts
import SynthikClient from "synthik-client";

// Initialize with default settings
const client = new SynthikClient();
```

### 3\. Available Imports

The SDK is fully typed and exports all necessary interfaces and classes for a seamless development experience.

```ts
import SynthikClient, {
  ColumnBuilder, // A fluent helper for defining columns
  type DatasetGenerationRequest, // Type for tabular generation requests
  type TextDatasetGenerationRequest, // Type for text generation requests
  type ColumnDescription,
  type GenerationStrategy,
  type TabularExportFormat,
} from "synthik-client";
```

### 4\. Generating Tabular Data

Create a `DatasetGenerationRequest` object and pass it to the `client.tabular.generate()` method.

#### Example: Generating User Profiles

```ts
import SynthikClient, { ColumnBuilder, type DatasetGenerationRequest } from "synthik-client";

const client = new SynthikClient();

const req: DatasetGenerationRequest = {
  num_rows: 50,
  topic: "User profiles",
  columns: [
    ColumnBuilder.string("full_name", { description: "User's full name", max_length: 80 }).build(),
    ColumnBuilder.int("age", { description: "Age in years", constraints: { min: 18, max: 90 } }).build(),
    ColumnBuilder.categorical("country", ["US", "CA", "GB"]).build(),
    ColumnBuilder.email().build(),
  ],
};

// Generate the data, specifying strategy and format
const result = await client.tabular.generate(req, { strategy: "adaptive_flow", format: "json" });
console.log(result);
```

### 5\. Generating Text Data

To generate text data, construct a `TextDatasetGenerationRequest` and use the `client.text.generate()` method.

#### Example: Generating Customer Support Conversations

```ts
const text = await client.text.generate({
  num_samples: 5,
  task_definition: "sentiment analysis of product reviews",
  data_domain: "e-commerce",
  data_description: "Product reviews with positive/negative/neutral",
  output_format: "instruction",
  sample_examples: [
    {
      instruction: "Analyze the sentiment of this product review",
      input: "This phone has amazing battery life and great camera quality!",
      output: "positive",
    },
  ],
});
console.log(text);
```

### 6\. Other Endpoints

The TypeScript client also provides typed methods for other API endpoints:

  * `client.tabular.strategies()`
  * `client.tabular.analyze(request)`
  * `client.tabular.validate(body)`
  * `client.text.info()`
  * `client.text.examples()`

-----

### Best Practices for Using the SDKs

  * **Be Descriptive**: For both tabular and text generation, the quality of the synthetic data is highly dependent on the quality of your descriptions (`topic`, `description`, `task_definition`, etc.). Provide as much context as you can.
  * **Use Constraints**: When generating tabular data, leverage `constraints` (`min`, `max`, `regex`, etc.) to ensure the data fits your exact requirements.
  * **Handle Asynchronicity (TypeScript)**: All TypeScript client methods are `async` and return a `Promise`. Always use `await` or `.then()` to handle the results.
  * **Choose the Right Format**: The `tabular.generate` method supports multiple output formats (`json`, `csv`, `parquet`). Choose the one that best integrates with your data pipeline.
