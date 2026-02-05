# Lægehuset DK - Opgave 03: EF Performance og Avancerede Queries

## Referencer

Case beskrivelse der danner grundlag for opgaven: [Lægehuset-dk.md](..\Lægehuset-dk.md)

**Bygger videre på:** Opgave 02 hvor I implementerede database persistens med EF Core.

## Formål

Formålet med denne opgave er at mestre **Entity Framework performance** og **avancerede query teknikker**. I skal lære at optimere jeres databaseadgang og forstå hvordan EF genererer SQL.

Efter denne opgave vil I kunne:
- Måle og sammenligne performance på forskellige EF approaches
- Vælge den rigtige loading strategy (Eager, Lazy, Explicit)
- Bruge SQL Server Management Studio til at analysere queries
- Optimere databasen med indexes via Fluent API
- Bruge Raw SQL når EF ikke er effektivt nok

## Afgrænsning

- Der skal **ikke** laves brugerinterface - kun test via `Program.cs` med console output
- Fokus er på **database performance** og **query optimering**
- I skal bruge **SQL Server Management Studio** (SSMS) side om side med EF
- Der skal genereres **store mængder testdata** (10.000+ rækker) for at se reelle performance forskelle

## Funktionelle Krav

### Data Generation

**Klasse: `TestDataGenerator`**
- Metode: `GenerateDoctors(int count)` - opretter n læger
- Metode: `GeneratePatients(int count)` - opretter n patienter  
- Metode: `GenerateConsultations(int count)` - opretter n konsultationer med tilfældige læger, patienter og typer
- Brug **batching** (`AddRange`) for at optimere insert hastighed
- Vis progress i console (f.eks. "Genereret 1000/10000...")

### Performance Test Suite

**Klasse: `PerformanceTester`**
Skal implementere følgende testmetoder:

1. **`TestEagerLoading()`**
   - Hent alle læger med deres konsultationer i ÉN query
   - Mål tid med `Stopwatch`
   - Vis antal SQL queries (brug `dbContext.Database.Log`)

2. **`TestLazyLoadingNPlus1()`**
   - Hent alle læger UDEN Include
   - Loop igennem og tilgå `læge.Konsultationsaftaler.Count` for hver læge
   - Mål tid og tæl antal SQL queries der genereres

3. **`TestExplicitLoading()`**
   - Hent alle læger
   - Brug `dbContext.Entry(læge).Collection(l => l.Konsultationsaftaler).Load()` manuelt
   - Mål tid og sammenlign

4. **`TestRawSql()`**
   - Samme resultat som Test 1, men brug `FromSqlRaw` med manuel SQL:
   ```sql
   SELECT d.*, c.* FROM Læger d 
   LEFT JOIN Konsultationsaftaler c ON d.Id = c.LaegeId
   ```
   - Mål tid og sammenlign med Eager Loading

### Avancerede Queries

**I `Program.cs` - implementer følgende:**

1. **FindOverlappendeAftaler** (Tjek for dobbeltbooking)
   - Find alle tilfælde hvor to konsultationer for samme læge overlapper tidsmæssigt
   - Hint: Sammenlign StartTidspunkt og SlutTidspunkt

2. **FindLedigeTiderForLæge**
   - For en given læge og dato, find alle tidsrum på dagen der IKKE er booket
   - Brug konsultationstypens varighed + 5 min buffer
   - Returner liste af ledige tidsintervaller

3. **DagensOversigtMedInclude**
   - Hent alle konsultationer for i dag
   - Inkluder Patient (navn, cpr), Læge (navn), og Type (navn, varighed)
   - Sorter efter starttidspunkt

4. **KompleksRapport**
   - Generer en rapport der viser: Læge navn, antal konsultationer denne måned, samlet tid (varighed + buffer)
   - Brug `GroupBy` og `Sum`

### Database Optimering

**Fluent API Konfigurationer:**
- Tilføj **index** på `Konsultationsaftale.StartTidspunkt` for hurtigere dato-søgning
- Tilføj **index** på `Patient.CprNummer` 
- Konfigurér `DateTime` præcision til 0 (uden millisekunder) for mere præcis sammenligning

## Non Funktionelle Krav

- **Performance:** `TestEagerLoading` må ikke tage mere end 100ms ved 1000 læger og 10.000 konsultationer
- **SQL Analyse:** Alle performance tests skal vise det faktiske SQL der genereres
- **SSMS Integration:** I skal kunne kopiere det genererede SQL fra EF over i SSMS og køre det der
- **Sammenligning:** Dokumentér tidsforskellen mellem Eager Loading, Lazy Loading (N+1), og Raw SQL
- **Batch Size:** Data generation skal bruge batching - må ikke tage mere end 30 sekunder at generere 10.000 konsultationer

## Proceskrav

### Trin 1: Test Data Generation (20 min)
1. Implementér `TestDataGenerator` klassen
2. Generer: 100 læger, 500 patienter, 10.000 konsultationer
3. Verificér i SQL Server Management Studio at data er korrekte

### Trin 2: Performance Baseline (25 min)
1. Implementér `PerformanceTester` klassen
2. Kør Test 1 (Eager Loading) - dette er jeres baseline
3. Mål tid og SQL query count
4. Kig på execution plan i SSMS

### Trin 3: Loading Strategies Sammenligning (30 min)
1. Implementér og kør Test 2 (Lazy Loading N+1)
2. Tæl hvor mange SQL queries der genereres (hint: det er mange!)
3. Implementér og kør Test 3 (Explicit Loading)
4. Dokumentér resultaterne i en tabel i kommentarer

