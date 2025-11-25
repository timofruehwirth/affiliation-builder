# Affiliation Builder

Build bipartite affiliation networks from JSON data using NetworkX.

[![PyPI version](https://badge.fury.io/py/affiliation-builder.svg)](https://pypi.org/project/affiliation-builder/)
[![Python 3.9+](https://img.shields.io/badge/python-3.9+-blue.svg)](https://www.python.org/downloads/)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

## Overview

**Affiliation Builder** is a Python package for creating bipartite networks from JSON data on co-affiliation relationships. It transforms structured data about entities (such as people and organizations) and their shared affiliations (such as in events) into NetworkX graph objects for analysis and visualization.

While designed with event-participant data in mind, the package works with any co-affiliation scenario where a set of entities connects to a set of items through shared relationships.

## Features

- **Flexible JSON input:** Supports various JSON structures (arrays, wrapped objects)
- **Multiple entity types:** Handle different entity types simultaneously (such as persons and organizations)
- **Simple and complex entities:** Work with string identifiers or objects
- **Rich metadata:** Preserve all JSON attributes as node properties
- **URL support:** Load data from local files or URLs
- **Comprehensive validation:** Detailed error messages and logging
- **NetworkX integration:** Returns standard NetworkX graph objects

## Requirements

- Python 3.9+
- NetworkX 3.0+
- Requests 2.31.0+

## Installation

```bash
pip install affiliation-builder
```

## Quick Start

```python
from affiliation_builder import build

# Build a bipartite network from JSON data
G = build(
    json_path='events.json',
    node_set_0_key='events',
    node_set_1_keys='participants',
    identifier_key='event_id',
    node_set_1_identifier_key='person_name'
)

# Returns a standard NetworkX Graph object
print(f"Nodes: {G.number_of_nodes()}")
print(f"Edges: {G.number_of_edges()}")

# Access node sets
node_set_0 = {n for n, d in G.nodes(data=True) if d['bipartite'] == 0}
node_set_1 = {n for n, d in G.nodes(data=True) if d['bipartite'] == 1}
```

## Understanding the Parameters

The `build()` function has 5 parameters that control how your JSON data maps to the bipartite network:

### Parameter 1: `json_path` (str or Path)

**What it is:** Path to your local JSON file or HTTP(S) URL

**Examples:**

```python
json_path='data/events.json'
json_path='https://example.com/data.json'
```

---

### Parameter 2: `node_set_0_key` (str or None)

**What it is:** JSON key containing your items (such as events)

**Use `None` if:** JSON is direct array of items (not wrapped in an object)

**Examples:**

Wrapped object format (specify the key):

```json
{
  "events": [
    {"id": "evt1", "participants": ["Alice", "Bob"]},
    {"id": "evt2", "participants": ["Bob", "Carol"]}
  ]
}
```

```python
node_set_0_key='events'
```

Direct array format (use None):

```json
[
  {"id": "evt1", "participants": ["Alice", "Bob"]},
  {"id": "evt2", "participants": ["Bob", "Carol"]}
]
```

```python
node_set_0_key=None
```

---

### Parameter 3: `node_set_1_keys` (str or list of str)
**What it is:** The JSON key(s) that contain the entities affiliated with each item

**Pass list when:** You have multiple entity types (e.g., both persons and organizations)

**Examples:**

Single entity type:

```json
{"id": "evt1", "participants": ["Alice", "Bob"]}
```

```python
node_set_1_keys='participants'
```

Multiple entity types:

```json
{
  "id": "evt1",
  "persons": ["Alice", "Bob"],
  "organizations": ["University A", "Company B"]
}
```

```python
node_set_1_keys=['persons', 'organizations']
```

---

### Parameter 4: `identifier_key` (str)

**What it is:** JSON key that uniquely identifies each item (e.g., event)

**Examples:**

```json
{"id": "evt1", "name": "Conference 2024", ...}
```

```python
identifier_key='id'
```

---

### Parameter 5: `node_set_1_identifier_key` (str or None, optional)

**What it is:** Key to extract identifiers from entity objects (when entities are objects, not strings)

**Use `None` (default) when:** Entities are simple strings/numbers

**Pass key when:** Entities are objects with multiple attributes

**Examples:**

Simple entities (strings):

```json
{"id": "evt1", "participants": ["Alice", "Bob"]}
```

```python
node_set_1_identifier_key=None
```

Complex entities (objects):

```json
{
  "id": "evt1",
  "participants": [
    {"person_name": "Alice", "role": "speaker", "affiliation": "MIT"},
    {"person_name": "Bob", "role": "attendee", "affiliation": "Stanford"}
  ]
}
```
```python
# Extract 'Alice' and 'Bob' as node IDs
# All other attributes (role, affiliation) are preserved as node properties
node_set_1_identifier_key='person_name'
```

## JSON Structure Examples

### Example 1: Wrapped Object with Simple Entities

```json
{
  "events": [
    {"name": "Conference 2024", "participants": ["Alice", "Bob", "Carol"]},
    {"name": "Workshop 2024", "participants": ["Bob", "David"]}
  ]
}
```
```python
G = build(
    json_path='events.json',
    node_set_0_key='events',
    node_set_1_keys='participants',
    identifier_key='name'
)
```

---

### Example 2: Direct Array with Complex Entities
```json
[
  {
    "project_id": "proj1",
    "members": [
      {"name": "Alice", "role": "lead", "department": "Engineering"},
      {"name": "Bob", "role": "contributor", "department": "Design"}
    ]
  }
]
```
```python
G = build(
    json_path='projects.json',
    node_set_0_key=None,  # Direct array
    node_set_1_keys='members',
    identifier_key='project_id',
    node_set_1_identifier_key='name'  # Extract name from member objects
)

# All attributes preserved as node properties
print(G.nodes['Alice'])  # {'bipartite': 1, 'role': 'lead', 'department': 'Engineering'}
```

---

### Example 3: Multiple Entity Types
```json
{
  "events": [
    {
      "name": "Summit 2024",
      "persons": ["Alice", "Bob"],
      "organizations": ["Company A", "University B"]
    }
  ]
}
```
```python
G = build(
    json_path='https://example.com/data/events.json',
    node_set_0_key='events',
    node_set_1_keys=['persons', 'organizations'],  # Multiple types
    identifier_key='name'
)
```

