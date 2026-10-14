(ifdd-type-index)=
# {{ cookiecutter.ifdd_type_prj_name }} (type)

Der statische Inhaltsdienst `type` eines IFDD-Knotens: Er liefert die
Vokabulare der Namensräume, die jede Eigenschaft der Dokumente in mehreren
Sprachen erklären. Er betreibt das Upstream-nginx-Image als unprivilegierter Benutzer hinter
dem Proxy und baut kein eigenes Image: Konfiguration und Inhalt werden nur
lesend eingebunden. Dieses Kapitel beschreibt die Instanz
`{{ cookiecutter.ifdd_type_prj_slug }}`, erzeugt aus dem Template `ifdd-de/type`,
aus Sicht derer, die sie einrichten, betreiben und absichern.

| | |
|---|---|
| Host | `{{ cookiecutter.ifdd_type_host }}` (Vorgabe von `IFDD_TYPE_HOST` in `.env`; vom Proxy geroutet) |
| Basis-URL | `{{ cookiecutter.ifdd_type_url }}` (`ifdd_type_url`, gemeinsam) |
| Image | `{{ cookiecutter.ifdd_nginx_image }}` (gemeinsam für schema, type und unit) |
| Inhalt | das Verzeichnis `IFDD_TYPE_CONTENT_DIR` in `.env`, meist ein Klon eines eigenen Inhalts-Repositorys |
| Netz | `{{ cookiecutter.ifdd_proxy_network }}` |
| Entwicklung | `http://127.0.0.1:{{ cookiecutter.ifdd_type_dev_port }}/` |

## Was die Instanz enthält

Die nginx-Konfiguration (`nginx/default.conf`, `nginx/mime.types`), zwei
Compose-Dateien, `.env` und Beispielinhalt in `examples/json-ld/`. Der
eigentliche Inhalt gehört nicht zur Instanz: Er liegt in einem eigenen
Repository, dessen Klon nur lesend eingebunden wird. `ifdd-de/schema`,
`ifdd-de/type` und `ifdd-de/unit` teilen diesen Aufbau.

## Vokabulare

Ein Namensraum ist ein Verzeichnis mit einem Dokument,
`<namespace>/index.jsonld`, das alle seine Vokabulare in `@graph` hält. nginx
beantwortet die Namensraum-IRI (`<namespace>/`) und jede IRI darunter
(`<namespace>/<name>`) mit diesem Dokument; ein Client findet seinen Knoten
über `@id`. So lässt sich eine Eigenschafts-IRI aus den Daten im Browser
öffnen und zeigt ihre Bedeutung, und ein Programm liest dasselbe JSON-LD.
Vorhandene Dateien werden unverändert ausgeliefert; eine IRI eines
unbekannten Namensraums antwortet 404. Die IRIs in den Dokumenten gleich den
URLs halten, unter denen sie ausgeliefert werden (unter
`{{ cookiecutter.ifdd_type_url }}`).

Die Beispiele nutzen nur Standard-Vokabulare: `rdfs:label` und
`rdfs:comment` je Sprache (`en`, `de`), `rdfs:range`, `rdfs:isDefinedBy`,
`sh:order` (Reihenfolge in der UI), `qudt:hasQuantityKind`,
`qudt:applicableUnit` (Einheiten des Dienstes unit,
`{{ cookiecutter.ifdd_unit_url }}`), `qudt:symbol` und `skos:exactMatch`.
Die UI zeigt das Label eines Namensraums als Titel seines Reiters, daher ist
es ein schlichter Name; Hinweise wie „Beispiel“ gehören in `rdfs:comment`.

## Architektur

```{mermaid}
flowchart LR
  Clients[Browser, Dienste] -->|HTTPS| Proxy[proxy: Traefik]
  Proxy -->|"{{ cookiecutter.ifdd_type_host }}, Port 8080"| Nginx[type: nginx]
  Nginx -->|nur lesend| Content[(Inhalts-Klon)]
  Repo[Inhalts-Repository] -->|git pull| Content
```

| Wer | Nutzt diesen Dienst für |
|---|---|
| ui | Beschriftungen, Hilfetexte und die Reihenfolge der Eigenschaften (im Browser, ursprungsübergreifend) |
| schema | die Namensräume seiner Kontexte zeigen hierher |
| Menschen und Programme | eine Eigenschafts-IRI aus den Daten, direkt geöffnet |

## Kapitel

```{toctree}
:maxdepth: 2

setup
build-release
operations
security
troubleshooting
reference
```

Die Werte der Variablen dieser Instanz und ihre Beziehungen zu den anderen
Diensten stehen im Kapitel über die Dienst-Instanzen
({ref}`ifdd-instances-static`).