### Trin 4: Raw SQL Sammenligning (20 min)
1. Implementér Test 4 med `FromSqlRaw`
2. Kopier SQL'en over i SSMS og kør den der
3. Sammenlign performance: EF Eager vs EF Raw vs Ren SQL i SSMS

### Trin 5: Database Optimering (20 min)
1. Tilføj Fluent API konfiguration med indexes
2. Generér ny migration: `dotnet ef migrations add AddPerformanceIndexes`
3. Kør performance tests igen - er der forbedring?

### Trin 6: Avancerede Queries (25 min)
1. Implementér de 4 avancerede queries i Program.cs
2. Mål performance på "DagensOversigtMedInclude"
3. Vis resultater i console med pæn formattering

Leverancer:
- `TestDataGenerator.cs` der kan fylde databasen op
- `PerformanceTester.cs` med 4 testmetoder og måleresultater
- `Program.cs` med 4 avancerede queries
- Migration der tilføjer performance indexes
- Dokumentation (i kodekommentarer) af performance resultater

## Hint

**Stopwatch Pattern**
```csharp
using System.Diagnostics;

var sw = new Stopwatch();
sw.Start();

// Din kode her
var result = dbContext.Læger.Include(l => l.Konsultationsaftaler).ToList();

sw.Stop();
Console.WriteLine($"Tid: {sw.ElapsedMilliseconds} ms");
Console.WriteLine($"Rækker: {result.Count}");
```

**SQL Logging i EF**
```csharp
// I DbContext OnConfiguring ELLER i Program.cs
optionsBuilder.LogTo(Console.WriteLine, LogLevel.Information);

// Eller kun for queries:
optionsBuilder.LogTo(
    msg => Console.WriteLine($"[SQL] {msg}"), 
    new[] { DbLoggerCategory.Database.Command.Name });
```

**Batch Insert**
```csharp
// Meget hurtigere end Add() i et loop
var læger = new List<Læge>();
for (int i = 0; i < 100; i++)
{
    læger.Add(new Læge { Navn = $"Læge {i}" });
}
dbContext.Læger.AddRange(læger);
dbContext.SaveChanges();
```

**N+1 Problem Demonstration**
```csharp
// Dette genererer 1 query for læger + N queries for konsultationer
var læger = dbContext.Læger.ToList(); // 1 query
foreach (var l in læger)
{
    // Her genereres en NY SQL query for HVER læge!
    Console.WriteLine(l.Konsultationsaftaler.Count);
}
// Total: 1 + 100 = 101 queries ved 100 læger!
```

**Raw SQL med EF**
```csharp
var læger = dbContext.Læger
    .FromSqlRaw(@"
        SELECT d.*, c.* 
        FROM Læger d
        LEFT JOIN Konsultationsaftaler c ON d.Id = c.LaegeId
        WHERE d.Id = {0}", doctorId)
    .Include(l => l.Konsultationsaftaler) // Kan stadig bruge EF efterfølgende!
    .ToList();
```

**Index i Fluent API**
```csharp
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    // Simple index
    modelBuilder.Entity<Konsultationsaftale>()
        .HasIndex(a => a.StartTidspunkt);
        
    // Composite index (hvis I vil søge på begge)
    modelBuilder.Entity<Konsultationsaftale>()
        .HasIndex(a => new { a.LaegeId, a.StartTidspunkt });
}
```

**Tjek indexes i SSMS**
```sql
-- Se alle indexes på en tabel
EXEC sp_helpindex 'Konsultationsaftaler';

-- Se execution plan (Ctrl+M i SSMS eller Include Actual Execution Plan knappen)
-- Læg mærke til om der bruges Index Seek eller Table Scan!
```

**Overlapping Query**
```csharp
// Find aftaler der overlapper i tid
var overlapping = dbContext.Konsultationsaftaler
    .Where(a1 => dbContext.Konsultationsaftaler
        .Any(a2 => 
            a2.Id != a1.Id && 
            a2.LaegeId == a1.LaegeId &&
            a2.StartTidspunkt < a1.StartTidspunkt.AddMinutes(a1.Type.Varighed + 5) &&
            a2.StartTidspunkt.AddMinutes(a2.Type.Varighed + 5) > a1.StartTidspunkt))
    .ToList();
```

## Bonus Udfordringer

**Level 1 - SQL Profiler:**
Installer SQL Server Profiler (eller brug Extended Events) og fang alle queries mens jeres program kører. Sammenlign med hvad EF logger.

**Level 2 - AsNoTracking:**
Test performance forskel på `AsNoTracking()` vs normal query når I kun skal læse data (ikke opdatere). Når skal man bruge det?

**Level 3 - Compiled Queries:**
Hvis I har en query der køres mange gange (f.eks. `HentDagensAftaler`), implementér den som en Compiled Query og mål performance forbedringen:
```csharp
private static readonly Func<LaegesContext, DateTime, IEnumerable<Konsultationsaftale>> 
    DagensAftalerQuery = 
        EF.CompileQuery((LaegesContext context, DateTime dato) =>
            context.Konsultationsaftaler
                .Where(a => a.StartTidspunkt.Date == dato));
```

**Level 4 - Bulk Operations:**
Hvis I skal slette mange konsultationer på én gang, test forskellen på:
- Slette én ad gangen i et loop
- Bruge `ExecuteSqlRaw` med `DELETE WHERE`
- Bruge NuGet pakken `EFCore.BulkExtensions`