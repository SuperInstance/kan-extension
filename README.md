# Kan Extension

[![crates.io](https://img.shields.io/crates/v/kan-extension.svg)](https://crates.io/crates/kan-extension)
[![docs.rs](https://docs.rs/kan-extension/badge.svg)](https://docs.rs/kan-extension)
[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)

> **Category theory Kan extensions for computational capability composition.**

---

## The Problem

When composing capabilities in multi-agent systems, you need a mathematical framework that guarantees consistency. Ad-hoc composition leads to broken invariants, unexpected interactions, and systems that can't be reasoned about formally.

## Why This Exists

Kan Extension brings category theory to capability composition:
- **Categories** with objects and morphisms
- **Functors** that map between categories preserving structure
- **Natural Transformations** between functors
- **Kan Extensions** (left and right) for extending functors along other functors
- **Functor Composition** (G ∘ F) with automatic chain resolution

This provides a rigorous mathematical foundation for composing agent capabilities.

## Architecture

```
  Category A ──Functor F──→ Category B ──Functor G──→ Category C
       │                                              │
       └──────── G ∘ F (composition) ─────────────────┘

  Objects: ObjId(String)    Morphisms: source → target
  Functors: obj_map + mor_map between categories
  
  Kan Extensions:
    Left Kan:  Lan_K(F) = colimit over comma category
    Right Kan: Ran_K(F) = limit over comma category
```

## Installation

```toml
[dependencies]
kan-extension = "0.1"
```

## API Reference

### `Category`

A small category with objects and morphisms:

```rust
use kan_extension::functor::{Category, Morphism, ObjId};

let mut cat = Category::new("AgentCapabilities");
cat.add_object("perceive");
cat.add_object("reason");
cat.add_morphism(Morphism::new(
    ObjId("perceive".into()), ObjId("reason".into()), "perceive_then_reason"
));
```

### `Functor`

A structure-preserving map between categories:

```rust
use kan_extension::functor::Functor;

let mut f = Functor::new("F", "source", "target");
f.map_object("a", "A");
f.map_object("b", "B");
f.map_morphism("a_to_b", "A_to_B");

assert_eq!(f.apply_object("a"), Some("A"));
assert!(f.is_defined_on_object("a"));
```

### `compose_functors`

Compose two functors (G ∘ F):

```rust
use kan_extension::functor::compose_functors;

let composed = compose_functors(&f, &g);
// Automatically chains object and morphism maps
```

## Usage Examples

### Example 1: Model Agent Capabilities as Categories

```rust
use kan_extension::functor::*;

let mut input_cat = Category::new("Input");
input_cat.add_object("sensor_data");
input_cat.add_object("text_input");

let mut process_cat = Category::new("Processing");
process_cat.add_object("embedding");
process_cat.add_object("classification");

let mut functor = Functor::new("embed", "Input", "Processing");
functor.map_object("sensor_data", "embedding");
functor.map_object("text_input", "embedding");
```

### Example 2: Functor Composition

```rust
use kan_extension::functor::*;

let mut f = Functor::new("F", "A", "B");
f.map_object("x", "y");

let mut g = Functor::new("G", "B", "C");
g.map_object("y", "z");

let gf = compose_functors(&f, &g);
assert_eq!(gf.apply_object("x"), Some("z"));
```

## Theoretical Background

**Kan Extensions** are one of the most fundamental constructions in category theory. Given functors F: C → D and K: C → E, the left Kan extension Lan_K(F) extends F along K to a functor E → D. They are universal constructions that subsume limits, colimits, adjunctions, and the Yoneda embedding.

## License

Licensed under the [MIT License](LICENSE).

## Contributing

1. Fork the repository
2. Create a feature branch
3. Write tests
4. Push and open a Pull Request

## Mathematical Background

### Categories

A **category** C consists of:
- A collection of **objects** (Ob(C))
- A collection of **morphisms** (Hom(C)) between objects
- **Composition** of morphisms: if f: A → B and g: B → C, then g ∘ f: A → C
- **Identity** morphisms: id_A: A → A for every object

### Functors

A **functor** F: C → D maps:
- Each object in C to an object in D
- Each morphism in C to a morphism in D
- Preserves composition: F(g ∘ f) = F(g) ∘ F(f)
- Preserves identities: F(id_A) = id_{F(A)}

### Kan Extensions

Given functors F: C → D and K: C → E:

**Left Kan Extension** (Lan_K F): The "best approximation" of extending F along K from the left. Computed as a colimit:

```
Lan_K(F)(e) = colimit_{(c, k: Kc → e)} F(c)
```

**Right Kan Extension** (Ran_K F): The "best approximation" from the right. Computed as a limit:

```
Ran_K(F)(e) = limit_{(c, k: e → Kc)} F(c)
```

### Why Kan Extensions Matter

Kan extensions are universal constructions that subsume:
- **Limits and colimits**: Limits are right Kan extensions, colimits are left
- **Adjunctions**: Every adjunction gives rise to Kan extensions
- **Yoneda lemma**: A special case of Kan extensions

In computational terms, Kan extensions provide the most general framework for composing capabilities across different abstraction levels.

## Performance Characteristics

| Operation | Complexity |
|-----------|-----------|
| Category operations | O(1) amortized |
| Functor application | O(1) HashMap lookup |
| Functor composition | O(n × m) where n,m = mappings |
| Object/morphism addition | O(1) |

## Comparison with Alternatives

| Feature | kan-extension | type-system | trait-composition |
|---------|-------------|-------------|-------------------|
| Formal mathematical basis | ✅ Category theory | ✅ Type theory | ❌ Ad-hoc |
| Functor composition | ✅ Native | ❌ | ❌ |
| Universal constructions | ✅ Kan extensions | ❌ | ❌ |
| Rust integration | ✅ | ✅ | ✅ |

## Usage Examples

### Example 3: Multi-Level Composition

```rust
use kan_extension::functor::*;

// Three levels: Input → Processing → Output
let mut input_cat = Category::new("Input");
input_cat.add_object("raw_data");
input_cat.add_object("parsed");

let mut process_cat = Category::new("Processing");
process_cat.add_object("embedding");
process_cat.add_object("features");

let mut output_cat = Category::new("Output");
output_cat.add_object("classification");
output_cat.add_object("score");

let mut f = Functor::new("parse", "Input", "Processing");
f.map_object("raw_data", "embedding");
f.map_object("parsed", "features");

let mut g = Functor::new("classify", "Processing", "Output");
g.map_object("embedding", "classification");
g.map_object("features", "score");

// Compose: Input → Output directly
let gf = compose_functors(&f, &g);
assert_eq!(gf.apply_object("raw_data"), Some("classification"));
```
