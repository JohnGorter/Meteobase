# Plan: Repository-opschoning, kwaliteitsborging en beveiligingsaudit (JohnGorter/Meteobase)

**Status:** Concept ter review
**Opgesteld door:** Architect (<@U0C8R4UD1HT>)
**Trello:** https://trello.com/b/0jkoGZuw/meteobase
**Drive-map:** https://drive.google.com/drive/folders/1qiqNLqOu0IbqGntfciv81rpKo4Nc9AeP?hl=nl

---

## Scope

Dit plan dekt vier werkpakketten op de repository. Alle bevindingen zijn geverifieerd op de geclonede `main`-branch (10.731 getrackte bestanden, 153 commits).

| # | Werkpakket | Betrokken modules |
|---|---|---|
| A | Build-artefacten en binaire referenties uit git verwijderen | `backend/*/bin`, `backend/*/obj`, `backend/*/packages`, `backend/*/.vs`, `backend/References/` |
| B | Geautomatiseerde kwaliteitscontrole (CI) opzetten | `.github/workflows/` (nieuw), `frontend/meteobase/python/test/`, `frontend/meteobase/python/wiwb/test_stats_app.py`, PHP en VB.NET |
| C | Beveiligingsaudit van invoer en formulieren | `frontend/backoffice/DataLoad/*.php`, `local_config.example.php`, `check_input()` |
| D | Opschonen van dode code en documentatie | `backend/*/Backup/` (44 bestanden), `docs/`, `README.md` |

### Geverifieerde bevindingen

| Onderwerp | Bevinding |
|---|---|
| Artefacten | 8.383 van 10.731 getrackte bestanden zitten in `bin/`, `obj/`, `packages/` of `.vs/` van de VB.NET-projecten. De `.gitignore` bevat wel `.vs/` en `*.exe`-regels, maar de bestaande bestanden blijven getrackt. |
| Referenties | `backend/References/` bevat MapWinGIS (incl. gdal-data en PROJ_NAD), Gembox, NpgSQL en Ionic. |
| Backups | 44 bestanden in `Backup/`-mappen (o.a. `WIWBFEEDBACK/Backup`, `WIWBSTEDELIJK/Backup/MBSTOCHASTEN`, `WIWBSTOCHASTEN/Backup`, `WIWBTOETSING/WIWBTOETSING/Backup`). |
| CI | `.github/` bevat alleen `CODEOWNERS`. Er is geen workflow. |
| Tests | Python: `frontend/meteobase/python/test/testwiwb.py`, `debug_requests.py` en `frontend/meteobase/python/wiwb/test_stats_app.py`. Er zijn geen PHP- of VB.NET-tests. |
| Superglobals | 25 PHP-bestanden gebruiken `$_POST`/`$_GET`/`$_REQUEST`. `check_input()` wordt in 12 bestanden gebruikt. |
| Onvalidatede invoer | `ScriptProcessReferenceL.php` (r. 34-37), `ScriptProcessReferenceH.php` (r. 31-34) en `ScriptProcessMakkink.php` (r. 31-34) lezen `$_POST` direct in variabelen. |
| Geheimen | `email.txt` wordt in de InnoSetup-scripts van 5 backend-executables gebruikt (`#define emailpassword`). Er is nog geen historie-scan uitgevoerd. |
| Documentatie | `docs/` bevat alleen `server-config/httpd.conf` en `php.ini`. De README beschrijft de architectuur, maar mist een lokale opzet en build-/releasehandleiding. |

---

## Dependencies

- Toegang tot de repository met schrijfrechten (PAT van JohnGorter) voor de branch- en PR-workflow.
- Akkoord van de repository-eigenaar op het verwijderen van getrackte artefacten en de `Backup/`-mappen. Dit is niet terug te draaien zonder de historie.
- Bevestiging van de licentie-status van `backend/References/` (MapWinGIS, Gembox, NpgSQL, Ionic). Zonder dit besluit wordt pakket A niet uitgevoerd voor die map.
- Een Windows-runner (of self-hosted runner) voor het bouwen van de VB.NET-projecten (pakket B, fase 2).
- Inventarisatie van wat er op de server staat (`local_config.php`, `email.txt`) voordat geheimen worden geroteerd.

