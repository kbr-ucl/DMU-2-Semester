# Lægehuset DK – Opgaver 02–07

## Oversigt

Opgaverne bygger trinvist videre på **Opgave 01** og fører løsningen fra en isoleret domænemodel til en komplet Clean Architecture-applikation med Blazor SSR-frontend.

| Opgave | Fokusområde | Nyt lag / teknologi |
|--------|-------------|---------------------|
| 01 | Domæneobjekter og TDD *(allerede udleveret)* | Domain |
| 02 | DDD-entiteter, Value Objects og Domain Services | Domain (udvidet) |
| 03 | Use Case-laget og CQS-commands | Use Case |
| 04 | Facade-laget – kontrakt, interfaces og DTO'er | Facade |
| 05 | Infrastructure-laget – EF Core og Query-handlers | Infrastructure |
| 06 | Dependency Injection og projektsammensætning | Composition Root |
| 07 | Blazor SSR-frontend der anvender Facaden | Blazor UI |

---

## Referencer

- Casebeskrivelse: [Lægehuset-dk.md](Lægehuset-dk.md)
- Arkitekturdokument: [Clean-Architecture.md](Clean-Architecture.md)
- Forrige opgave: [Lægehuset-dk-opgave-01.md](Lægehuset-dk-opgave-01.md)

---

---

# Opgave 02 – DDD-modellering af domænet

## Formål

Formålet er at modellere Lægehuset DK's domæne med DDD-principper: entities med private setters og eksplicitte metoder, Value Objects som records og en Domain Service til overlapskontrol.

## Afgrænsning

- Domænet skal stadig kunne kompilere og testes **uden** database, web-framework eller andre eksterne afhængigheder.
- Der gemmes stadig ikke data – alt lever i hukommelsen under test.

## Funktionelle krav

### Entities

1. **Konsultationsaftale** (entity)
   - Alle egenskaber har **private setters**.
   - Oprettes via konstruktor med `Patient`, `Læge`, `Konsultationstype` og starttidspunkt. Konstruktøren sikrer at objektet altid er i en gyldig tilstand.
   - Sluttidspunktet beregnes automatisk ud fra konsultationstypens varighed.
   - Skal have en `Status`-egenskab (enum: `Planlagt`, `Ankommet`, `Igangværende`, `Afsluttet`, `Aflyst`).
   - Metode: `ÆndrKonsultationstype(Konsultationstype nyType)` – må kun ske hvis status er `Planlagt`.
   - Metode: `MarkerAnkommet()` – ændrer status fra `Planlagt` til `Ankommet`.
   - Metode: `Aflys()` – ændrer status til `Aflyst`, men kun hvis status er `Planlagt` eller `Ankommet`.
   - Metode: `Afslut(string notat)` – kræver et ikke-tomt notat.
   - Alle metoder der ændrer tilstand skal kaste en `DomainException` ved ugyldige tilstandsovergange.

2. **Patient** (entity)
   - Egenskaber: `Navn`, `CPR`, `Telefon`, `Email`.
   - CPR skal valideres i konstruktøren (10 cifre).

3. **Læge** (entity)
   - Egenskaber: `Navn`, `Speciale`.

4. **Konsultationstype** (entity)
   - Egenskaber: `Navn`, `Varighed` (TimeSpan).
   - Standardtyper: Almindelig konsultation (20 min), Vaccination (10 min), Receptfornyelse (10 min), Rådgivning (15 min).

### Value Objects

5. **Tidsinterval** (value object – `record`)
   - Egenskaber: `Fra` (DateTime), `Til` (DateTime).
   - Validering: `Til` skal være efter `Fra`.
   - Beregnet egenskab: `Varighed` (TimeSpan).

### Domain Service

6. **KonsultationsoverlapService**
   - Metode: `HarOverlap(Konsultationsaftale ny, IEnumerable<Konsultationsaftale> eksisterende)` – returnerer `true` hvis den nye aftale overlapper med en eksisterende (husk den faste buffer på 5 minutter).
   - Aflyst-aftaler skal ignoreres i overlapskontrol.

### DomainException

7. En custom exception-klasse `DomainException` der arver fra `Exception`.

## Non-funktionelle krav

- Alle egenskaber skal have **private setters** (DDD-princip).
- Tilstandsændringer sker **kun** via eksplicitte metoder.
- Value Objects implementeres som **C# records**.
- Buffertiden (5 min) skal være en **systemindstilling** – ikke en del af konsultationstypen.

