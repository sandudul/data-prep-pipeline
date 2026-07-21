<div align="center">

# 🔄 Data Processor

**A modular, extensible, and reusable data processing pipeline for Python.**

[![Python](https://img.shields.io/badge/Python-3.8%2B-blue?logo=python&logoColor=white)](https://www.python.org/)
[![License](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)
[![Tests](https://img.shields.io/badge/Tests-pytest-orange?logo=pytest&logoColor=white)](https://pytest.org)
[![Version](https://img.shields.io/badge/Version-0.1.0-purple)](pyproject.toml)

Built on **Object-Oriented** principles and **Abstract Base Classes**, `data_processor` gives you a clean, consistent interface to build robust ETL (Extract → Transform → Load) pipelines — no matter where your data comes from or where it needs to go.

</div>

---

## 📋 Table of Contents

- [Overview](#-overview)
- [Architecture](#-architecture)
- [Installation](#-installation)
- [Quick Start](#-quick-start)
- [API Reference](#-api-reference)
  - [Readers](#-readers)
  - [Transformers](#-transformers)
  - [Writers](#-writers)
  - [Pipeline (Workflow)](#-pipeline-workflow)
- [Full Example](#-full-example)
- [Project Structure](#-project-structure)
- [Running Tests](#-running-tests)
- [Extending the Package](#-extending-the-package)
- [Dependencies](#-dependencies)

---

## 🧭 Overview

`data_processor` is designed around a simple but powerful philosophy:

> **Read → Transform → Write**

Each step is decoupled via abstract base classes, meaning you can freely swap out any component without touching the rest of your pipeline. Load from CSV, transform the data, and write to JSON — or swap the reader to Excel tomorrow — your transformation logic never changes.

---

## 🏗️ Architecture

The package is built around three abstract base classes defined in `core.py`:

```
DataReader      (ABC)  →  defines .read()       → returns DataType
DataTransformer (ABC)  →  defines .transform()  → returns DataType
DataWriter      (ABC)  →  defines .write()       → returns None
```

> `DataType` is a type alias for `List[Dict[str, Any]]` — a standard list of row dictionaries, making it easy to inspect, debug, and manipulate data at any stage.

The `DataPipeline` class in `workflows.py` orchestrates these three components:

```
┌──────────────┐     ┌─────────────────────────────┐     ┌──────────────┐
│  DataReader  │────▶│  DataTransformer (chained)  │────▶│  DataWriter  │
│  (source)    │     │  transform 1 → 2 → 3 → ...  │     │  (output)    │
└──────────────┘     └─────────────────────────────┘     └──────────────┘
```

---

## ⚙️ Installation

Clone the repository and install the package locally:

```bash
git clone https://github.com/your-username/data-prep-pipeline.git
cd data-prep-pipeline
```

**Standard install** (core package only):
```bash
pip install -e .
```

**Development install** (includes `pytest` for running tests):
```bash
pip install -e .[dev]
```

---

## 🚀 Quick Start

```python
from data_processor.readers import CSVReader
from data_processor.transformers import MissingValueImputer, TypeCaster
from data_processor.writers import CSVWriter
from data_processor.workflows import DataPipeline

# 1. Define source and destination
reader = CSVReader("data/input.csv")
writer = CSVWriter("data/output.csv")

# 2. Build the pipeline
pipeline = DataPipeline(reader, writer)

# 3. Chain transformers
pipeline.add_transformer(MissingValueImputer("Department", "Unknown"))
pipeline.add_transformer(TypeCaster("Salary", float))

# 4. Run it
data = pipeline.run()
print(f"✅ Done! Processed {len(data)} rows.")
```

---

## 📖 API Reference

### 📥 Readers

All readers extend `DataReader` and implement `.read() → DataType`.

---

#### `CSVReader`
Reads data from a **CSV file**.

```python
from data_processor.readers import CSVReader

reader = CSVReader(filepath="data/employees.csv", delimiter=",")
data = reader.read()
```

| Parameter   | Type  | Default | Description                |
|-------------|-------|---------|----------------------------|
| `filepath`  | `str` | —       | Path to the CSV file       |
| `delimiter` | `str` | `','`   | Column delimiter character |

---

#### `JSONReader`
Reads data from a **JSON file** (expects a top-level array of objects).

```python
from data_processor.readers import JSONReader

reader = JSONReader(filepath="data/records.json")
data = reader.read()
```

| Parameter  | Type  | Default | Description           |
|------------|-------|---------|-----------------------|
| `filepath` | `str` | —       | Path to the JSON file |

> ⚠️ The JSON file must be a **list of dictionaries** at the top level. Raises `ValueError` otherwise.

---

#### `ExcelReader`
Reads data from an **Excel `.xlsx` file**. Requires `openpyxl`.

```python
from data_processor.readers import ExcelReader

reader = ExcelReader(filepath="data/report.xlsx", sheet_name="Sheet1")
data = reader.read()
```

| Parameter    | Type            | Default | Description                                    |
|--------------|-----------------|---------|------------------------------------------------|
| `filepath`   | `str`           | —       | Path to the `.xlsx` file                       |
| `sheet_name` | `str` or `None` | `None`  | Sheet to read; uses the active sheet if `None` |

> The first row is treated as the **header row**. Any `None` header cell is auto-named `Column_N`.

---

### 🔧 Transformers

All transformers extend `DataTransformer` and implement `.transform(data) → DataType`. They are **pure** — they never mutate input data, always returning a new list.

---

#### `ColumnFilter`
Keeps only the **specified columns** and drops everything else.

```python
from data_processor.transformers import ColumnFilter

transformer = ColumnFilter(columns_to_keep=["Name", "Salary", "Department"])
data = transformer.transform(data)
```

| Parameter         | Type        | Description                    |
|-------------------|-------------|--------------------------------|
| `columns_to_keep` | `List[str]` | List of column names to retain |

---

#### `MissingValueImputer`
Replaces `None` or empty string values in a column with a **default value**.

```python
from data_processor.transformers import MissingValueImputer

transformer = MissingValueImputer(column="Department", default_value="Unknown")
data = transformer.transform(data)
```

| Parameter       | Type  | Description                             |
|-----------------|-------|-----------------------------------------|
| `column`        | `str` | The column name to check                |
| `default_value` | `Any` | The value to use when a cell is missing |

---

#### `TypeCaster`
Casts the values of a column to a **target Python type** (e.g. `int`, `float`, `str`).

```python
from data_processor.transformers import TypeCaster

transformer = TypeCaster(column="Salary", new_type=float)
data = transformer.transform(data)
```

| Parameter  | Type   | Description                                        |
|------------|--------|----------------------------------------------------|
| `column`   | `str`  | The column name to cast                            |
| `new_type` | `type` | The Python type to cast to (e.g. `int`, `float`)  |

> If the cast fails for a row, the original value is silently preserved.

---

### 📤 Writers

All writers extend `DataWriter` and implement `.write(data) → None`.

---

#### `CSVWriter`
Writes data to a **CSV file**. Column headers are inferred from the data keys.

```python
from data_processor.writers import CSVWriter

writer = CSVWriter(filepath="output/result.csv", delimiter=",")
writer.write(data)
```

| Parameter   | Type  | Default | Description               |
|-------------|-------|---------|---------------------------|
| `filepath`  | `str` | —       | Destination CSV file path |
| `delimiter` | `str` | `','`   | Column delimiter character |

---

#### `JSONWriter`
Writes data to a **JSON file** with configurable indentation.

```python
from data_processor.writers import JSONWriter

writer = JSONWriter(filepath="output/result.json", indent=4)
writer.write(data)
```

| Parameter  | Type  | Default | Description                |
|------------|-------|---------|----------------------------|
| `filepath` | `str` | —       | Destination JSON file path |
| `indent`   | `int` | `4`     | JSON indentation level     |

---

### 🔀 Pipeline (Workflow)

The `DataPipeline` class wires everything together and runs the full ETL flow.

```python
from data_processor.workflows import DataPipeline

pipeline = DataPipeline(reader=reader, writer=writer)  # writer is optional
pipeline.add_transformer(transformer_1)
pipeline.add_transformer(transformer_2)

data = pipeline.run()
```

| Method                          | Description                                                            |
|---------------------------------|------------------------------------------------------------------------|
| `__init__(reader, writer=None)` | Creates a pipeline with a reader and an optional writer                |
| `add_transformer(transformer)`  | Appends a transformer to the chain; **returns `self`** for chaining   |
| `run()`                         | Executes read → transform(s) → write; returns the final processed data |

**Transformer chaining** is supported via method chaining:

```python
pipeline \
    .add_transformer(MissingValueImputer("Dept", "N/A")) \
    .add_transformer(TypeCaster("Age", int)) \
    .add_transformer(ColumnFilter(["Name", "Age", "Dept"]))
```

---

## 💡 Full Example

Read from an Excel file, clean up missing department values, cast salaries to floats, and write the result to CSV:

```python
from data_processor.readers import ExcelReader
from data_processor.transformers import MissingValueImputer, TypeCaster
from data_processor.writers import CSVWriter
from data_processor.workflows import DataPipeline

reader = ExcelReader("dataset/Datasets.xlsx")
writer = CSVWriter("dataset/processed_dataset.csv")

pipeline = DataPipeline(reader, writer)
pipeline \
    .add_transformer(MissingValueImputer("Department", "Unknown")) \
    .add_transformer(TypeCaster("Salary", float))

processed = pipeline.run()

print(f"✅ Pipeline finished! Processed {len(processed)} rows.")
print("First 2 rows:")
for row in processed[:2]:
    print(row)
```

**Console output:**
```
Reading data...
Applying transformation 1: MissingValueImputer...
Applying transformation 2: TypeCaster...
Writing data to destination using CSVWriter...
Workflow completed.
✅ Pipeline finished! Processed 150 rows.
```

---

## 📁 Project Structure

```
data-prep-pipeline/
│
├── src/
│   └── data_processor/
│       ├── __init__.py         # Package entry point
│       ├── core.py             # Abstract base classes (DataReader, DataTransformer, DataWriter)
│       ├── readers.py          # CSVReader, JSONReader, ExcelReader
│       ├── transformers.py     # ColumnFilter, MissingValueImputer, TypeCaster
│       ├── writers.py          # CSVWriter, JSONWriter
│       └── workflows.py        # DataPipeline orchestrator
│
├── tests/
│   ├── test_readers.py         # Unit tests for readers
│   ├── test_transformers.py    # Unit tests for transformers
│   └── test_workflows.py       # Integration tests for the pipeline
│
├── dataset/                    # Sample datasets
├── test_full_pipeline.py       # End-to-end pipeline demo script
├── pyproject.toml              # Package metadata and dependencies
├── requirements.txt            # Runtime requirements
└── README.md
```

---

## 🧪 Running Tests

Tests are written with `pytest`. Make sure you have dev dependencies installed:

```bash
pip install -e .[dev]
```

Run the full test suite:

```bash
pytest tests/
```

Run a specific test file with verbose output:

```bash
pytest tests/test_transformers.py -v
```

Run the end-to-end pipeline demo:

```bash
python test_full_pipeline.py
```

---

## 🧩 Extending the Package

The package is designed to be extended. To add a new component, simply subclass the appropriate ABC from `data_processor.core`:

**Custom Reader:**
```python
from data_processor.core import DataReader, DataType

class MyAPIReader(DataReader):
    def __init__(self, endpoint: str):
        self.endpoint = endpoint

    def read(self) -> DataType:
        # Fetch and return data from your API
        ...
```

**Custom Transformer:**
```python
from data_processor.core import DataTransformer, DataType

class UpperCaseTransformer(DataTransformer):
    def __init__(self, column: str):
        self.column = column

    def transform(self, data: DataType) -> DataType:
        return [{**row, self.column: str(row.get(self.column, "")).upper()} for row in data]
```

**Custom Writer:**
```python
from data_processor.core import DataWriter, DataType

class DatabaseWriter(DataWriter):
    def write(self, data: DataType) -> None:
        # Insert rows into your database
        ...
```

Once created, drop them straight into any `DataPipeline` — no other changes needed.

---

## 📦 Dependencies

| Package    | Version    | Purpose                               |
|------------|------------|---------------------------------------|
| `openpyxl` | `>= 3.1.2` | Reading `.xlsx` Excel files           |
| `pytest`   | `>= 7.0`   | Running the test suite *(dev only)*   |

Python **3.8+** is required.

---

<div align="center">

Made with ❤️ · [MIT License](LICENSE)

</div>