---

## Implementation Steps

### Pakket A: Artefacten opschonen (eerst, aparte PR)

1. Maak een branch `chore/untrack-build-artifacts` aan vanaf `main`.
2. Verwijder uit de index, zonder bestanden lokaal te wissen: `git rm -r --cached` voor alle paden die matchen op `bin/`, `obj/`, `packages/` en `.vs/` onder `backend/`. Controleer het resultaat met `git ls-files | grep -E '(^|/)(bin|obj|packages|\.vs)/'` (verwacht: 0).
3. Pas `.gitignore` aan zodat de regels generiek zijn: `**/bin/`, `**/obj/`, `**/packages/`, `**/.vs/`. Controleer dat de bestaande regels `backend/*/InnoSetup/*.exe` blijven werken.
4. Voer een aparte beslissing uit voor `backend/References/`. Alternatief A: alleen verplaatsen naar een externe opslag, met een `References/README.md` die de herkomst beschrijft. Alternatief B: laten staan tot de licentiecheck is afgerond.
5. Verwijder de 44 bestanden in `Backup/`-mappen in een aparte commit, zodat de wijziging makkelijk terug te draaien is.
6. Controleer met een schone build (VS 2022, Release) dat alle negen executables nog bouwen zonder de verwijderde mappen. Controleer ook de InnoSetup-scripts.

### Pakket B: Kwaliteitscontrole (fase 1 eerst, daarna fase 2)

7. Maak `.github/workflows/ci.yml` aan met drie jobs:
   - `python-tests`: installeer de dependencies uit de Python-modules en draai `pytest` op `frontend/meteobase/python/test/` en `frontend/meteobase/python/wiwb/test_stats_app.py`. Let op: `testwiwb.py` en `debug_requests.py` kunnen netwerk of data nodig hebben. Markeer die als `integration` en laat ze buiten de standaardrun.
   - `php-lint`: draai `php -l` op alle `*.php`-bestanden onder `frontend/` (matrix op PHP-versie die de server gebruikt; zie `docs/server-config/php.ini`).
   - `dotnet-build` (fase 2): bouw de VB.NET-solutions op een `windows-latest`-runner. Gebruik het `.sln`-bestand per module.
8. Zet branch protection op `main`: de jobs zijn verplicht voor merge.
9. Schrijf na fase 1 een test voor één `check_input()`-scenario in PHP (bijv. met PHPUnit), zodat er ook PHP-dekking is.

### Pakket C: Beveiligingsaudit (parallel aan B, aparte PR per bestand)

10. Inventariseer alle `$_POST`-, `$_GET`- en `$_REQUEST`-toegang in de 25 PHP-bestanden. Maak een tabel met bestand, regel, veld, validatie ja/nee. Lever die als bijlage bij de PR.
11. Verplaats alle directe toegang naar één centrale functie in de bestaande `check_input()`-helper (of een opvolger met een schema per veld). Begin met `ScriptProcessReferenceL.php`, `ScriptProcessReferenceH.php` en `ScriptProcessMakkink.php`.
12. Controleer alle database-aanroepen. Gebruik `pg_prepare` en `pg_execute`, of de PDO-equivalent, en geen string-concatenatie met gebruikersinvoer. Dit moet volgens de recente commits (`9fae179`, `8fac561`) al grotendeels het geval zijn; de audit bevestigt dat per bestand.
13. Controleer de CSRF- en honeypot-dekking op alle POST-formulieren onder `frontend/backoffice/`.
14. Draai een geheimenscan op de volledige historie met `gitleaks detect --log-opts="--all"`. Bevindingen gaan naar een aparte security-issue, niet in een publieke PR-beschrijving.
15. Verplaats de `email.txt`-afhankelijkheid van de InnoSetup-scripts naar een documentatiestap: welk bestand lokaal moet bestaan, en welke waarden erin staan. Voeg geen geheimen toe aan de repository.

### Pakket D: Documentatie en structuur