## Proceskrav

- TDD: Skriv tests der afdækker alle tilstandsovergange og edge cases **før** implementering.
- Der forventes som minimum tests for:
  - Oprettelse af konsultationsaftale med korrekt beregnet sluttidspunkt.
  - Alle statusovergange (gyldige og ugyldige).
  - Overlapskontrol med og uden buffer.
  - Validering af CPR, tidsinterval og konsultationstype.

## Hint

- `Tidsinterval` kan med fordel bruges både i `Konsultationsaftale` og i `KonsultationsoverlapService`.
- Buffertiden kan modelleres som en konstant eller via en konfigurationsklasse i domænet.

---

---

# Opgave 03 – Use Case-laget (Commands)

## Formål

Formålet er at oprette Use Case-laget, der orkestrerer applikationslogikken. Use Cases modtager DTO'er, materialiserer domæneobjekter, udfører forretningslogik via domænets metoder og persisterer resultatet via repository-interfaces.

## Afgrænsning

- Kun **commands** (skrivninger) – queries kommer i opgave 04/05.
- Repository-interfaces defineres i dette lag, men **implementeres ikke** endnu.
- Use Cases returnerer `Task` (void) – de ændrer tilstand, men returnerer ikke data (CQS).

## Funktionelle krav

### Repository-interfaces

1. **IKonsultationsaftaleRepository**
   - `Task<Konsultationsaftale?> HentAsync(Guid id)`
   - `Task<IReadOnlyList<Konsultationsaftale>> HentForLægeAsync(Guid lægeId, DateTime dato)`
   - `Task TilføjAsync(Konsultationsaftale aftale)`
   - `Task GemAsync()`

2. **IPatientRepository**
   - `Task<Patient?> HentAsync(Guid id)`
   - `Task TilføjAsync(Patient patient)`
   - `Task GemAsync()`

3. **ILægeRepository**
   - `Task<Læge?> HentAsync(Guid id)`
   - `Task<IReadOnlyList<Læge>> HentAlleAsync()`

4. **IKonsultationstypeRepository**
   - `Task<Konsultationstype?> HentAsync(Guid id)`
   - `Task<IReadOnlyList<Konsultationstype>> HentAlleAsync()`

### Use Cases

5. **OpretKonsultationsaftaleUseCase**
   - Modtager: `OpretKonsultationsaftaleRequest` (patientId, lægeId, konsultationstypeId, starttidspunkt).
   - Henter patient, læge, konsultationstype via repositories.
   - Henter eksisterende aftaler for lægen på den pågældende dag.
   - Anvender `KonsultationsoverlapService` til at sikre, at der ikke er overlap.
   - Opretter `Konsultationsaftale` via domænekonstruktøren.
   - Persisterer via repository.

6. **ÆndrKonsultationstypeUseCase**
   - Modtager: `ÆndrKonsultationstypeRequest` (konsultationsaftaleId, nyKonsultationstypeId).
   - Henter aftale og ny konsultationstype.
   - Kalder `ÆndrKonsultationstype()` på domæneobjektet.
   - Persisterer.

7. **AflysKonsultationsaftaleUseCase**
   - Modtager: `AflysKonsultationsaftaleRequest` (konsultationsaftaleId).
   - Kalder `Aflys()` på domæneobjektet.
   - Persisterer.

8. **MarkerAnkommetUseCase**
   - Modtager: `MarkerAnkommetRequest` (konsultationsaftaleId).
   - Kalder `MarkerAnkommet()` på domæneobjektet.
   - Persisterer.

9. **AfslutKonsultationsaftaleUseCase**
   - Modtager: `AfslutKonsultationsaftaleRequest` (konsultationsaftaleId, notat).
   - Kalder `Afslut(notat)` på domæneobjektet.
   - Persisterer.

### Exceptions

10. Opret en `NotFoundException` der kastes når et repository returnerer `null`.

## Non-funktionelle krav

- Hvert Use Case har **én public metode**: `Task Udfør(…Request request)`.
- Use Cases indeholder **ikke** forretningslogik – de delegerer til domænets metoder.
- Request-DTO'er bruger **primitive typer** (Guid, string, DateTime).

## Proceskrav

