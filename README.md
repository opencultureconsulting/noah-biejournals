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

## Mapping

Die Ausgangsdaten werden wie folgt in METS/MODS übertragen:

Generell:

* id: [vierstellige Nummer aus OAI identifier(config/harvest.ini)] mit [Präfix](config/02-id.json) `biejournals_`

MODS:

* titleInfo@lang: 
* titleInfo@type: 
* titleInfo/title: 
* titleInfo/nonsort: 
* titleInfo/subTitle: 
* name@type: `personal` (wenn name/displayForm vorhanden)
* name@authority=gnd@valueURI: 
* name/displayForm: 
* name/namePart@type=family: 
* name/namePart@type=given: 
* name/nameIdentifier@type=orcid
* name/role/roleTerm@authority=marcrelator: 
* [typeOfResource](config/template.txt): `text`
* genre@authority=dini: 
* originInfo/dateIssued: 
* originInfo/dateIssued@keyDate: 
* originInfo/dateOther: 
* originInfo/dateOther@keyDate: 
* language/languageTerm@authority=iso639-2b: 
* physicalDescription/extent@unit=page: 
* abstract: 
* abstract@lang: 
* note: 
* note@type: 
* subject@lang: 
* subject/topic: 
* classification@authority=ddc: 
* classification@authority=ioo@displayLabel: 
* relatedItem@type: `host` (wenn relatedItem/titleInfo/title vorhanden)
* relatedItem/titleInfo/title: 
* relatedItem/part/detail@type=volume/number: 
* relatedItem/part/detail@type=issue/number: 
* relatedItem/part/extent@unit=page/start: 
* relatedItem/part/extent@unit=page/end: 
* identifier@type=urn: 
* identifier@type=doi: 
* identifier@type=sys: 
* accessCondition: 
* accessCondition@xlink:href: 
* extension/vl:doctype: 
* recordInfo/recordIdentifier: wie id (s.o.)

METS:

* [fileSec/fileGrp@USE](config/template.txt): `pdf upload` oder `generic file` abhängig von Dateiendung in URL
* fileSec/fileGrp/file@MIMETYPE: 
* fileSec/fileGrp/file/Flocat@xlink:href: 
* structMap/div@LABEL: 