16. Voeg `docs/architectuur.md` toe met een overzicht van frontend (PHP, Python), backend-executables (VB.NET), PostgreSQL en SQLite. Gebruik de bestaande README-tekst als bron.
17. Voeg `docs/lokaal-draaien.md` toe met de stappen om lokaal te starten: `local_config.php` aanmaken op basis van `local_config.example.php`, de Python-omgeving en de database.
18. Voeg `docs/build-en-release.md` toe met het bouwen van de executables en het maken van de InnoSetup-installers.
19. Verwijs in de README naar de drie documenten. Vervang de architectuursectie niet volledig; voeg alleen links toe.

---

## Testing Requirements

- **Pakket A:** Geen unit-tests. Verificatie is een schone build van alle negen VB.NET-executables en het succesvol compileren van de InnoSetup-scripts. Controleer met `git ls-files` dat het aantal artefacten 0 is.
- **Pakket B:** De CI-workflow moet groen zijn op de PR. Test dat een opzettelijk kapotte PHP-bestand door `php-lint` wordt afgewezen (één keer, lokaal of in een testbranch).
- **Pakket C, unit:** Voor elke functie die invoer valideert: test geldige waarden, lege waarden, te lange waarden en ongeldige types. Dit zijn gewone `TestCase`-tests, geen Gherkin, omdat het server-side logica betreft zonder functionele acceptatiecriteria.
- **Pakket C, integratie:** Voor de drie gewijzigde DataLoad-scripts: een POST met ontbrekende velden krijgt een foutmelding en wordt niet naar de database doorgestuurd. Een POST met geldige velden slaat op.
- **Dekking:** Pakket C moet alle `$_POST`-regels in de drie gewijzigde bestanden dekken. Controleer dat de gewijzigde regels na afloop 0 directe superglobal-toegang hebben.
- **Regressie:** Draai de bestaande Python-tests vóór en na elke wijziging. Resultaten moeten gelijk blijven.

---

## Risks & Considerations

| Risico | Impact | Mitigatie |
|---|---|---|
| Verwijderen van `bin/`, `obj/` en `packages/` breekt een build die stilzwijgend afhankelijk is van die bestanden (bijv. NuGet-pakketten zonder restore). | Hoog | Eerst een schone build op een aparte machine. Pas `git rm --cached` toe in een aparte PR. Herstel is mogelijk via de historie. |
| `backend/References/` bevat licentiegebonden bestanden (MapWinGIS, Gembox). | Hoog | Licentiecheck vóór verwijdering. Geen verwijdering zonder akkoord van de eigenaar. |
| Historie-herschrijving (BFG, `git filter-repo`) om geheimen te verwijderen. | Hoog | Alleen na bevestiging van de bevindingen. Alle clones en forks moeten opnieuw worden gekloond. Roteer eerst de geheimen. |
| CI-jobs die netwerk of data vereisen (`testwiwb.py`). | Middel | Markeer deze tests als `integration` en draai ze niet standaard. |
| Strengere validatie kan bestaande formulieren breken. | Middel | Valideer in fases. Test de backoffice-formulieren handmatig na elke PR. |
| Windows-runner voor VB.NET is duurder en trager. | Laag | Fase 2. Eerst fase 1 (Python en PHP) laten draaien. |
| Dit plan verandert geen productiecode buiten de genoemde bestanden. | - | Elke wijziging is een aparte PR met een duidelijke scope. |

---

## Follow-up

- Na pakket A: bijwerken van de README-sectie over de repository-structuur en het bouwen.
- Na pakket B fase 2: een badge in de README voor de CI-status.
- Na pakket C: een ADR (`docs/adr/`) waarin de gekozen validatiestrategie wordt vastgelegd.
- Periodieke controle van `gitleaks` in de CI (wekelijks of op elke PR).
- Bekijk na afronding of de `frontend/meteobase/python/`-structuur (`mbase_api` en `wiwb`) met de backend-executables kan worden samengevoegd tot één duidelijke interface.
- Eigenaar van het plan: JohnGorter. Uitvoering: Developer-agent per pakket, daarna Reviewer-agent.
