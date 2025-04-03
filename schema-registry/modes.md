# Schema Registry Compatibility Modes

## Summary

### JSON Schema

- [BACKWARD Mode](#backward-mode)

  - Allowed transformations
    - [Adding optional fields](#Adding-optional-fields)
    - [Making required fields optional](#Making-required-fields-optional)
    - [Widening value ranges](#Widening-value-ranges)
    - [Adding enum values](#Adding-enum-values)
    - [Relaxing string patterns](#Relaxing-string-patterns)
    - [Adding new types to unions](#Adding-new-types-to-unions)
  - Non-allowed transformations
    - [Removing fields](#Removing-fields)
    - [Adding required fields](#Adding-required-fields)
    - [Changing property types](#Changing-property-types)
    - [Narrowing value ranges](#Narrowing-value-ranges)
    - [Removing enum values](#Removing-enum-values)
    - [Restricting string patterns](#Restricting-string-patterns)
    - [Adding additionalProperties: false](#Adding-additionalProperties-false)

- [BACKWARD_ALL Mode](#backward_all-mode)

  - Allowed transformations
    - All transformations allowed in BACKWARD mode
  - Non-allowed transformations
    - Same as BACKWARD mode but checked against all previous schema versions

- [FORWARD Mode](#forward-mode)

  - Allowed transformations
    - [Removing optional fields](#Removing-optional-fields)
    - [Adding required fields](#Adding-required-fields-forward)
    - [Narrowing value ranges](#Narrowing-value-ranges-forward)
    - [Removing enum values](#Removing-enum-values-forward)
    - [Making string patterns more restrictive](#Making-string-patterns-more-restrictive)
  - Non-allowed transformations
    - [Adding optional fields](#Adding-optional-fields-forward)
    - [Removing required fields](#Removing-required-fields)
    - [Changing property types](#Changing-property-types-forward)
    - [Widening value ranges](#Widening-value-ranges-forward)
    - [Adding enum values](#Adding-enum-values-forward)

- [FORWARD_ALL Mode](#forward_all-mode)

  - Allowed transformations
    - All transformations allowed in FORWARD mode
  - Non-allowed transformations
    - Same as FORWARD mode but checked against all previous schema versions

- [FULL Mode](#full-mode)

  - Allowed transformations
    - Intersection of transformations allowed in both BACKWARD and FORWARD modes
  - Non-allowed transformations
    - Union of transformations not allowed in either BACKWARD or FORWARD modes

- [FULL_ALL Mode](#full_all-mode)
  - Allowed transformations
    - All transformations allowed in FULL mode
  - Non-allowed transformations
    - Same as FULL mode but checked against all previous schema versions

# JSON Schema

## BACKWARD Mode

- **New consumers** can read data from the **immediate previous schema version**.
- **Old consumers** **cannot** read new data.

### Adding optional fields

```mermaid
flowchart LR
    subgraph "Schema Versions"
        V1["Schema V1 {id*, name*}"]
        V2["Schema V2 {id*, name*, email}"]
    end

    subgraph "Consumers"
        C1["Consumer V1"]
        C2["Consumer V2"]
    end

    V1 -->|✅| C1
    V1 -->|✅| C2
    V2 -->|❌| C1
    V2 -->|✅| C2

    classDef schema fill:#e1f7d5,stroke:#333,stroke-width:2px,color:#000000
    classDef consumer fill:#a1c4f7,stroke:#333,stroke-width:2px,color:#000000
    class V1,V2 schema
    class C1,C2 consumer
```

### Making required fields optional

```mermaid
flowchart LR
    subgraph "Schema Versions"
        V1["Schema V1 {id*, name*, email*}"]
        V2["Schema V2 {id*, name*, email}"]
    end

    subgraph "Consumers"
        C1["Consumer V1"]
        C2["Consumer V2"]
    end

    V1 -->|✅| C1
    V1 -->|✅| C2
    V2 -->|❌| C1
    V2 -->|✅| C2

    classDef schema fill:#e1f7d5,stroke:#333,stroke-width:2px,color:#000000
    classDef consumer fill:#a1c4f7,stroke:#333,stroke-width:2px,color:#000000
    class V1,V2 schema
    class C1,C2 consumer
```

### Widening value ranges

```mermaid
flowchart LR
    subgraph "Schema Versions"
        V1["Schema V1 {age: integer(0-65)}"]
        V2["Schema V2 {age: integer(0-100)}"]
    end
    subgraph "Consumers"
        C1["Consumer V1"]
        C2["Consumer V2"]
    end
    V1 -->|✅| C1
    V1 -->|✅| C2
    V2 -->|❌| C1
    V2 -->|✅| C2
    classDef schema fill:#e1f7d5,stroke:#333,stroke-width:2px,color:#000000
    classDef consumer fill:#a1c4f7,stroke:#333,stroke-width:2px,color:#000000
    class V1,V2 schema
    class C1,C2 consumer
```

### Adding enum values

```mermaid
flowchart LR
    subgraph "Schema Versions"
        V1["Schema V1 {status: enum('active', 'inactive')}"]
        V2["Schema V2 {status: enum('active', 'inactive', 'pending')}"]
    end

    subgraph "Consumers"
        C1["Consumer V1"]
        C2["Consumer V2"]
    end

    V1 -->|✅| C1
    V1 -->|✅| C2
    V2 -->|❌| C1
    V2 -->|✅| C2

    classDef schema fill:#e1f7d5,stroke:#333,stroke-width:2px,color:#000000
    classDef consumer fill:#a1c4f7,stroke:#333,stroke-width:2px,color:#000000
    class V1,V2 schema
    class C1,C2 consumer
```

---

## BACKWARD_ALL Mode

- **New consumers** can read data from **all previous schema versions**.
- **Old consumers** **cannot** read new data.

### Adding optional fields

```mermaid
flowchart LR
    subgraph "Schema Versions"
        V1["Schema V1 {id*}"]
        V2["Schema V2 {id*, name}"]
        V3["Schema V3 {id*, name, email}"]
    end

    subgraph "Consumers"
        C1["Consumer V1"]
        C2["Consumer V2"]
        C3["Consumer V3"]
    end

    V1 -->|✅| C1
    V1 -->|✅| C2
    V1 -->|✅| C3

    V2 -->|❌| C1
    V2 -->|✅| C2
    V2 -->|✅| C3

    V3 -->|❌| C1
    V3 -->|❌| C2
    V3 -->|✅| C3

    classDef schema fill:#e1f7d5,stroke:#333,stroke-width:2px,color:#000000
    classDef consumer fill:#a1c4f7,stroke:#333,stroke-width:2px,color:#000000
    class V1,V2,V3 schema
    class C1,C2,C3 consumer
```

---

## FORWARD Mode

- **Old consumers** can read data produced with new schema version.
- **New consumers** **cannot** read old data.

### Removing optional fields

```mermaid
flowchart LR
    subgraph "Schema Versions"
        V1["Schema V1 {id*, name}"]
        V2["Schema V2 {id*}"]
    end

    subgraph "Consumers"
        C1["Consumer V1"]
        C2["Consumer V2"]
    end

    V1 -->|✅| C1
    V1 -->|❌| C2
    V2 -->|✅| C1
    V2 -->|✅| C2

    classDef schema fill:#e1f7d5,stroke:#333,stroke-width:2px,color:#000000
    classDef consumer fill:#a1c4f7,stroke:#333,stroke-width:2px,color:#000000
    class V1,V2 schema
    class C1,C2 consumer
```

---

## FORWARD_ALL Mode

- **Old consumers** can read data produced with **all new schema version**
- **New consumers** **cannot** read old data.

### Adding required fields

```mermaid
flowchart LR
    subgraph "Schema Versions"
        V1["Schema V1 {id*}"]
        V2["Schema V2 {id*, name*}"]
        V3["Schema V3 {id*, name*, email*}"]
    end

    subgraph "Consumers"
        C1["Consumer V1"]
        C2["Consumer V2"]
        C3["Consumer V3"]
    end

    V1 -->|✅| C1
    V1 -->|❌| C2
    V1 -->|❌| C3

    V2 -->|✅| C1
    V2 -->|✅| C2
    V2 -->|❌| C3

    V3 -->|✅| C1
    V3 -->|✅| C2
    V3 -->|✅| C3

    classDef schema fill:#e1f7d5,stroke:#333,stroke-width:2px,color:#000000
    classDef consumer fill:#a1c4f7,stroke:#333,stroke-width:2px,color:#000000
    class V1,V2,V3 schema
    class C1,C2,C3 consumer
```

---

## FULL Mode

- **Both new and old consumers** can read data across schema versions.
- It checks against the **previous schema versions only** (not all past versions).

### Example

```mermaid
flowchart LR
    subgraph "Schema Versions"
        V1["Schema V1 {id*, name*, email*}"]
        V2["Schema V2 {id*, name*, email}"]
    end

    subgraph "Consumers"
        C1["Consumer V1"]
        C2["Consumer V2"]
    end

    V1 -->|✅| C1
    V1 -->|✅| C2
    V2 -->|✅| C1
    V2 -->|✅| C2

    classDef schema fill:#e1f7d5,stroke:#333,stroke-width:2px,color:#000000
    classDef consumer fill:#a1c4f7,stroke:#333,stroke-width:2px,color:#000000
    class V1,V2 schema
    class C1,C2 consumer
```

---

## FULL_ALL Mode

- Every schema version must be compatible with **all** past and future versions.
- The **strictest mode**.

### **Example:**

```mermaid
flowchart LR
    subgraph "Schema Versions"
        V1["Schema V1 {id*, name*}"]
        V2["Schema V2 {id*, name*, email}"]
        V3["Schema V3 {id*, name*, email, age}"]
        V4["Schema V4 {id*, email*, age} (INCOMPATIBLE)"]
    end

    subgraph "Consumers"
        C1["Consumer V1"]
        C2["Consumer V2"]
        C3["Consumer V3"]
    end

    V1 -->|✅| C1
    V1 -->|✅| C2
    V1 -->|✅| C3

    V2 -->|✅| C1
    V2 -->|✅| C2
    V2 -->|✅| C3

    V3 -->|✅| C1
    V3 -->|✅| C2
    V3 -->|✅| C3

    V4 -.->|❌| C1
    V4 -.->|❌| C2
    V4 -.->|❌| C3

    classDef schema fill:#e1f7d5,stroke:#333,stroke-width:2px,color:#000000
    classDef consumer fill:#a1c4f7,stroke:#333,stroke-width:2px,color:#000000
    classDef incompatible fill:#f8d7da,stroke:#333,stroke-width:2px,color:#000000
    class V1,V2,V3 schema
    class C1,C2,C3 consumer
    class V4 incompatible
```

## Personal Notes

**Compatibility Modes**

| Mode             | Description                                         | Compatibility Behavior                                                   |
| ---------------- | --------------------------------------------------- | ------------------------------------------------------------------------ |
| **Backward**     | New schemas can read old data                       | New consumers can read data from immediate previous schema version       |
| **Backward All** | New schemas can read all old data                   | New consumers can read data from all previous schema versions            |
| **Forward**      | Old schemas can read new data                       | Old consumers can read data produced with new schema version             |
| **Forward All**  | Old schemas can read all new data                   | Old consumers can read data produced with all new schema versions        |
| **Full**         | Both new and old schemas can read each other's data | Both new and old consumers can read data across adjacent schema versions |
| **Full All**     | Both new and old schemas can read all data          | Every schema version is compatible with all past and future versions     |

## Additional Resources

- [Comparison of JSON schema validator implementations](https://www.creekservice.org/articles/2023/11/14/json-validator-comparison.html)
