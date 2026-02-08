# Lægehuset DK - Opgave 02: Database Persistence

## Referencer

Case beskrivelse der danner grundlag for opgaven: [Lægehuset-dk.md](..\Lægehuset-dk.md)

**Bygger videre på:** Opgave 01 fra programmeringsfaget, hvor I lavede domæne logik for `Konsultationsaftale`, `Konsultationstype`, `Patient` og `Læge`.

## Formål

Formålet med at løse opgaven er at tilføje **database persistens** til jeres eksisterende Lægehuset DK domæne model ved hjælp af Entity Framework Core.

I Opgave 01 implementerede I forretnings logik og validering, men data blev ikke gemt permanent. Nu skal systemet kunne:
- Gemme konsultationsaftaler permanent i en database
- Genbruge patienter og læger på tværs af konsultationer
- Automatisk seed konsultationstyper ved opstart
- Fungere som et produktionsklart persistenslag

## Afgrænsning
- I denne version fokuserer vi på **grundlæggende EF Core persistens** (CRUD operationer)
- Der skal **ikke** laves brugerinterface - test funktionalitet via `Program.cs` console output
- **Behold** jeres forretnings logik fra Opgave 01 (beregnet sluttidspunkt, validering osv.)

## Funktionelle Krav
- Der skal oprettes en SQL Server database via EF Core migrations
- Database skal indeholde tabeller for: `Patient`, `Laege`, `Konsultationstype`, `Konsultationsaftale`
- Konsultationstyper skal seedes automatisk i databasen (de 4 standardtyper)
- Alle entiteter skal have en `Id` property (primary key)
- `Konsultationsaftale` skal have foreign keys til `Patient`, `Laege` og `Konsultationstype`
- Navigation properties skal konfigureres korrekt (one-to-many relationships)
- **Beregnet `SlutTidspunkt` property** fra Opgave 01 skal bevares og fungere
- Der skal kunne **oprettes** patienter, læger og konsultationsaftaler i databasen
- Der skal kunne **læses** data fra databasen med relaterede entiteter (brug `Include()`)
- Der skal kunne **opdateres** eksisterende konsultationsaftaler (fx markere som afsluttet)
- Der skal kunne **slettes** konsultationsaftaler fra databasen
- Start tidspunkt skal ligge i fremtiden (behold validering fra Opgave 01)
- Konsultationstypen skal kunne ændres på en eksisterende aftale

## Non Funktionelle Krav
- Løsningen skal følge EF Core best practices og konventioner
- Behold OO principper fra Opgave 01 (indkapsling, separation of concerns)
- DbContext skal være korrekt konfigureret med connection string
- Test alle CRUD operationer i `Program.cs`
- Verificer at beregnet `SlutTidspunkt` stadig fungerer efter database roundtrip
- Verificer at seeded konsultationstyper findes i databasen
- Vis console output der demonstrerer funktionalitet
- Migrations skal kunne køres uden fejl
- Database skal oprettes automatisk ved første kørsel
- Foreign key constraints skal fungere korrekt

## Proceskrav

1. **Setup**: Installer EF Core packages og opret `DbContext`
2. **Entities**: Opdater jeres eksisterende klasser med EF Core krav (Id, foreign keys, navigation properties)
3. **Migrations**: Opret initial migration og seed data migration
4. **Testing**: Implementer CRUD operationer i `Program.cs` og verificer output

Leverancer:
- Fungerende console application med EF Core integration
- Migrations mappe med migrations filer
- `Program.cs` med CRUD test scenarios
- Database der oprettes automatisk

## Hint

**Entity Opdateringer**

Jeres eksisterende classes skal udvides med EF Core krav, men **behold jeres forretnings logik**:

**Migration Commands**
```bash
# Opret første migration
dotnet ef migrations add InitialCreate

# Opdater database (opretter database hvis den ikke findes)
dotnet ef database update

# Tilføj seed data migration
dotnet ef migrations add SeedKonsultationstyper

# Anvend seed data
dotnet ef database update
```

**Almindelige Fejl at Undgå**
- **Glem ikke `= null!`** på required navigation properties (ellers compiler warnings)
- **Initialiser collections** med `= new()` for at undgå null reference exceptions
- **Husk TrustServerCertificate=True** i connection string for nyere SQL Server versioner
- **Kør `dotnet ef database update`** efter hver migration
- **Brug Include()** når du skal bruge relaterede entiteter (ellers er de null)

**Bonus Udfordringer (hvis I bliver færdige før tid)**
1. **Fluent API Configuration**: Konfigurer relationships eksplicit med Fluent API i stedet for at stole på konventioner
2. **Seed Test Data**: Seed også nogle test patienter og læger automatisk
3. **Validering**: Implementer jeres validering fra Opgave 01 (start tidspunkt i fremtiden, CPR format osv.)