# OO-LD Schema
The Object Oriented Linked Data Schema

## Overview

OO-LD Schema combines a [JSON-SCHEMA](https://json-schema.org/) and a [JSON-LD](https://json-ld.org/) context in the same document.
Schemas can be aggregated using both the JSON-SCHEMA `$ref` and the JSON-LD remote `@context` pointing the same resource.

```mermaid
%%{init: {'theme': 'neutral' } }%%
classDiagram
    class OOLD_Class_A {
        JSON-SCHEMA + JSON-LD
        @context: ...
        properties: a...
    }

    class OOLD_Class_B {
        JSON-SCHEMA + JSON-LD
        @context: ./A
        allOf: ./A
        properties: b...
    }

    class OOLD_Instance_B {
        JSON
        @context: ./B
        $schema: ./B
        a: ...
        b: ...
    }

    OOLD_Class_A <-- OOLD_Class_B: extends

    OOLD_Class_B <-- OOLD_Instance_B: type
```

## Related Work

| Name      | Description |
| ----------- | ----------- |
| [JSON-SCHEMA](https://json-schema.org/) | Base of this work. Does not include linked data concepts. |
| [JSON-LD](https://json-ld.org/) | Base of this work. Does not restrict the structure of a json file. |
| [OWL](https://www.w3.org/TR/2012/REC-owl2-quick-reference-20121211/) | Focus on logical modelling. Only applicable to RDF.  |
| [SHACL](https://www.w3.org/TR/shacl/) | Only applicable to RDF. |
| [LinkML](https://github.com/linkml/linkml/issues/1618) | Custom schema language focussed on data modelling. Both [importers](https://linkml.io/schema-automator/packages/importers.html) and [exporters](https://linkml.io/linkml/generators/index.html) to JSON-SCHEMA (and others) exists. Custom annotations for UI generation not (yet) supported (see [#1618](https://github.com/linkml/linkml/issues/1618)). |
| [dlite](https://github.com/SINTEF/dlite) | Custom schema language focussed on scientific data |
| [NOMAD](http://nomad-lab.eu/prod/v1/staging/docs/schemas/basics.html) | Custom schema language focussed on scientific data |
| [OTTR](https://ottr.xyz/)   | Mixture of custom template and schema language. Limited toolset to convert from/to other formats. |
### LinkML
 https://github.com/linkml/linkml/issues/1618
