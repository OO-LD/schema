# OO-LD Schema
The Object Oriented Linked Data Schema - work in process!

## Overview

OO-LD Schema aims to connect the structural modelling of objects and subobjects with the modelling of the semantic relations without reinventing the wheel. It therefor combines existing standards, primary [JSON-SCHEMA](https://json-schema.org/) and a [JSON-LD](https://json-ld.org/) context in the same document.

![](https://opensemantic.world/wiki/Special:Redirect/file/OSW95a74be1e22d4b6e9e4f836127d5915a.drawio.svg)
> JSON, JSON-SCHEMA and JSON-LD technology stack with [OpenSemanticLab](https://github.com/OpenSemanticLab) as example document store / platform

## Quickstart

If you are not familiar yet with [JSON-SCHEMA](https://json-schema.org/) or [JSON-LD](https://json-ld.org/) you should first have a look at dedicated tutorials like [OSW JSON-SCHEMA Tutorial](https://opensemantic.world/wiki/Item:OSWf4a9514baed04859a4c6c374a7312f10) and [OSW JSON-LD Tutorial](https://opensemantic.world/wiki/Item:OSW911488771ea449a6a34051f8213d7f2f).

The core idea is that an OO-LD document is always both a valid JSON-SCHEMA and a JSON-LD remote context ( != JSON-LD document). In this way a complete OO-LD class / schema hierarchy is consumeable by JSON-SCHEMA-only and JSON-LD-only tools while OO-LD aware tools can provide extended features on top (e.g. UI autocomplete dropdowns for string-IRI fields based e.g. on a SPARQL backend, SHACL shape or JSON-LD frame generation).

A minimal example:
```json
{
  "@context": {
    "schema": "http://schema.org/",
    "name": "schema:name"
  },
  "title": "Person",
  "type": "object",
  "properties": {
    "name": {
      "type": "string",
      "description": "First and Last name",
    }
  }
}
```

You can explore this in the [interactive playground](https://oo-ld.github.io/playground/)

Please note that **OO-LD schema documents should not be interpreted as JSON-LD documents** because this would apply `@context` on the schema itself. The motivation behind this is to have a single document so schemas can be aggregated using both the JSON-SCHEMA `$ref` and the JSON-LD remote `@context` pointing the same resource.

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

You can read how this is implemented in OpenSemanticWorld/Lab in the [introduction](https://opensemantic.world/wiki/Item:OSWdb485a954a88465287b341d2897a84d6) and [schema documentation draft](https://opensemantic.world/wiki/Item:OSWab674d663a5b472f838d8e1eb43e6784).

More details tbd...

## Standard extensions
### JSON-LD
Current support covers [v1.1] (https://www.w3.org/TR/json-ld/).

#### Multi-Mapping
JSON-LD allows only a single keyword-IRI mapping (or more precisely, ignores all but the last mapping). Currently there is no way to express that a property has two ids (e. g. with `"label": {"@id": ["schema:name", "skos:prefLabel"]}`, see also [json-ld/json-ld.org#160](https://github.com/json-ld/json-ld.org/issues/160)). As a workaround, an additional context notation is provided: `<property>*(*)` pointing to additional `@id` mappings to provide at least a documentation for alternative options or custom RDF generation.

```json
{
    "@context": [
        {
            "@version": 1.1,
            "skos": "https://www.w3.org/TR/skos-reference/",
            "schema": "https://schema.org/",
            "label": "skos:prefLabel",
            "label*": "schema:name",
            "label**": "..."
        }
    ]
}
```

### JSON-SCHEMA
Current support covers [Draft 4] (https://json-schema.org/specification-links#draft-4).

#### Multilanguange support
Keywords `title` and `description` can be extended with additional keywords `title*` and `description*`, which hold and object with lang-keys (de, en, etc.) pointing to the translated strings.
Mapping of `title*[lang]` must be provided by schema preprocessing.
```json
{
    "title": "Default Title",
    "title*": {"en": "Title (en)", "de": "Titel (de)"}
}
```

#### Range of properties
JSON-SCHEMA itself supports linked data only in form of subobject. References to independent external object are just URL-strings without any further restrictions. To express constrains on the type of the object as we know it from OWL and SHACL the keyword `range` is introduced (see also [json-schema-org/json-schema-vocabularies#55](https://github.com/json-schema-org/json-schema-vocabularies/issues/55)). Note: Same as `$ref`, `range` must point to a resolvable resource.

##### Draft v0.1:

`range` is an IRI 

```json
{
  "@context": {
    "schema": "http://schema.org/",
    "works_for": "schema:worksFor"
  },
  "title": "Person",
  "type": "object",
  "properties": {
    "works_for": {
      "type": "string",
      "range": "schema:Organization",
      "description": "IRI pointing to an instance of schema:Organization",
    }
  }
}
```

##### Draft v0.2:

`range` is an OO-LD schema

This will allow the following constellations:

###### Inline type restriction
```json
"range": {
  "@context": {
    "schema": "http://schema.org/",
    "type": "@type"
  },
  "properties": {
    "type": {
      "type": "string",
      "const": "schema:Organization",
    }
  }
}
```


###### Reference to existing schema
```json
"range": {
  "allOf": {
    "$ref": "Organization.schema.json"
  }
}
```

Full Example:

```json
{
  "@context": {
    "schema": "http://schema.org/",
    "works_for": "schema:worksFor",
    "type": "@type"
  },
  "$id": "Person.schema.json",
  "title": "Person",
  "type": "object",
  "properties": {
    "type": {
      "type": "string",
      "const": "schema:Person",
    }
    "works_for": {
      "type": "string",
      "range": {
        "allOf": {
          "$ref": "Organization.schema.json"
        }
      },
      "description": "IRI pointing to an instance of schema:Organization",
    }
  }
}
```

```json
{
  "@context": {
    "schema": "http://schema.org/",
    "type": "@type"
  },
  "$id": "Organization.schema.json",
  "title": "Organization",
  "type": "object",
  "properties": {
    "type": {
      "type": "string",
      "const": "schema:Organization",
    }
  }
}
```


#### UI Generation
Additional keywords defined by [JSON-SCHEMA Editor](https://github.com/json-editor/json-editor), see [Basic features](https://github.com/json-editor/json-editor#readme) and [Further details](https://github.com/json-editor/json-editor/blob/master/README_ADDON.md)

#### Code Generation

In general we want to keep keywords in 'instance' JSON-documents (=> property names in schemas) strict `^[A-z_]+[A-z0-9_]*$` to avoid escaping or replacing when mapping to other languages. This works well with [aliasing](https://www.w3.org/TR/json-ld11/#aliasing-keywords), e.g.
```json
{
  "@context": {
    "schema": "http://schema.org/",
    "name": "schema:name",
    "type": "@type"
  },
  "title": "Person",
  "type": "object",
  "properties": {
    "type": {
      "type": "string",
      "default": "schema:Person",
    },
    "name": {
      "type": "string",
      "description": "First and Last name",
    }
  }
}
```

##### Python

The Person schema above translates smoothly to python (pydantic) via https://github.com/koxudaxi/datamodel-code-generator:
```python
class Person(BaseModel):
    type: Optional[str] = "schema:Person"
    name: Optional[str] = None
    """First and Last name"""
```
what would not be the case if we use `@type` or `schema:name` as property names (See also [python playground](https://oo-ld.github.io/playground-python-yaml/)).
From pydantic it's also straight forward to [generate JSON- / OpenAPI-Schemas](https://docs.pydantic.dev/latest/concepts/json_schema/), especially via [FastAPI](https://fastapi.tiangolo.com/features/).

## Tooling
* General
  * [JSON-LD Tooling](https://json-ld.org/#developers)
  * [JSON-SCHEMA Tooling](https://json-schema.org/implementations)
* OO-LD Specific
  * Python: [oold-python](https://github.com/OpenSemanticWorld/oold-python)
  * Javascript Framework for graph visualization and editing: [interactive-semantic-graph](https://github.com/OpenSemanticLab/interactive-semantic-graph)
  * Fully integrated platform (currently) based on Semantic Mediawiki: [docker-compose](https://github.com/OpenSemanticLab/osl-mw-docker-compose), [Demo](https://demo.open-semantic-lab.org/wiki/Main_Page)

## Registry
* [OpenSemanticWorld Package Registry](https://github.com/OpenSemanticWorld-Packages), deployed e. g. [OpenSemanticWorld](https://opensemantic.world/)

## Discussion
* In the context of YAML-LD: https://github.com/json-ld/yaml-ld/issues/19

## Related Work

![](https://opensemantic.world/wiki/Special:Redirect/file/OSW01a9133879e94df19a8e617d91d28f39.drawio.svg)
> OO-LD as bridge between linked data and the general software domain

### Schema
| Name      | Description |
| ----------- | ----------- |
| [JSON-SCHEMA](https://json-schema.org/) | Base of this work. Does not include linked data concepts. |
| [JSON-LD](https://json-ld.org/) | Base of this work. Does not restrict the structure of a json file. |
| [OWL](https://www.w3.org/TR/2012/REC-owl2-quick-reference-20121211/) | Focus on logical modelling. Only applicable to RDF.  |
| [SHACL](https://www.w3.org/TR/shacl/) | Only applicable to RDF. |
| [Asset Administration Shell](https://github.com/admin-shell-io/aas-specs) | Industry 4.0 related data schema for assets |
| [Semantic Aspect Meta Model](https://eclipse-esmf.github.io/samm-specification/2.0.0/index.html) | SHACL subset written in turtl, e. g. used for the [data models in Catena-X](https://github.com/eclipse-tractusx/sldt-semantic-models)
| [SmartDataModels](https://smartdatamodels.org/) | JSON-SCHEMA defined data models used by  FIWARE Foundation, TM Forum, OASC and IUDX |
| [Common Data Model](https://eclipse-esmf.github.io/samm-specification/2.0.0/index.html) | Business related data models developed by Microsoft |
| [LinkML](https://github.com/linkml/linkml/issues/1618) | Custom schema language focussed on data modelling. Both [importers](https://linkml.io/schema-automator/packages/importers.html) and [exporters](https://linkml.io/linkml/generators/index.html) to JSON-SCHEMA (and others) exists. Custom annotations for UI generation not (yet) supported (see [#1618](https://github.com/linkml/linkml/issues/1618)). |
| [TreeLDR](https://www.spruceid.dev/treeldr/treeldr-overview) | Custom linked data schema language that can be converted to JSON-SCHEMA, JSON-LD context, RDF and Rust code |
| [REST-API-LD](https://datatracker.ietf.org/doc/draft-polli-restapi-ld-keywords/03/) | Annotated OpenAPI schemas with rendering support in [Swagger-UI](https://italia.github.io/swagger-editor/). Option to generate it from OO-LD. |
| [dlite](https://github.com/SINTEF/dlite) | Custom schema language focussed on scientific data |
| [NOMAD](http://nomad-lab.eu/prod/v1/staging/docs/schemas/basics.html) | Custom schema language focussed on scientific data |
| [Human Cell Atlas](https://data.humancellatlas.org/metadata) | Data schemas for the biology and medical domain |
| [OTTR](https://ottr.xyz/)   | Mixture of custom template and schema language. Limited toolset to convert from/to other formats. |
| [BatteryKnowledgeGraph](https://github.com/BIG-MAP/BatteryKnowledgeGraph) | Battery related linked data set |

### Data
| Name      | Description |
| ----------- | ----------- |
| [BatteryKnowledgeGraph](https://github.com/BIG-MAP/BatteryKnowledgeGraph) | Battery related linked data set |

## Mappings

### NOMAD

[NOMAD schemas](https://sol-oasis.physik.hu-berlin.de/nomad-oasis/docs/schemas/basics.html) have compareable concepts about reusable objects (sections) with typed properties (quantities).
Example:
```yaml
definitions:
  sections:
    Element:
      quantities:
        label:
          type: str
        density:
          type: np.float64
          unit: g/cm**3
        isotopes:
          type: int
          shape: ['*']
    Composition:
      quantities:
        composition:
          type: str
      sub_sections:
        elements:
          section: Element
          repeats: true
```
can be expressed as the following JSON-SCHEMA (formated as yaml)

```yaml
definitions:
  sections:
    Element:
      properties:
        label:
          type: string
        density:
          type: number
          unit: g/cm**3
        isotopes:
          type: array
          items:
            type: integer
          format: table
    Composition:
      properties:
        composition:
          type: str
        elements:
          type: array
          format: table
          items:
            $ref: '#/definitions/sections/Element'
```

generating/validating the same JSON/YAML data (see also [playground](https://oo-ld.github.io/playground-yaml/?data=N4Ig9gDgLglmB2BnEAuUMDGCA2MBGqIAZglAIYDuApomALZUCsIANOHgFZUZQD62ZAJ5gArlELwwAJzplsrEIgwALKrNSgAAlnhQqAD3FoQVbGqq7kKEMqhQIiFAHonFAMwwAJk7V0wT5TJEAAUyKXE2ATxTQlt7RxclVVkAOmkAcyd4MgYQAF82TyoiGHgYWAQrUERuCqQNEABRMwZdBogpSCpwmBoGqJjjKEEIKkJEKClS9PzCi0RywQbh0YkROmipBREyoxBMjDoAKiO3WZAYWiguqpAVseswqSEFcrVb%2B8JSvXTu85IZGQ9uQ8GZ8nkCiAAML0CBgBZ1dqdUY9PrGLB0OEIuDwZYjB6KSbnUzmSx41aPKTPJZsAGyYFkUFjNhvOi3AAkUmKhAAxE4iiVdjjEE4ajxhU5mqTxBDZZDOdzrHyBaVyhKxXURTDMfC1QgFAKyCJsEZQOyMa09gAZMBkTwAAloDHthuNUBdQLI9uu9rhEGNQKo3tU9qonnK0mQbAxWL1uOsAAkAEwAeQUJMtVgA2qABvJEwb5otUAAGFIlislgAcbgA7IwWVcbqgswBGFhJlhuAC6BVzjMGIDTcyQxZQrZSrYALO2Lk3RtnWwA2Fit2urqu93uQxDKMAUXjdTpSKwXXTdMji%2FVsKDJAl4MBgKATZ4QKcKMgcMj6VCTERUNgAGsqCoCBeAQKgwCIXgADc5H%2FKwiDkGpIQ4Wh41AGNdURRNU3TFoLGfFt%2B2ifMQATQtR2GUty0rGt60bJ9mxQNsOy7XsWBIwdhxAIoqKWcdJxnRjrgXFtl1XddW03PJuzYc16EtQgbTtR16CDV0TQ9chvTAX1IADPRgyDMMIxPcEgA%3D), e.g.

```yaml
composition: H2O
elements:
  - label: H
    density: 8.375e-05
    isotopes: [1, 2, 3]
  - label: O
    density: 1.141
    isotopes: [16, 17, 18]
```
by using the following mapping (work in progress):

| NOMAD Schema      | JSON-Schema |  Note |
| ----------- | ----------- | ----------- |
| quantities | properties | |
| type (int, str, ...) | type (integer, string, ...) | specific python types like `np.int32` can be annotated in the JSON-LD context |
| unit, m_annotations, .. | format / options| additional custom annotation keywords can be kept or mapped to format and options |
| shape[*] | type: array, items: type: number | specific values can be mapped to minItems and maxItems |
| shape[*, *] | type: array, items: type: array, items: type: number | nested array |
| sub_sections: ... : repeats: true | type: array, items: type: object | array of objects
|... | | |
