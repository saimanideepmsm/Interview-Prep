# YAML Learning Notes

## 1. What is YAML?

**YAML** stands for **"YAML Ain't Markup Language."**

YAML is a human-readable **data serialization language** used to represent structured data.

It is commonly used for:

* Configuration files
* Application settings
* Data exchange
* Automation tools
* CI/CD configuration

---

## 2. What is Serialization?

**Serialization** is the process of converting structured data or objects into a format that can be:

* Stored
* Transmitted
* Read later
* Reconstructed back into its original structure

For example:

```text
Name = Manideep
Age = 25
City = Hyderabad
```

Can be represented using YAML as:

```yaml
name: Manideep
age: 25
city: Hyderabad
```

---

## 3. Examples of Serialization Languages

Some commonly used serialization formats are:

| Format           | Common Usage                   |
| ---------------- | ------------------------------ |
| YAML             | Configuration files            |
| JSON             | APIs and web applications      |
| XML              | Enterprise applications        |
| TOML             | Configuration files            |
| Protocol Buffers | High-performance data exchange |
| MessagePack      | Compact data serialization     |

### YAML vs JSON

YAML:

```yaml
name: Manideep
age: 25
city: Hyderabad
```

JSON:

```json
{
  "name": "Manideep",
  "age": 25,
  "city": "Hyderabad"
}
```

YAML is generally easier for humans to read because it uses indentation to represent structure.

---

# 4. YAML Syntax

YAML files normally use either:

```text
.yaml
```

or:

```text
.yml
```

The basic YAML syntax is:

```yaml
key: value
```

Example:

```yaml
name: Manideep
age: 25
role: Developer
```

---

## 5. YAML Variables / Key-Value Pairs

YAML itself does not have programming-language-style variables.

Instead, YAML represents data using **key-value pairs**.

```yaml
name: Manideep
age: 25
city: Hyderabad
```

Here:

* `name` → key
* `Manideep` → value
* `age` → key
* `25` → value

---

## 6. Strings

Strings can be written without quotes:

```yaml
name: Manideep
city: Hyderabad
```

Or with quotes:

```yaml
name: "Manideep"
city: "Hyderabad"
```

---

## 7. Numbers

YAML supports numbers:

```yaml
age: 25
experience: 3
salary: 50000
```

---

## 8. Boolean Values

Boolean values can be represented using:

```yaml
isDeveloper: true
isStudent: false
```

---

## 9. Lists

Lists are represented using `-`.

```yaml
skills:
  - Java
  - Docker
  - Kubernetes
  - YAML
```

Another example:

```yaml
cities:
  - Hyderabad
  - Bangalore
  - Chennai
```

---

## 10. Nested Objects

YAML uses indentation to represent nested data.

```yaml
employee:
  name: Manideep
  role: Developer
  address:
    city: Hyderabad
    country: India
```

The structure is:

```text
employee
│
├── name
├── role
│
└── address
    ├── city
    └── country
```

---

## 11. Indentation

Indentation is extremely important in YAML.

Correct:

```yaml
employee:
  name: Manideep
  age: 25
```

Incorrect:

```yaml
employee:
name: Manideep
age: 25
```

Use **spaces**, not tabs, for indentation.

---

## 12. Comments

Comments start with `#`.

```yaml
# Employee information

name: Manideep
age: 25
```

Inline comments are also possible:

```yaml
name: Manideep  # Employee name
age: 25         # Employee age
```

---

## 13. Multiple Data Types

A YAML file can contain different data types together:

```yaml
name: Manideep
age: 25
isDeveloper: true

skills:
  - Java
  - Docker
  - YAML
```

Here:

```text
name         → String
age          → Number
isDeveloper  → Boolean
skills       → List
```

---

## 14. Important YAML Rules

### Rule 1: Use `key: value`

```yaml
name: Manideep
```

### Rule 2: Use spaces for indentation

```yaml
employee:
  name: Manideep
```

### Rule 3: Avoid tabs

Use spaces instead of tabs.

### Rule 4: YAML is case-sensitive

These are different:

```yaml
name: Manideep
Name: Manideep
```

### Rule 5: Lists use `-`

```yaml
skills:
  - Java
  - Python
  - YAML
```

### Rule 6: Comments use `#`

```yaml
# This is a comment
name: Manideep
```

---

# 15. Simple YAML Example

```yaml
employee:
  name: Manideep
  age: 25
  role: Developer
  isActive: true

  address:
    city: Hyderabad
    country: India

  skills:
    - Java
    - Docker
    - Kubernetes
    - YAML
```

This single example demonstrates:

* Key-value pairs
* Strings
* Numbers
* Boolean values
* Lists
* Nested objects
* Indentation
* YAML structure

---

# 16. Learning Progress

* [x] Understand what YAML means
* [x] Understand serialization
* [x] Learn examples of serialization languages
* [x] Understand YAML structure
* [x] Learn key-value pairs
* [x] Learn strings
* [x] Learn numbers
* [x] Learn Boolean values
* [x] Learn lists
* [x] Learn nested objects
* [x] Learn indentation
* [x] Learn comments
* [x] Understand YAML data types
* [x] Create a simple YAML file

---

## Quick Reference

```yaml
# String
name: Manideep

# Number
age: 25

# Boolean
isDeveloper: true

# List
skills:
  - Java
  - Docker
  - YAML

# Nested object
address:
  city: Hyderabad
  country: India
```

### Key Takeaway

YAML is a **human-readable data serialization language** that primarily uses:

```text
key: value
```

and **indentation** to represent structured data.
