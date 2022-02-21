# Harvesting und Transformation der Daten des Zeitschriftenservers BieJournals für das Portal noah.nrw

Dieser Workflow harvestet die Daten des Zeitschriftenservers [BieJournals](https://www.biejournals.de/) der UB Bielefeld im Format Dublin Core und transformiert diese in METS/MODS für das Portal [noah.nrw](https://noah.nrw/).

Die Daten in diesem Repository werden [alle 24 Stunden nachts ab 03:21 Uhr](https://github.com/opencultureconsulting/noah-biejournals/blob/main/.github/workflows/default.yml#L6) aktualisiert.

## Systemvoraussetzungen

- GNU/Linux (getestet mit Fedora 32)
- JAVA 8+ (für OpenRefine)
- PHP 7.3+ und Composer (für VuFindHarvest)
- [go-task](https://github.com/go-task/task) 3.10.0+
- xmllint (libxml2)

## Workflow

Der Workflow wird in [Taskfile](Taskfile.yml) definiert und kann entweder lokal (`task default`) oder mit [GitHub Actions](.github/workflows/) ausgeführt werden. Er besteht aus zwei Hauptbestandteilen:

1. Der Task `harvest` lädt die öffentlichen Datensätze der auf BieJournals gehosteten Zeitschriften über OAI-PMH-Schnittstellen im Format Dublin Core. Das Ergebnis sind XML-Dateien im Verzeichnis [input](input).

2. Der Task `transform` transformiert die heruntergeladenen Daten in METS/MODS. Das Ergebnis sind XML-Dateien im Verzeichnis [output](output).

Beide genannten Tasks verwenden einen Cache, um nur neue Daten abzurufen bzw. zu verarbeiten. Über OAI bekanntgemachte Löschungen werden berücksichtigt. Der Task `reset` führt bei Bedarf ein vollständiges Harvesting inklusive Transformation aus.

## Mapping

Die Ausgangsdaten werden wie folgt in METS/MODS übertragen:

Generell:

* id: [Nummer aus OAI identifier](config/harvest.ini) mit [Präfix](config/02-id.json) `biejournals_`

MODS:

* [titleInfo@lang](config/04-titleInfo.json): **dc:title@xml:lang** mit Mapping auf iso693-2b
* [titleInfo@type](config/04-titleInfo.json): `translated` wenn **dc:language** = `deu` und **dc:title@xml:lang** nicht `de-DE`
* [titleInfo/title](config/04-titleInfo.json): 1. Teil von **dc:title** vor Trennzeichen `: `, Artikel am Anfang entfernen und Doppelpunkt am Ende entfernen
* [titleInfo/nonsort](config/04-titleInfo.json): Artikel am Anfang von **dc:title**
* [titleInfo/subTitle](config/04-titleInfo.json): 2. Teil von **dc:title** nach Trennzeichen `: ` und Doppelpunkt am Ende entfernen
* [name@type](config/04-name.json): `personal` (wenn **name/displayForm** vorhanden)
* ~~name@authority=gnd@valueURI~~
* [name/displayForm](config/04-name.json): **dc:creator** wenn es `,` enthält
  * 2004, 2018, 2034, 2035, 2084: `-, -` entfernen
  * 3163: `Natalia Garcia Cervantes` ändern in `Cervantes, Natalia Garcia`
  * 3479: `Elke A. Gornik` ändern in `Gornik, Elke A.`
* [name/namePart@type=family](config/04-name.json): 1. Teil von **name/displayForm** vor Trennzeichen `, `
* [name/namePart@type=given](config/04-name.json): 2. Teil von **name/displayForm** nach Trennzeichen `, `
* ~~name/nameIdentifier@type=orcid~~
* [name/role/roleTerm@authority=marcrelator](config/04-name.json): `ctb` (wenn **name/displayform** vorhanden)
* [typeOfResource](config/template.txt): `text`
* [genre@authority=dini](config/04-genre.json): `article`
* [originInfo/dateIssued](config/04-originInfo.json): **dc:date**
* [originInfo/dateIssued@keyDate](config/04-originInfo.json): `yes` (wenn **dc:date** vorhanden)
* ~~originInfo/dateOther~~
* ~~originInfo/dateOther@keyDate~~
* [language/languageTerm@authority=iso639-2b](config/04-language.json): **dc:language** mit Mapping auf iso693-2b
* [physicalDescription/extent@unit=page](config/04-physicalDescription.json): Seitenzahlen extrahieren aus Literaturangabe in **dc:source** und Differenz berechnen
* [abstract](config/04-abstract.json): **dc:description**
* [abstract@lang](config/04-abstract.json): **dc:description@xml:lang** mit Mapping auf iso693-2b
* ~~note~~
* ~~note@type~~
* [subject@lang](config/04-subject.json): **dc:subject@xml:lang** mit Mapping auf iso693-2b
* [subject/topic](config/04-subject.json): **dc:subject** mit Trennzeichen `,` sowie `;` aufteilen
* ~~classification@authority=ddc~~
* ~~classification@authority=ioo@displayLabel~~
* [relatedItem@type](config/04-relatedItem.json): `host` (wenn **relatedItem/titleInfo/title** vorhanden)
* [relatedItem/titleInfo/title](config/04-relatedItem.json): 1. Teil von **dc:source** mit Trennzeichen `;` (mit Filter **dc:source@xml:lang** auf `de-DE`)
* [relatedItem/originInfo/publisher](config/04-relatedItem.json): **dc:publisher**
* [relatedItem/identifier@type=issn](config/04-relatedItem.json): Werte aus **dc:source** im Format 1234-5678 und davon nur den ersten Wert und Bindestrich entfernen
* [relatedItem/part/detail@type=volume/number](config/04-relatedItem.json): 2. Teil von **dc:source** mit Trennzeichen `;` und daraus Nummer die auf `Bd. ` folgt (mit Filter **dc:source@xml:lang** auf `de-DE`)
* [relatedItem/part/detail@type=issue/number](config/04-relatedItem.json): 2. Teil von **dc:source** mit Trennzeichen `;` und daraus Nummer die auf `Nr. ` folgt (mit Filter **dc:source@xml:lang** auf `de-DE`)
* [relatedItem/part/extent@unit=page/start](config/04-relatedItem.json): Letzter Teil von **dc:source** mit Trennzeichen `;` und daraus Teil 1 (Nummer) mit Trennzeichen `-` (mit Filter **dc:source@xml:lang** auf `de-DE`)
* [relatedItem/part/extent@unit=page/end](config/04-relatedItem.json):  Letzter Teil von **dc:source** mit Trennzeichen `;` und daraus Teil 2 (Nummer) mit Trennzeichen `-` (mit Filter **dc:source@xml:lang** auf `de-DE`)
* ~~identifier@type=urn~~
* [identifier@type=doi](config/04-identifier.json): DOIs (beginnend mit 10.) aus **dc:identifier**
* ~~identifier@type=sys~~
* [accessCondition](config/04-accessCondition.json): Für `creativecommons.org` in **dc:rights** den Canonical Name aus der URL extrahieren; für `dppl` in **dc:rights** `Digital Peer Publishing Licence (v3)`
* [accessCondition@xlink:href](config/04-accessCondition.json): URL aus **dc:rights** (mit Filter `creativecommons.org` oder `dppl`)
* [extension/vl:doctype](config/04-extension.json): `oaArticle`
* recordInfo/recordIdentifier: wie **id** (s.o.), Umsetzung [direkt im Template](config/template.txt)

METS:

* [fileSec/fileGrp@USE](config/04-fileSec.json): `pdf upload` für das erste Vorkommen von `application/pdf` in **dc:format**; `generic file` für alle anderen Links (beginnend mit `http`) aus **dc:relation**
  * Datensätze ohne Direktlink auf ein PDF (d.h. kein `application/pdf` in **dc:format**) löschen
* [fileSec/fileGrp/file@MIMETYPE](config/04-fileSec.json): **dc:format**
* [fileSec/fileGrp/file/Flocat@xlink:href](config/04-fileSec.json): Links (beginnend mit `http`) aus **dc:relation** und `/view/` durch `/download/` ersetzen
* ~~structMap/div@LABEL~~
