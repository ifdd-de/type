(ifdd-type-index)=
# {{ cookiecutter.ifdd_type_prj_name }} (type)

The static content service `type` of an IFDD node: it serves the
vocabularies of the namespaces, which explain every property of the
documents in several languages. It runs the upstream nginx image as an unprivileged user behind the
proxy and builds no image of its own: the configuration and the content are
mounted read-only. This chapter describes the instance
`{{ cookiecutter.ifdd_type_prj_slug }}`, generated from the template
`ifdd-de/type`, from the point of view of the people who set it up, run and
secure it.

| | |
|---|---|
| Host | `{{ cookiecutter.ifdd_type_host }}` (default of `IFDD_TYPE_HOST` in `.env`; routed by the proxy) |
| Base URL | `{{ cookiecutter.ifdd_type_url }}` (`ifdd_type_url`, shared) |
| Image | `{{ cookiecutter.ifdd_nginx_image }}` (shared by schema, type and unit) |
| Content | the directory `IFDD_TYPE_CONTENT_DIR` in `.env`, usually a clone of a separate content repository |
| Network | `{{ cookiecutter.ifdd_proxy_network }}` |
| Development | `http://127.0.0.1:{{ cookiecutter.ifdd_type_dev_port }}/` |

## What the instance contains

The nginx configuration (`nginx/default.conf`, `nginx/mime.types`), two
compose files, `.env`, and example content in `examples/json-ld/`. The real
content is not part of the instance: it lives in its own repository, whose
clone is mounted read-only. `ifdd-de/schema`, `ifdd-de/type` and
`ifdd-de/unit` share this structure.

## Vocabularies

A namespace is one directory with one document, `<namespace>/index.jsonld`,
holding all of its vocabularies in `@graph`. nginx answers the namespace IRI
(`<namespace>/`) and every IRI below it (`<namespace>/<name>`) with that
document; a client finds its node by `@id`. So a property IRI in the data
can be opened in a browser and shows its meaning, and a program reads the
same JSON-LD. Files that exist are served as they are; an IRI of an unknown
namespace answers 404. Keep the IRIs in the documents equal to the URLs they
are served at (below `{{ cookiecutter.ifdd_type_url }}`).

The examples use standard vocabularies only: `rdfs:label` and
`rdfs:comment` per language (`en`, `de`), `rdfs:range`,
`rdfs:isDefinedBy`, `sh:order` (order in the UI), `qudt:hasQuantityKind`,
`qudt:applicableUnit` (units of the unit service,
`{{ cookiecutter.ifdd_unit_url }}`), `qudt:symbol` and `skos:exactMatch`.
The UI shows the label of a namespace as the title of its tab, so it is a
plain name; notes such as "example" go into `rdfs:comment`.

## Architecture

```{mermaid}
flowchart LR
  Clients[browsers, services] -->|HTTPS| Proxy[proxy: Traefik]
  Proxy -->|"{{ cookiecutter.ifdd_type_host }}, port 8080"| Nginx[type: nginx]
  Nginx -->|read-only| Content[(content clone)]
  Repo[content repository] -->|git pull| Content
```

| Who | Uses this service for |
|---|---|
| ui | labels, help texts and the order of the properties (in the browser, cross-origin) |
| schema | the namespaces of its contexts point here |
| people and programs | a property IRI from the data, opened directly |

## Chapters

```{toctree}
:maxdepth: 2

setup
build-release
operations
security
troubleshooting
reference
```

The values of the variables of this instance and how they relate to the
other services are listed in the chapter on the service instances
({ref}`ifdd-instances-static`).
