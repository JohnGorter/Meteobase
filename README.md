# Meteobase

Meteobase is een webapplicatie en -service die wordt gefinancierd door de Nederlandse waterschappen. Het project wordt in opdracht van de waterschappen aanbesteed, georganiseerd en gefaciliteerd door Het Waterschapshuis (https://www.hetwaterschapshuis.nl/).

#### Doel en doelgroep ####
De applicatie en service zijn in het leven geroepen om medewerkers van waterschappen en adviesbureaus in de watersector te ondersteunen bij het uitvoeren van modelstudies waarvoor meteorologische gegevens nodig zijn. De applicatie ontsluit Nederlandse historische neerslag- en verdampingsgegevens en presenteert die in een kant-en-klaar bestandsformaat ten behoeve van veelgebruikte modellen (SOBEK, SIMGRO, MODFLOW). Daarnaast ontsluit Meteobase statistische data omtrent neerslag in Nederland en heeft het een uitgebreide literatuursectie.

#### Ontwikkelteam ####
Bouw van de applicatie en beheer & onderhoud is vanaf de start in 2011 in handen van:

* Siebe Bosch en John (Hydroconsult, https://github.com/SiebeBosch)

met bijdragen van:

* Arend Ketelaar (GeoPro): front-end
* Maxim Bureac: front-end
* Daniël Tollenaar (D2Hydro, https://github.com/d2hydro): python-script regenduurlijnen
* Jacques Doeleman (Iconica, https://github.com/JacquesDIconica): front-end
* Stefan Koopmanschap (Ingewikkeld.net): implementatie cookies
* Mike Doornenbal en Niels Gorter (Iconica): migratie naar nieuwe server, december 2023

#### Architectuur ####
De architectuur bestaat uit:
* een front-end, bestaande uit:
  * de website, geschreven in PHP
  * Een python-script voor de in-app regenduurlijnen-applicatie en gebiedsreductie-applicatie
* een backend, bestaande uit:
  * Een negental executables, geschreven in VB.NET en bijgehouden in Visual Studio 2022 van Microsoft:
     * WIWBBASIS: levert meetreeksen van de neerslagstations van het KNMI: uursommen, etmaalsommen, neerslag en verdamping
     * WIWBRASTER2ASCI: levert geijkte radarneerslagsommen en ruimtelijk verdeelde verdamping in ASCII-rasterformaat
     * WIWBRASTERBYPOLY: levert geijke radarneerslag en ruimtelijk verdeelde verdamping, geaggregeerd naar polygonen
     * WIWBNATIVERASTER: levert de geijkte radarneerslag en ruimtelijk verdeelde verdamping in het oorspronkelijke formaat (HDF5)
     * WIWBSTEDELIJK: levert de stedelijke neerslaggebeurtenissen
     * WIWBSTOCHASTEN: levert kansen bij neerslagvolumes, voor huidig klimaat en diverse klimaatprojecties
     * WIWBTOETSING: levert langjarige klimaatgecorrigeerde neerslagreeksen voor diverse klimaatregio's in Nederland
     * WIWBHERHALINGSTIJD: genereert een kaart met de terugkeertijd in jaren voor de neerslag op gegeven dag en met gegeven duur
     * WIWBFEEDBACK: verstuurt de input van het feedbackformulier op de site naar info-at-meteobase.nl
     * WIWBANONYMIZE: een executable die op de server is gescheduled om iedere 1e van de maand alle logins ouder dan 1 maand te anonimiseren.
  * Een PostgreSQL-database t.b.v. opslag gebruiksstatistieken
  * Een Apache webserver
* Opslag van:
  * Literatuur
  * Langjarige reeksen met neerslagsommen (de zogeheten toetsingsreeksen)
  * Statistieken van neerslagvolumes
  * Stedelijke neerslaggebeurtenissen
  * De SATDATA 2.0 dataset (voorloper van de nu operationele SATDATA 3.0 dataset) 
* installatiescripts voor de executables, geschreven in Inno Script Studio

#### Herkomst van gegevens ####
Historische neerslag- en verdampingsgegevens worden door Meteobase betrokken van de WIWB-server via de WIWB-API (https://portal.hydronet.com/data/files/Technische%20Instructies%20WIWB%20API.pdf). We merken op dat deze API niet publiekelijk toegankelijk is. Een eigen implementatie van de applicatie of executables zal daarom alleen functioneren vanaf een IP-adres dat door de beheerder van WIWB is gewhitelist.

Toetsingsreeksen, neerslagstatistieken en stedelijke neerslaggebeurtenissen staan opgeslagen op de server van Meteobase zelf en worden rechtstreeks bevraagd door de daarvoor ontwikkelde executables. 

#### Installatie van de server ####
Een map met installatiebestanden van programma's die nodig zijn om een server in te richten voor het hosten van Meteobase staat onder deze Dropbox-link:
https://www.dropbox.com/sh/na6rjqd2wv1h4wq/AABxVYapjquKOfKcmHimZdfta?dl=0

#### AVG Compliance ####
Op de server van Meteobase worden iedere eerste van de maand automatisch alle registraties geanonimiseerd. Dit gebeurt door de naam te verwijderen uit de database en alles tot het apenstaartje te verwijderen van het opgegeven e-mailadres. De bedrijfsnaam blijft bestaan zodat Het Waterschapshuis statistieken kan opstellen van categorieën gebruikers. 

#### Permanente bestanden en literatuur ####
Op de server staat naast de applicatie zelf een groot aantal forse bestanden die niet in Gitub zijn ondergebracht. Het gaat onder meer om langjarige neerslagreeksen, de SATDATA 2.0-datasets en alle literatuur. Deze bestanden staan op de server in de directory meteobase\downloads\fixed\. Wij hebben ze eveneens beschikbaar gesteld via de volgende dropbox-link: https://www.dropbox.com/sh/mq409bff8y0v5xk/AACxMnYdCAQmiiX_SVcju0z-a?dl=0

#### keys en wachtwoorden ####
Om de applicatie operationeel te krijgen zijn de volgende wachtwoorden en keys benodigd:
* licentie op Gembox Spreadsheet (https://www.gemboxsoftware.com/spreadsheet)
* de connection-string voor de backend-database (SQLite) met neerslagstatistieken
* het wachtwoord voor de uitgaande mails die geautomatiseerd naar gebruikers worden gestuurd
* de host, poortnummer, username, wachtwoord voor de PostgreSQL-database met gebruikersstatistieken

All deze keys en wachtwoorden zijn opgeslagen in losse tekstbestanden in .gitignore en dus niet publiekelijk beschikbaar. Ze worden uitsluitend gedeeld met de eigenaar van Meteobase: Het Waterschapshuis. Mocht u zelf een werkende afgeleide van Meteobase willen vervaardigen, dan is het de eigen verantwoordelijkheid om een licentie op Gembox Spreadsheet te nemen, op de server een PostgreSQL-database te installeren en configureren en een IP-whitelisting aan te vragen bij de beheerder van WIWB (momenteel Hydrologic).

De verantwoordelijkheid voor het beheer en onderhoud van Meteobase ligt tot en met Q3 van 2023 bij Hydroconsult.





