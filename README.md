# Harvesting und Transformation der Daten des Zeitschriftenservers BieJournals für das Portal noah.nrw

Dieser Workflow harvestet die Daten des Zeitschriftenservers [BieJournals](https://www.biejournals.de/) der UB Bielefeld und transformiert diese in METS/MODS für das Portal [noah.nrw](https://noah.nrw/).

## Systemvoraussetzungen

- GNU/Linux (getestet mit Fedora 32)
- JAVA 8+ (für OpenRefine)
- PHP 7.3+ und Composer (für VuFindHarvest)
- [go-task](https://github.com/go-task/task) 3.0.0+
- xmllint (libxml2)

## Workflow

Der Workflow wird in [Taskfile](Taskfile.yml) definiert und kann entweder lokal oder mit GitHub Actions ausgeführt werden.

Der erste task `harvest` lädt die öffentlichen Datensätze der auf BieJournals gehosteten Zeitschriften über OAI-PMH-Schnittstellen im Format Dublin Core. Das Ergebnis sind XML-Dateien im Verzeichnis [input](input).

Der zweite Task `transform` transformiert die heruntergeladenen Daten in METS/MODS. Das Ergebnis sind XML-Dateien im Verzeichnis [output](output).

Beide Tasks verwenden einen Cache, um nur neue Daten abzurufen bzw. zu verarbeiten. Über OAI bekanntgemachte Löschungen werden berücksichtigt. Der Task `reset` führt bei Bedarf ein vollständiges Harvesting inklusive Transformation aus.

## Mapping

Die Ausgangsdaten werden wie folgt in METS/MODS übertragen:

Generell:

* id: [vierstellige Nummer aus OAI identifier(config/harvest.ini)] mit [Präfix](config/02-id.json) `biejournals_`

MODS:

* [titleInfo@lang](config/04-titleInfo.json): `dc:title@xml:lang` mit Mapping auf iso693-2b
* [titleInfo@type](config/04-titleInfo.json): `translated` wenn `dc:language` = `deu` und `dc:title@xml:lang` nicht `de-DE`
* [titleInfo/title](config/04-titleInfo.json): 1. Teil von `dc:title` vor Trennzeichen `: `, Artikel am Anfang entfernen und Doppelpunkt am Ende entfernen
* [titleInfo/nonsort](config/04-titleInfo.json): Artikel am Anfang von `dc:title`
* [titleInfo/subTitle](config/04-titleInfo.json): 2. Teil von `dc:title` nach Trennzeichen `: ` und Doppelpunkt am Ende entfernen
* [name@type](config/04-name.json): `personal` (wenn name/displayForm vorhanden)
* [name@authority=gnd@valueURI: -
* [name/displayForm](config/04-name.json): `dc:creator` wenn es `,` enthält
  * 2004, 2018, 2034, 2035, 2084: `-, -` entfernen
  * 3163: `Natalia Garcia Cervantes` ändern in `Cervantes, Natalia Garcia`
  * 3479: `Elke A. Gornik` ändern in `Gornik, Elke A.`
* [name/namePart@type=family](config/04-name.json): 1. Teil von `name/displayForm` vor Trennzeichen `, `
* [name/namePart@type=given](config/04-name.json): 2. Teil von `name/displayForm` nach Trennzeichen `, `
* [name/nameIdentifier@type=orcid: -
* [name/role/roleTerm@authority=marcrelator](config/04-name.json): `ctb` (wenn name/displayform vorhanden)
* [typeOfResource](config/template.txt): `text`
* [genre@authority=dini](config/04-genre.json): `article`
* [originInfo/dateIssued](config/04-originInfo.json): `dc:date`
* [originInfo/dateIssued@keyDate](config/04-originInfo.json): `yes` (wenn `dc:date` vorhanden)
* originInfo/dateOther: -
* originInfo/dateOther@keyDate: -
* [language/languageTerm@authority=iso639-2b](config/04-language.json): `dc:language` mit Mapping auf iso693-2b
* [physicalDescription/extent@unit=page](config/04-physicalDescription.json): Seitenzahlen extrahieren aus Literaturangabe in `dc:source` und Differenz berechnen
* [abstract](config/04-abstract.json): `dc:description`
* [abstract@lang](config/04-abstract.json): `dc:description@xml:lang` mit Mapping auf iso693-2b
* note: -
* note@type: -
* subject@lang: 
* subject/topic: 
* classification@authority=ddc: -
* classification@authority=ioo@displayLabel: -
* relatedItem@type: `host` (wenn relatedItem/titleInfo/title vorhanden)
* relatedItem/titleInfo/title: 
* relatedItem/originInfo/publisher: (nur der erste Wert)
* relatedItem/identifier@type=issn: (nur der erste Wert, ohne Bindestrich)
* relatedItem/part/detail@type=volume/number: 
* relatedItem/part/detail@type=issue/number: 
* relatedItem/part/extent@unit=page/start: 
* relatedItem/part/extent@unit=page/end: 
* identifier@type=urn: -
* identifier@type=doi: 
* identifier@type=sys: 
* accessCondition: 
* accessCondition@xlink:href: 
* extension/vl:doctype: `oaArticle`
* recordInfo/recordIdentifier: wie id (s.o.)

METS:

* [fileSec/fileGrp@USE](config/04-fileSec.json): `pdf upload` für das erste Vorkommen von `application/pdf` in `dc:format`; `generic file` für alle anderen Links (beginnend mit `http`) aus `dc:relation`
  * Datensätze ohne Direktlink auf ein PDF (d.h. kein `application/pdf` in `dc:format`) löschen
* [fileSec/fileGrp/file@MIMETYPE](config/04-fileSec.json): `dc:format`
* [fileSec/fileGrp/file/Flocat@xlink:href](config/04-fileSec.json): Links (beginnend mit `http`) aus `dc:relation` und `/view/` durch `/download/` ersetzen
* structMap/div@LABEL: -