- TDD med **Moq** til at mocke repository-interfaces.
- Test at Use Cases kalder de rigtige domæne-metoder og repository-metoder.
- Test at `NotFoundException` kastes ved manglende entiteter.
- Test at overlapskontrol forhindrer dobbeltbooking.

## Hint

- `KonsultationsoverlapService` kan injiceres i `OpretKonsultationsaftaleUseCase` som en dependency.
- Request-DTO'erne kan defineres midlertidigt i Use Case-projektet – de flyttes til Facade-laget i opgave 04.

---

---

# Opgave 04 – Facade-laget (kontrakt)

## Formål

Formålet er at oprette Facade-laget, som udgør den **eneste indgang** til kernen. Facaden definerer interfaces og DTO'er – ingen implementering. Interfaces opdeles efter CQS-princippet.

## Afgrænsning

- Facade-laget indeholder **kun** interfaces og DTO'er.
- Facade-laget har **ingen** referencer til andre lag.
- Use Case-klasserne fra opgave 03 skal opdateres til at **implementere** Facade-lagets interfaces.

## Funktionelle krav

### Use Case-interfaces (Commands)

1. **IOpretKonsultationsaftaleUseCase**
   - `Task Udfør(OpretKonsultationsaftaleRequest request)`

2. **IÆndrKonsultationstypeUseCase**
   - `Task Udfør(ÆndrKonsultationstypeRequest request)`

3. **IAflysKonsultationsaftaleUseCase**
   - `Task Udfør(AflysKonsultationsaftaleRequest request)`

4. **IMarkerAnkommetUseCase**
   - `Task Udfør(MarkerAnkommetRequest request)`

5. **IAfslutKonsultationsaftaleUseCase**
   - `Task Udfør(AfslutKonsultationsaftaleRequest request)`

### Query-interfaces (Reads)

6. **IKonsultationsaftaleQueries**
   - `Task<KonsultationsaftaleDto?> HentAsync(HentKonsultationsaftaleRequest request)`
   - `Task<IReadOnlyList<KonsultationsaftaleDto>> HentDagensAftalerAsync(HentDagensAftalerRequest request)`
   - `Task<IReadOnlyList<LedigTidDto>> HentLedigeTiderAsync(HentLedigeTiderRequest request)`

7. **IPatientQueries**
   - `Task<PatientDto?> HentAsync(HentPatientRequest request)`
   - `Task<IReadOnlyList<PatientDto>> SøgAsync(SøgPatientRequest request)`

### Request-DTO'er

8. Flyt alle Request-DTO'er fra Use Case-laget til Facade-laget. Alle parametre skal være **primitive typer**.

### Response-DTO'er

9. Opret response-DTO'er som **C# records**:
   - `KonsultationsaftaleDto` (Id, PatientNavn, LægeNavn, Konsultationstype, Fra, Til, Status)
   - `PatientDto` (Id, Navn, Telefon, Email)
   - `LedigTidDto` (LægeId, LægeNavn, Fra, Til)
   - `KonsultationstypeDto` (Id, Navn, Varighed)

## Non-funktionelle krav

- Facade-projektet har **ingen** projektreference til Domain, Use Case eller Infrastructure.
- Use Case-interfaces har **aldrig** en returværdi (`Task`, ikke `Task<T>`).
- Query-interfaces returnerer **altid** DTO'er.
- Domæne-entities eksponeres **aldrig** gennem Facaden.

## Proceskrav

- Opdater Use Case-klasser fra opgave 03, så de implementerer de tilsvarende Facade-interfaces.
- Verificer at alle eksisterende tests stadig passerer efter refaktorisering.

## Hint

- Brug `record` til alle DTO'er – det giver immutabilitet med minimal kode.
- Request-DTO'er til queries bør også modelleres som records (fx `HentDagensAftalerRequest(DateTime Dato, Guid? LægeId)`).

---

---

# Opgave 05 – Infrastructure-laget (EF Core og Queries)

## Formål

Formålet er at implementere Infrastructure-laget med Entity Framework Core. Her implementeres repository-interfaces (fra Use Case-laget) og query-interfaces (fra Facade-laget).

## Afgrænsning

- Der bruges **SQLite** som database (simpelt setup).
- Migrations oprettes med EF Core.
- Seed-data oprettes for konsultationstyper og et par testlæger.

## Funktionelle krav

### DbContext

