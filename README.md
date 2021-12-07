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

(folgt)
