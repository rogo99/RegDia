23. Sept. 2026

Grundregeln

- Maßgeblich ist immer der Scan; ocr_text ist nur eine Lesehilfe. 
- Annotiert wird, was gedruckt ist, nicht was man erschließen kann. Was nicht dasteht, bleibt leer. 
- Unsichere Lesung: beste Lesung eintragen und unsicher: <Feld> in bemerkung schreiben. 
- Während der Annotation nicht in die Pipeline-Ausgabe schauen (Anker-Bias). 
- Eine Zeile pro Eintrag. Taucht dieselbe Person in zwei Rubriken auf (z. B. Hueterer unter Begräbnisse und Congregation), sind das zwei Zeilen. 
- Zusammensetzen von Feldern: Stehen die Angaben für ein Feld an verschiedenen Stellen im Eintrag, werden die Teile in der Reihenfolge des Drucks mit Komma und Leerzeichen verbunden. Beispiel: „Ein Junggeselle, Nahmens, Johann Georg Mergner, ein Kaufmannsdiener allhier“ → subject_occupation_verbatim: Junggeselle, ein Kaufmannsdiener allhier. Innerhalb eines zusammenhängenden Textstücks bleibt die gedruckte Zeichensetzung erhalten. Es werden keine Kommas ergänzt, wo der Druck „und“ oder gar nichts hat (Burger und Kramhändler allhier bleibt so). 

Schreibung

Diese Regeln gelten für alle Textfelder.
- Die Originalschreibung bleibt erhalten, also Beysitzer, Burger, Wittiber. 
- ſ wird zu s, Trennstriche am Zeilenende werden aufgelöst (Bran-teweinbrenner → Branteweinbrenner). 
- Abkürzungen bleiben wie gedruckt (Joh., Hochfürstl.). 
- Titel und Anreden (Hr., Tit., S.T., Jgfr., Frau) kommen nicht in Namensfelder, sondern in bemerkung.
Felder Demografie
- Die Spalten entsprechen Romans JSON-Schema; normalisierte Felder tragen den Wert ein, den die Pipeline liefern soll. 

| Feld | Regel                                                                                                                                                                                                                                          | Beispiel                       |
| --- |------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|--------------------------------|
| event | Nach der Rubrikenüberschrift, nicht nach dem Inhalt: Getaufete → baptised, Begrabene → buried, Gestorbene (Congregation, Bruderschaft) → deaths                                                                                                | buried                         |
| date_verbatim | Wie gedruckt                                                                                                                                                                                                                                   | 2.Jan. · 31 Dec. · Eod. die    |
| date_normalised | JJJJ-MM-TT. „Eod. die“ = Datum des vorherigen Eintrags; fehlender Monat = Monat des vorherigen Eintrags; Dezember-Einträge in einer Januar-Ausgabe = 1769; in JSON Datein: Nur Tag und Monat auf Englisch für Konsistenz mit den generierten Einträgen | 1770-01-02 (JSON: 2 January)   |
| subject_given_name | Alle Vornamen der Person, um die es geht (Täufling, Verstorbene/r)                                                                                                                                                                             | Septimus Andreas               |
| subject_family_name | Bei Täuflingen meist leer, in diesem Fall vom Vater übernehmen. Weibliche Formen wie gedruckt                                                                                                                          | Leidlin                        |
| subject_occupation_verbatim | Beruf und Stand des Subjekts vom Namen bis vor die Altersangabe, inkl. Stand-Wörter und Ortszusätze. Angaben des Vaters gehören nicht hierher                                                                                                  | Burger und Kramhändler allhier |
| rel_role | Höchstens eine Bezugsperson: „Vater, X“ oder „des X … Sohn/Tochter“ → father; „des X … Ehewirthin/Hausfrau/Wittib“ → husband; keine genannt → leer                                                                                             | father                         |
| rel_given_name / rel_family_name | Name der Bezugsperson wie gedruckt, ohne Titel                                                                                                                                                                                                 | Heinrich Paul / Oppermann      |
| rel_occupation_verbatim | Beruf und Stand der Bezugsperson, gleiche Regeln wie beim Subjekt                                                                                                                                                                              | Burger und Branteweinbrenner   |
| age_verbatim | Wie gedruckt, inkl. „alt“                                                                                                                                                                                                                      | 5 Jahr und 10 Monath alt       |
| age_days | Jahr = 365, Monat = 30, Woche = 7, Tag = 1; „weniger“ abziehen; „Viertel Jahr“ = 91; „27½ Jahr“ = 27,5 × 365. Unleserlich → leer + Notiz                                                                                                       | 2125                           |

Bei „ein uneheliches Kind“ nur den Namen eintragen und „unehelich“ in bemerkung schreiben.

Nicht annotieren

- Jahressummen
- Legitimationes
- Avertissements
- Preislisten
- Kopfzeilen bleiben außen vor.

Passagiere (optional)

- Nur annotieren, wenn nach der Demografie noch Zeit bleibt. 
- Eine Zeile pro Person. Reisen zwei zusammen („Hr. Pellegrini … und Mſr. Werno“), bekommen beide eine Zeile mit gleichem Datum und Transport. 
- Boten, Postwagen und Estafetten haben keinen Namen: Namensfelder leer, Beschreibung (Amberger ord. Bothe) in occupation_verbatim. 
- transport wie gedruckt, ohne „Per“: Calesch, Posta, Kutsche, Wagen, zu Fuß. 
- origin = Ort nach „von/aus“; lodging = Gasthof nach „log. in“ (schwarzen Bären). 
- time = Uhrzeit, falls genannt (Vormittags); s_value = nur die Zahl nach „ſ.“ in value, kompletter Ausdruck in verbatim.

Inter-Annotator-Agreement und Ablauf

- Beatrice & Roman annotieren alle Zeilen. 
- Jede Person arbeitet in einer eigenen Datei: gold_num02_beatrice.csv bzw. gold_num02_roman.csv. 
- Nicht absprechen, bevor beide fertig sind. 
- Agreement feldweise berechnen, bevor Unterschiede geklärt werden. 
- Unterschiede gemeinsam am Scan klären; das Ergebnis kommt in die finale Gold-Datei. 
- Die drei Zusatzausgaben (eine pro Jahresdrittel, per Zufall mit festem Seed gezogen) annotiert Beatrice nach denselben Regeln, nur Taufen und Begräbnisse. 
- In der Vorlage steht in Beispielzeile 1 noch Den 2.Jan.; nach dieser Richtlinie wird daraus 2.Jan..