1. **LægehusetDbContext**
   - DbSets for: `Konsultationsaftale`, `Patient`, `Læge`, `Konsultationstype`.
   - Konfiguration af Value Object `Tidsinterval` som Owned Type eller Complex Type.
   - Konfiguration af relationer og begrænsninger.

### Repository-implementeringer

2. Implementer alle fire repository-interfaces fra opgave 03:
   - `KonsultationsaftaleRepository`
   - `PatientRepository`
   - `LægeRepository`
   - `KonsultationstypeRepository`
   
3. `GemAsync()` skal kalde `SaveChangesAsync()` **uden** at kalde `Update()` – EF Core's change tracking håndterer opdateringer automatisk (se Clean-Architecture.md).

### Query-handler implementeringer

4. **KonsultationsaftaleQueriesImpl** – implementerer `IKonsultationsaftaleQueries` fra Facaden.
   - Brug `AsNoTracking()` for alle queries (performance).
   - Map direkte til DTO'er via `Select()`.
   - `HentDagensAftalerAsync` – returnerer alle aftaler for en given dato (evt. filtreret på læge).
   - `HentLedigeTiderAsync` – beregner ledige tider for en given dato og konsultationstype, med hensyntagen til buffer.

5. **PatientQueriesImpl** – implementerer `IPatientQueries` fra Facaden.

### Seed-data

6. Seed følgende data via `OnModelCreating` eller en separat seed-klasse:
   - De fire konsultationstyper (Almindelig konsultation, Vaccination, Receptfornyelse, Rådgivning).
   - Tre læger med navne og specialer.

## Non-funktionelle krav

- Queries bruger **`AsNoTracking()`** – ingen change tracking for læseoperationer.
- Repositories bruger **ikke** `Update()` – EF Core tracker ændringer automatisk.
- Infrastructure-projektet refererer til **Domain**, **Use Case** og **Facade**.

## Proceskrav

- Opret initial migration og verificer at databasen kan oprettes.
- Skriv integrationstests med **InMemory-database** eller **SQLite in-memory** for at teste queries og repositories.
- Overvej at bruge Moq til at teste query-handlers isoleret.

## Hint

- `Tidsinterval` kan konfigureres som Owned Type med `OwnsOne()` i EF Core.
- For `HentLedigeTiderAsync` kan det være nemmere at beregne ledige tider i C#-kode end i SQL.

---

---

# Opgave 06 – Dependency Injection og sammensætning

## Formål

Formålet er at samle alle lag i en kørende applikation via Dependency Injection. Alle interfaces fra Facade og Use Case skal registreres i DI-containeren, så de kan resolves af Blazor-applikationen.

## Afgrænsning

- Opgaven fokuserer på **composition root** – det sted hvor alle afhængigheder bindes sammen.
- Der oprettes endnu ikke UI – det kommer i opgave 07.

## Funktionelle krav

### Blazor-projekt (Composition Root)

1. Opret et **Blazor Server**-projekt (SSR) som det ydre lag.

2. Registrer services i `Program.cs`:
   - `LægehusetDbContext` med SQLite-connection string.
   - Alle repositories (Scoped).
   - Alle Use Cases (Scoped).
   - Alle Query-implementeringer (Scoped).
   - `KonsultationsoverlapService` (Scoped eller Transient).

3. Blazor-projektet refererer til **Facade** og **Infrastructure**.
   - Blazor-koden bruger **kun** Facade-interfaces og DTO'er.
   - Blazor-koden har **aldrig** direkte adgang til Domain-entities.

### Projektstruktur

4. Den endelige Solution-struktur skal være:

```
Lægehuset/
├── src/
│   ├── Lægehuset.Domain/
│   ├── Lægehuset.UseCases/
│   ├── Lægehuset.Facade/
│   ├── Lægehuset.Infrastructure/
│   └── Lægehuset.Web/              ← Blazor SSR
├── tests/
│   ├── Lægehuset.Domain.Tests/
│   ├── Lægehuset.UseCases.Tests/
│   └── Lægehuset.Infrastructure.Tests/
└── Lægehuset.sln
```

### Afhængighedsregler

5. Verificer at projektreferencerne overholder Dependency Rule:

| Projekt | Refererer til |
|---------|---------------|
| Domain | Intet |
| Facade | Intet |
| UseCases | Domain, Facade |
| Infrastructure | Domain, UseCases, Facade |
| Web | Facade, Infrastructure |
| Domain.Tests | Domain |
| UseCases.Tests | UseCases, Domain, Facade |
| Infrastructure.Tests | Infrastructure, Domain, Facade |

