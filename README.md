# Data Processor

A modular, reusable, and document data processing package in Python. 
Built using Object-Oriented principles, this package provides abstract base classes for building robust data pipelines.

## Features

- **Readers**: Load data from various sources (CSV, JSON, Excel).
- **Transformers**: Clean and modify data in standard forms. Current transformers include `ColumnFilter`, `TypeCaster`, and `MissingValueImputer`.
- **Writers**: Output processed data to desired formats (e.g. CSV).
- **Workflows**: Run sequence of readers, transformers, and writers with a simple pipeline interface.

## Installation

You can install the package in editable mode along with its dependencies using:

```bash
pip install -e .
```

To install with development dependencies (like `pytest` for running tests):

```bash
pip install -e .[dev]
```

## Quick Start (Usage Example)

Here is a quick example of how you can use the `DataProcessor` package to read from an Excel file, apply some transformations, and write the output to a CSV file.

```python
from data_processor.readers import ExcelReader
from data_processor.transformers import TypeCaster, MissingValueImputer
from data_processor.writers import CSVWriter
from data_processor.workflows import DataPipeline

# 1. Setup Reader and Writer
reader = ExcelReader('dataset/Datasets.xlsx')
writer = CSVWriter('dataset/processed_dataset.csv')

# 2. Setup Pipeline
pipeline = DataPipeline(reader, writer)

# 3. Add Transformers
# Replace missing values in "Department" with "Unknown"
pipeline.add_transformer(MissingValueImputer("Department", "Unknown"))
# Cast "Salary" column to float
pipeline.add_transformer(TypeCaster("Salary", float))

# 4. Run Workflow
processed_data = pipeline.run()

print(f"Pipeline finished! Processed {len(processed_data)} rows.")
```

## Running Tests

To run the test suite, ensure you have installed the development dependencies (`pip install -e .[dev]`), then run:

```bash
pytest tests/
```
