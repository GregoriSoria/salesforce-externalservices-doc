---
# Feel free to add content and custom Front Matter to this file.
# To modify the layout, see https://jekyllrb.com/docs/themes/#overriding-theme-defaults

layout: default
title: "SFES Doc | Limitações"
permalink: /limitacoes
---

##### Navegação

- [Voltar ao início](./)
- [Tutorial](./tutorial)

---

## Limitações das definições do Schema OpenAPI:

#### When you create your API spec, keep the following in mind.

- You can use the `GET`, `PATCH`, `PUT`, `POST`, and `DELETE` methods in a schema.
- A property must include a value.
- Each parameter must have a name.
- Request headers are supported.
- **Response headers aren’t supported.** (System.ExternalServiceMethods.invokeOperation)
- Form parameters are supported.
- Security settings defined in the API spec are ignored and defer to security settings in the Named Credential.

#### The following [[OpenAPI]] schema components are ignored:

- security requirement objects and security definitions
- tag objects
- external documentation objects
- For `allOf`, `oneOf`, and `anyOf`, the schema object property `discriminator` is supported. The `discriminator/mapping` in [[OpenApi]] 3.0 is ignored The `discriminator` property determines the schema referenced by its type name. For more information, see the Swagger OpenAPI 3.0 specification [Inheritance and Polymorphism](https://swagger.io/docs/specification/data-models/inheritance-and-polymorphism/).

#### Supported data types:

- boolean
- date
- datetime
- double
- float
- integer
- long
- string
- any type (as Apex Object)
- object: top level named and nested anonymous objects
- anonymous and top-level named lists (can nest both named and anonymous arrays)
- additionalProperties as maps
- allOf as an object composition
- [[OpenAPI]] 3.0 only:
    - anyOf (any of primitive, list, or object schema types; only object schemas can constitute composition types)
    - oneOf (one of primitive, list, or object schema types)

> Ref: https://help.salesforce.com/s/articleView?id=sf.external_services_schema_def_definition.htm&type=5