## Non-funktionelle krav

- Blazor-projektet kender **kun** til Facade og Infrastructure (for DI-registrering).
- Ingen direkte reference fra Web til Domain eller UseCases.

## Proceskrav

- Opret løsningen med den viste mappestruktur.
- Byg løsningen og verificer at alle projekter kompilerer uden fejl.
- Kør alle eksisterende tests og verificer at de stadig passerer.
- Start applikationen og verificer at DI-containeren kan resolve alle services.

## Hint

- Overvej at oprette en `ServiceCollectionExtensions`-klasse i Infrastructure-projektet, der registrerer alle Infrastructure- og UseCase-services. Så skal Web-projektet kun kalde én metode.
- Brug `builder.Services.AddDbContext<LægehusetDbContext>(...)` til EF Core.

---

---

# Opgave 07 – Blazor SSR-frontend

## Formål

Formålet er at bygge en Blazor Server (SSR) frontend, der giver sekretæren et brugervenligt interface til at administrere konsultationsaftaler. Alle data hentes og ændres **udelukkende via Facade-lagets interfaces**.

## Afgrænsning

- Fokus er på funktionalitet – ikke avanceret design.
- Blazor-komponenterne injicerer **kun** Facade-interfaces (`IOpretKonsultationsaftaleUseCase`, `IKonsultationsaftaleQueries` osv.).
- Ingen direkte brug af Domain-entities eller repositories i UI-koden.

## Funktionelle krav

### Sider

1. **Dagens overblik** (`/`)
   - Viser alle konsultationsaftaler for dags dato i en tabel/liste.
   - Kolonner: Tidspunkt, Patient, Læge, Type, Status.
   - Mulighed for at filtrere på læge.
   - Knap til at markere en patient som "Ankommet".
   - Knap til at aflyse en aftale.
   - Brug `IKonsultationsaftaleQueries.HentDagensAftalerAsync(...)`.

2. **Opret konsultationsaftale** (`/aftaler/opret`)
   - Formular med felter: Patient (søg/vælg), Læge (vælg), Konsultationstype (vælg), Dato.
   - Når type og dato er valgt, vises de næste 10 ledige tider (via `IKonsultationsaftaleQueries.HentLedigeTiderAsync(...)`).
   - Brugeren vælger en ledig tid og opretter aftalen (via `IOpretKonsultationsaftaleUseCase`).
   - Validering: Starttidspunkt skal ligge i fremtiden.

3. **Patient-administration** (`/patienter`)
   - Søg i eksisterende patienter.
   - Opret ny patient med stamdata.

4. **Aftaldetaljer** (`/aftaler/{id}`)
   - Vis detaljer for en konsultationsaftale.
   - Mulighed for at ændre konsultationstype (via `IÆndrKonsultationstypeUseCase`).
   - Mulighed for at afslutte konsultation med notat (via `IAfslutKonsultationsaftaleUseCase`).

### Fejlhåndtering

5. Vis brugervenlige fejlmeddelelser når:
   - En `DomainException` kastes (fx ugyldig tilstandsovergang).
   - En `NotFoundException` kastes (fx aftale ikke fundet).
   - Overlapskontrol fejler (tidspunktet er allerede optaget).

## Non-funktionelle krav

- Blazor-komponenter injicerer **kun** Facade-interfaces – aldrig repositories, DbContext eller Domain-entities.
- Alle data der vises er **DTO'er** fra Facaden.
- Alle handlinger der ændrer data går via **Use Case-interfaces** fra Facaden.

## Proceskrav

- Implementer siderne trinvist – start med "Dagens overblik".
- Test manuelt at hele flowet virker: opret patient → opret aftale → marker ankommet → afslut.
- Verificer at fejlmeddelelser vises korrekt for ugyldige handlinger.

## Hint

- I Blazor SSR håndteres formularindsendelser med `EditForm` og `OnValidSubmit`.
- Brug `@inject IKonsultationsaftaleQueries Queries` i Razor-filer til at injicere Facade-interfaces.
- Husk at `try/catch` omkring Use Case-kald, så `DomainException` og `NotFoundException` kan fanges og vises for brugeren.
- Overvej at lave en fælles `ErrorDisplay`-komponent til fejlmeddelelser.
