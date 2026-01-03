# Kan du hjælpe mig med at forstå soild principperne

Selvfølgelig — og fedt at du vil forstå SOLID! Det er fem grundlæggende principper inden for objektorienteret design, der hjælper dig med at skrive kode, som er lettere at vedligeholde, teste og udvide. Jeg forklarer dem på en jordnær måde, uden jargon.

------

# 🧱 SOLID-principperne — forklaret enkelt

## 🟦 **S — Single Responsibility Principle (SRP)**

**En klasse skal kun have én grund til at ændre sig.**  
 Med andre ord: Én klasse = Én opgave.

**Hvorfor?**  
 Hvis en klasse gør for mange ting, bliver den svær at teste og ændre uden at ødelægge noget andet.

**Eksempel:**  
 En `Invoice`-klasse bør *ikke* både beregne moms, gemme data i databasen og sende e-mails. Del det op i flere klasser.

------

## 🟩 **O — Open/Closed Principle (OCP)**

**Software skal være åben for udvidelse, men lukket for ændring.**

**Hvad betyder det?**  
 Du skal kunne tilføje ny funktionalitet *uden* at ændre eksisterende kode.

**Eksempel:**  
 Hvis du har en betalingsløsning, skal du kunne tilføje MobilePay uden at ændre koden for Visa og Mastercard — du tilføjer bare en ny klasse.

------

## 🟨 **L — Liskov Substitution Principle (LSP)**

**En subclass skal kunne bruges i stedet for sin superclass uden problemer.**

**Kort sagt:**  
 Hvis `Bird` har en metode `fly()`, så skal en `Penguin` *ikke* arve fra `Bird`, fordi pingviner ikke kan flyve.

**Hvorfor?**  
 Arv skal give mening — ellers opstår mærkelige bugs.

------

## 🟧 **I — Interface Segregation Principle (ISP)**

**Klienter skal ikke tvinges til at implementere metoder, de ikke bruger.**

**Eksempel:**  
 Et stort interface som `IAnimal` med metoderne `Fly()`, `Swim()`, `Run()` er dårligt.
 En fisk skal ikke tvinges til at implementere `Run()`.

**Løsning:**  
 Lav mindre, mere specifikke interfaces:

- `IFlyable`
- `ISwimmable`
- `IRunnable`

------

## 🟥 **D — Dependency Inversion Principle (DIP)**

**Afhæng af abstraktioner, ikke konkrete implementeringer.**

**Hvad betyder det?**  
 Klasser skal ikke kende detaljer om hinanden — de skal kommunikere gennem interfaces.

**Eksempel:**  
 I stedet for at `Car` opretter en `GasEngine` direkte, skal den modtage et interface `IEngine`.
 Så kan du senere skifte til `ElectricEngine` uden at ændre `Car`.

------

# 🎯 Kort opsummeret i én tabel

| Bogstav | Navn                  | Essens                                          |
| ------- | --------------------- | ----------------------------------------------- |
| S       | Single Responsibility | Én klasse = én opgave                           |
| O       | Open/Closed           | Udvid, men ændr ikke                            |
| L       | Liskov Substitution   | Subklasser skal opføre sig som deres baseklasse |
| I       | Interface Segregation | Små, specifikke interfaces                      |
| D       | Dependency Inversion  | Afhæng af abstraktioner, ikke konkrete klasser  |

------

Hvis du vil, kan jeg også:

- give konkrete kodeeksempler (C#, Java, Python — hvad du vil)
- vise gode og dårlige eksempler
- lave en lille øvelse, så du kan teste din forståelse

Hvad vil du gerne dykke videre ned i?

________

# giv konkrete eksempler i c#.  Og herefter en lille øvelse

Selvfølgelig — lad os gøre SOLID helt konkret med C#‑eksempler og bagefter en lille øvelse, hvor du selv kan prøve kræfter med principperne.

------

# 🟦 S — Single Responsibility Principle (SRP)

### ❌ Dårligt eksempel (én klasse gør for meget)

```csharp
public class Invoice
{
    public decimal CalculateTotal() { /* ... */ }

    public void SaveToDatabase() { /* ... */ }

    public void SendEmail() { /* ... */ }
}
```

### ✅ Godt eksempel (opdelt i ansvar)

```csharp
public class Invoice
{
    public decimal CalculateTotal() { /* ... */ }
}

public class InvoiceRepository
{
    public void Save(Invoice invoice) { /* ... */ }
}

public class EmailService
{
    public void SendInvoice(Invoice invoice) { /* ... */ }
}
```

------

# 🟩 O — Open/Closed Principle (OCP)

### ❌ Dårligt eksempel (ændrer eksisterende kode)

```csharp
public class PaymentService
{
    public void Pay(string method)
    {
        if (method == "Visa") { /* ... */ }
        else if (method == "Mastercard") { /* ... */ }
        else if (method == "MobilePay") { /* ... */ } // nyt → ændrer koden
    }
}
```

### ✅ Godt eksempel (åben for udvidelse)

```csharp
public interface IPaymentMethod
{
    void Pay();
}

public class VisaPayment : IPaymentMethod
{
    public void Pay() { /* ... */ }
}

public class MobilePayPayment : IPaymentMethod
{
    public void Pay() { /* ... */ }
}

public class PaymentService
{
    public void Pay(IPaymentMethod method)
    {
        method.Pay();
    }
}
```

------

# 🟨 L — Liskov Substitution Principle (LSP)

### ❌ Dårligt eksempel (subclass bryder forventninger)

```csharp
public class Bird
{
    public virtual void Fly() { }
}

public class Penguin : Bird
{
    public override void Fly()
    {
        throw new Exception("Penguins can't fly!");
    }
}
```

### ✅ Godt eksempel (korrekt hierarki)

```csharp
public abstract class Bird { }

public interface IFlyingBird
{
    void Fly();
}

public class Sparrow : Bird, IFlyingBird
{
    public void Fly() { /* ... */ }
}

public class Penguin : Bird
{
    // Ingen Fly-metode → ingen brud på forventninger
}
```

------

# 🟧 I — Interface Segregation Principle (ISP)

### ❌ Dårligt eksempel (for stort interface)

```csharp
public interface IAnimal
{
    void Fly();
    void Run();
    void Swim();
}
```

### ✅ Godt eksempel (små, fokuserede interfaces)

```csharp
public interface IFlyable { void Fly(); }
public interface IRunable { void Run(); }
public interface ISwimmable { void Swim(); }

public class Dog : IRunable { public void Run() { } }
public class Duck : IFlyable, ISwimmable { public void Fly() { } public void Swim() { } }
```

------

# 🟥 D — Dependency Inversion Principle (DIP)

### ❌ Dårligt eksempel (afhængig af konkret klasse)

```csharp
public class Car
{
    private GasEngine _engine = new GasEngine();
}
```

### ✅ Godt eksempel (afhængig af abstraktion)

```csharp
public interface IEngine
{
    void Start();
}

public class GasEngine : IEngine
{
    public void Start() { /* ... */ }
}

public class ElectricEngine : IEngine
{
    public void Start() { /* ... */ }
}

public class Car
{
    private readonly IEngine _engine;

    public Car(IEngine engine)
    {
        _engine = engine;
    }
}
```

------

# 🧪 Lille øvelse til dig

Her er en klasse, der *bryder flere SOLID‑principper*.
 Din opgave er at:

1. Identificere hvilke principper der brydes
2. Refaktorere koden, så den følger SOLID

### 🔧 Opgavekode

```csharp
public class ReportGenerator
{
    public string GenerateReport()
    {
        return "Report content";
    }

    public void SaveToFile(string content)
    {
        File.WriteAllText("report.txt", content);
    }

    public void SendEmail(string content)
    {
        // send email...
    }
}
```

------



# Løsningsforslag

Her får du et klart og gennemarbejdet løsningsforslag til øvelsen — og jeg forklarer også, hvilke SOLID‑principper der bliver brudt, og hvordan man retter dem.

------

# 🧩 **1. Hvilke SOLID‑principper bliver brudt?**

### 🔵 **SRP — Single Responsibility Principle**

`ReportGenerator` gør tre ting:

- genererer rapport
- gemmer rapport til fil
- sender e-mail

→ Den har *flere grunde til at ændre sig*, hvilket bryder SRP.

------

### 🟣 **OCP — Open/Closed Principle**

Hvis du vil gemme rapporten i en database i stedet for en fil, eller sende den via SMS i stedet for e-mail, skal du ændre klassen.

→ Den er *ikke* åben for udvidelse, men *lukket* for ændring.

------

### 🔴 **DIP — Dependency Inversion Principle**

Klassen afhænger direkte af:

- `File.WriteAllText`
- en konkret e-mail‑implementering

→ Den burde afhænge af abstraktioner (interfaces), ikke konkrete detaljer.

------

# 🛠️ **2. Løsningsforslag (refaktoreret kode)**

Her er en SOLID‑venlig version, opdelt i ansvar og afhængig af interfaces.

------

## ✨ **Trin 1: Del ansvaret op (SRP)**

```csharp
public class ReportGenerator
{
    public string GenerateReport()
    {
        return "Report content";
    }
}
```

------

## ✨ **Trin 2: Lav abstractions for lagring og afsendelse (DIP + OCP)**

```csharp
public interface IReportSaver
{
    void Save(string content);
}

public interface IReportSender
{
    void Send(string content);
}
```

------

## ✨ **Trin 3: Implementér konkrete strategier (åbent for udvidelse)**

### Gem til fil

```csharp
public class FileReportSaver : IReportSaver
{
    public void Save(string content)
    {
        File.WriteAllText("report.txt", content);
    }
}
```

### Send via e-mail

```csharp
public class EmailReportSender : IReportSender
{
    public void Send(string content)
    {
        // send email...
    }
}
```

------

## ✨ **Trin 4: Orkestrér det hele i en højere‑niveau klasse**

```csharp
public class ReportService
{
    private readonly IReportSaver _saver;
    private readonly IReportSender _sender;

    public ReportService(IReportSaver saver, IReportSender sender)
    {
        _saver = saver;
        _sender = sender;
    }

    public void ProcessReport()
    {
        var generator = new ReportGenerator();
        var content = generator.GenerateReport();

        _saver.Save(content);
        _sender.Send(content);
    }
}
```

------

# 🎉 **Resultat**

Nu:

- har hver klasse ét ansvar (SRP)
- kan du tilføje nye måder at gemme eller sende rapporter på uden at ændre eksisterende kode (OCP)
- afhænger systemet af interfaces i stedet for konkrete klasser (DIP)

------

# 🧪 Vil du have en ny øvelse?

Jeg kan lave en ny opgave, fx:

- en klasse der bryder ISP
- et arvehierarki der bryder LSP
- en større mini‑applikation hvor du selv skal refaktorere

Hvad vil du helst arbejde videre med?

________

# en større mini‑applikation hvor du selv skal refaktorere

Perfekt — her får du en **større mini‑applikation**, der *bevidst* bryder flere SOLID‑principper.
 Din opgave bliver at:

1. Identificere hvilke principper der brydes
2. Refaktorere applikationen, så den følger SOLID
3. (Hvis du vil) få mit løsningsforslag bagefter

Jeg har lavet den så realistisk som muligt, som noget man faktisk kunne møde i en virksomhed.

------

# 🏗️ **Mini‑applikation: “Order Processing System” (dårlig version)**

Forestil dig et system, der håndterer ordrer i en webshop.

Her er den *bevidst dårlige* implementation:

```csharp
public class Order
{
    public int Id { get; set; }
    public decimal Amount { get; set; }
    public string CustomerEmail { get; set; }
}

public class OrderProcessor
{
    public void Process(Order order)
    {
        // 1. Validate order
        if (order.Amount <= 0)
            throw new Exception("Invalid amount");

        // 2. Save to database
        using (var connection = new SqlConnection("connectionstring"))
        {
            connection.Open();
            var cmd = connection.CreateCommand();
            cmd.CommandText = $"INSERT INTO Orders VALUES ({order.Id}, {order.Amount}, '{order.CustomerEmail}')";
            cmd.ExecuteNonQuery();
        }

        // 3. Charge payment
        if (order.Amount > 1000)
        {
            // charge with PayPal
        }
        else
        {
            // charge with CreditCard
        }

        // 4. Send confirmation email
        var smtp = new SmtpClient("smtp.server.com");
        smtp.Send("shop@company.com", order.CustomerEmail, "Order Confirmation", "Thanks for your order!");
    }
}
```

------

# 🚨 **Hvad er galt her?**

Denne klasse bryder næsten *alle* SOLID‑principper:

- **SRP**: Den validerer, gemmer i database, håndterer betaling og sender e-mail
- **OCP**: Hvis du vil tilføje MobilePay, skal du ændre koden
- **LSP**: Betalingslogikken vil bryde sammen, hvis du laver nye betalingsmetoder
- **ISP**: Der er ingen interfaces — alt er hårdt koblet
- **DIP**: Klassen afhænger af konkrete klasser som `SqlConnection`, `SmtpClient`, PayPal‑logik osv.

------

# 🎯 **Din opgave**

Refaktorér systemet, så det følger SOLID.

Du må gerne:

- introducere interfaces
- lave nye klasser
- bruge dependency injection
- opdele ansvar
- lave strategier for betaling
- lave repositories
- lave services
- lave validators

**Du bestemmer selv arkitekturen — bare den følger SOLID.**

------

# 📦 **Bonus: Ekstra krav (valgfrit)**

Hvis du vil udfordre dig selv:

- Tilføj logging
- Tilføj flere betalingsmetoder
- Tilføj mulighed for at sende SMS i stedet for e-mail
- Tilføj unit tests (du behøver ikke skrive dem, bare design til det)

------

# 👉 Vil du have mit løsningsforslag, når du har prøvet selv?

Jeg kan lave:

- en fuld SOLID‑refaktorering
- en lagdelt arkitektur (Domain, Application, Infrastructure)
- eller en mere simpel version, hvis du foretrækker det

Hvordan vil du gribe opgaven an?

__________

# en fuld SOLID‑refaktorering. Efterfulgt af en lagdelt arkitektur (Domain, Application, Infrastructure)

Lad os tage den i to trin:

1. Først en fuld SOLID‑refaktorering af din “OrderProcessing”
2. Derefter lægger vi den over i en lagdelt arkitektur: Domain / Application / Infrastructure  [developersvoice.com](https://developersvoice.com/blog/layered-architecture-explained/)  [DEV Community](https://dev.to/dotnetfullstackdev/layered-n-tier-architecture-in-net-core-51ic)  [en.ittrip.xyz](https://en.ittrip.xyz/c-sharp/domain-layering-csharp)

------

## 1. Fuld SOLID‑refaktorering (stadig som “én app”)

### 1.1 Domainmodel

```csharp
public class Order
{
    public int Id { get; }
    public decimal Amount { get; }
    public string CustomerEmail { get; }

    public Order(int id, decimal amount, string customerEmail)
    {
        if (amount <= 0)
            throw new ArgumentException("Amount must be greater than zero.");

        if (string.IsNullOrWhiteSpace(customerEmail))
            throw new ArgumentException("Customer email is required.");

        Id = id;
        Amount = amount;
        CustomerEmail = customerEmail;
    }
}
```

- **SRP:** `Order` har ansvar for at være gyldig som domæneobjekt (invariants).

------

### 1.2 Abstraktioner (interfaces)

```csharp
public interface IOrderRepository
{
    void Save(Order order);
}

public interface IPaymentMethod
{
    void Charge(Order order);
}

public interface INotificationService
{
    void SendOrderConfirmation(Order order);
}

public interface IOrderValidator
{
    void Validate(Order order);
}
```

- **DIP:** Høj-niveau logik afhænger af interfaces, ikke konkrete klasser.
- **OCP:** Nye strategier (betaler, repositories, notifikationer) kan tilføjes uden at ændre eksisterende kode.

------

### 1.3 Konkrete implementeringer

#### Repository (database)

```csharp
public class SqlOrderRepository : IOrderRepository
{
    private readonly string _connectionString;

    public SqlOrderRepository(string connectionString)
    {
        _connectionString = connectionString;
    }

    public void Save(Order order)
    {
        using var connection = new SqlConnection(_connectionString);
        connection.Open();

        using var cmd = connection.CreateCommand();
        cmd.CommandText =
            "INSERT INTO Orders (Id, Amount, CustomerEmail) VALUES (@id, @amount, @email)";
        cmd.Parameters.AddWithValue("@id", order.Id);
        cmd.Parameters.AddWithValue("@amount", order.Amount);
        cmd.Parameters.AddWithValue("@email", order.CustomerEmail);

        cmd.ExecuteNonQuery();
    }
}
```

------

#### Betalingsstrategier

```csharp
public class PayPalPayment : IPaymentMethod
{
    public void Charge(Order order)
    {
        // PayPal API kald her
    }
}

public class CreditCardPayment : IPaymentMethod
{
    public void Charge(Order order)
    {
        // Kreditkort API kald her
    }
}
```

Hvis du vil vælge betalingsmetode ud fra beløb, kan du lave en `IPaymentStrategy`:

```csharp
public interface IPaymentStrategy
{
    IPaymentMethod GetPaymentMethodFor(Order order);
}

public class AmountBasedPaymentStrategy : IPaymentStrategy
{
    private readonly IPaymentMethod _paypal;
    private readonly IPaymentMethod _creditCard;

    public AmountBasedPaymentStrategy(IPaymentMethod paypal, IPaymentMethod creditCard)
    {
        _paypal = paypal;
        _creditCard = creditCard;
    }

    public IPaymentMethod GetPaymentMethodFor(Order order)
    {
        return order.Amount > 1000 ? _paypal : _creditCard;
    }
}
```

------

#### Notifikation (mail)

```csharp
public class SmtpNotificationService : INotificationService
{
    private readonly string _smtpServer;
    private readonly string _fromAddress;

    public SmtpNotificationService(string smtpServer, string fromAddress)
    {
        _smtpServer = smtpServer;
        _fromAddress = fromAddress;
    }

    public void SendOrderConfirmation(Order order)
    {
        using var smtp = new SmtpClient(_smtpServer);
        smtp.Send(
            _fromAddress,
            order.CustomerEmail,
            "Order Confirmation",
            "Thanks for your order!"
        );
    }
}
```

------

#### Validator (valgfri, hvis du ikke lægger alt i konstruktør)

```csharp
public class OrderValidator : IOrderValidator
{
    public void Validate(Order order)
    {
        if (order.Amount <= 0)
            throw new ArgumentException("Amount must be greater than zero.");

        if (string.IsNullOrWhiteSpace(order.CustomerEmail))
            throw new ArgumentException("Customer email is required.");
    }
}
```

------

### 1.4 Høj-niveau service (SOLID‑venlig “OrderProcessor”)

```csharp
public class OrderProcessor
{
    private readonly IOrderRepository _orderRepository;
    private readonly IPaymentStrategy _paymentStrategy;
    private readonly INotificationService _notificationService;
    private readonly IOrderValidator _validator;

    public OrderProcessor(
        IOrderRepository orderRepository,
        IPaymentStrategy paymentStrategy,
        INotificationService notificationService,
        IOrderValidator validator)
    {
        _orderRepository = orderRepository;
        _paymentStrategy = paymentStrategy;
        _notificationService = notificationService;
        _validator = validator;
    }

    public void Process(Order order)
    {
        _validator.Validate(order);

        _orderRepository.Save(order);

        var paymentMethod = _paymentStrategy.GetPaymentMethodFor(order);
        paymentMethod.Charge(order);

        _notificationService.SendOrderConfirmation(order);
    }
}
```

- **SRP:** `OrderProcessor` orkestrerer use casen “process order” – intet andet.
- **DIP:** Alt er afhængigt af interfaces.
- **OCP:** Nye betalingsmetoder, nye notifikationstyper, andre persistensstrategier kan tilføjes uden at ændre `OrderProcessor`.

------

## 2. Lagdelt arkitektur: Domain / Application / Infrastructure

Nu pakker vi det ind i en klassisk lagdelt arkitektur, som ofte bruges i .NET: Domain, Application, Infrastructure  [Microsoft Learn](https://learn.microsoft.com/en-us/dotnet/architecture/microservices/microservice-ddd-cqrs-patterns/ddd-oriented-microservice)  [developersvoice.com](https://developersvoice.com/blog/layered-architecture-explained/)  [DEV Community](https://dev.to/dotnetfullstackdev/layered-n-tier-architecture-in-net-core-51ic)  [en.ittrip.xyz](https://en.ittrip.xyz/c-sharp/domain-layering-csharp).

### 2.1 Overblik

- **Domain layer**
  - Kernen: entiteter, value objects, domænelogik, domæneinterfaces
- **Application layer**
  - Use cases / services (`OrderProcessor`), DTOs, orchestrering
- **Infrastructure layer**
  - Database, SMTP, eksterne APIs – konkrete implementationer af domæneinterfaces

------

### 2.2 Mappestruktur / namespaces

Fx:

```text
MyShop.Domain
    Order.cs
    IOrderRepository.cs
    IPaymentMethod.cs
    IPaymentStrategy.cs
    INotificationService.cs
    IOrderValidator.cs

MyShop.Application
    Orders
        ProcessOrderCommand.cs
        ProcessOrderResult.cs
        OrderProcessor.cs

MyShop.Infrastructure
    Persistence
        SqlOrderRepository.cs
    Payments
        PayPalPayment.cs
        CreditCardPayment.cs
        AmountBasedPaymentStrategy.cs
    Notifications
        SmtpNotificationService.cs
```

------

### 2.3 Domain layer (ren, uden “teknologi”)

```csharp
namespace MyShop.Domain;

public class Order
{
    public int Id { get; }
    public decimal Amount { get; }
    public string CustomerEmail { get; }

    public Order(int id, decimal amount, string customerEmail)
    {
        if (amount <= 0)
            throw new ArgumentException("Amount must be greater than zero.");

        if (string.IsNullOrWhiteSpace(customerEmail))
            throw new ArgumentException("Customer email is required.");

        Id = id;
        Amount = amount;
        CustomerEmail = customerEmail;
    }
}

public interface IOrderRepository
{
    void Save(Order order);
}

public interface IPaymentMethod
{
    void Charge(Order order);
}

public interface IPaymentStrategy
{
    IPaymentMethod GetPaymentMethodFor(Order order);
}

public interface INotificationService
{
    void SendOrderConfirmation(Order order);
}

public interface IOrderValidator
{
    void Validate(Order order);
}
```

- Domain kender **ikke** til `SqlConnection`, `SmtpClient`, frameworks osv.  [Microsoft Learn](https://learn.microsoft.com/en-us/dotnet/architecture/microservices/microservice-ddd-cqrs-patterns/ddd-oriented-microservice)  [en.ittrip.xyz](https://en.ittrip.xyz/c-sharp/domain-layering-csharp)

------

### 2.4 Application layer (use case + DTO’er)

```csharp
namespace MyShop.Application.Orders;

using MyShop.Domain;

public class ProcessOrderCommand
{
    public int Id { get; set; }
    public decimal Amount { get; set; }
    public string CustomerEmail { get; set; }
}

public class ProcessOrderResult
{
    public bool Success { get; set; }
    public string? Error { get; set; }
}

public class OrderProcessor
{
    private readonly IOrderRepository _orderRepository;
    private readonly IPaymentStrategy _paymentStrategy;
    private readonly INotificationService _notificationService;
    private readonly IOrderValidator _validator;

    public OrderProcessor(
        IOrderRepository orderRepository,
        IPaymentStrategy paymentStrategy,
        INotificationService notificationService,
        IOrderValidator validator)
    {
        _orderRepository = orderRepository;
        _paymentStrategy = paymentStrategy;
        _notificationService = notificationService;
        _validator = validator;
    }

    public ProcessOrderResult Handle(ProcessOrderCommand command)
    {
        try
        {
            var order = new Order(command.Id, command.Amount, command.CustomerEmail);

            _validator.Validate(order);
            _orderRepository.Save(order);

            var paymentMethod = _paymentStrategy.GetPaymentMethodFor(order);
            paymentMethod.Charge(order);

            _notificationService.SendOrderConfirmation(order);

            return new ProcessOrderResult { Success = true };
        }
        catch (Exception ex)
        {
            // I praksis: log i Application eller Infrastructure
            return new ProcessOrderResult
            {
                Success = false,
                Error = ex.Message
            };
        }
    }
}
```

- Application layer koordinerer flows, men indeholder ingen tekniske implementeringsdetaljer  [developersvoice.com](https://developersvoice.com/blog/layered-architecture-explained/)  [DEV Community](https://dev.to/dotnetfullstackdev/layered-n-tier-architecture-in-net-core-51ic)  [en.ittrip.xyz](https://en.ittrip.xyz/c-sharp/domain-layering-csharp).

------

### 2.5 Infrastructure layer (konkrete “adapters”)

```csharp
namespace MyShop.Infrastructure.Persistence;

using System.Data.SqlClient;
using MyShop.Domain;

public class SqlOrderRepository : IOrderRepository
{
    private readonly string _connectionString;

    public SqlOrderRepository(string connectionString)
    {
        _connectionString = connectionString;
    }

    public void Save(Order order)
    {
        using var connection = new SqlConnection(_connectionString);
        connection.Open();

        using var cmd = connection.CreateCommand();
        cmd.CommandText =
            "INSERT INTO Orders (Id, Amount, CustomerEmail) VALUES (@id, @amount, @email)";
        cmd.Parameters.AddWithValue("@id", order.Id);
        cmd.Parameters.AddWithValue("@amount", order.Amount);
        cmd.Parameters.AddWithValue("@email", order.CustomerEmail);

        cmd.ExecuteNonQuery();
    }
}
namespace MyShop.Infrastructure.Payments;

using MyShop.Domain;

public class PayPalPayment : IPaymentMethod
{
    public void Charge(Order order)
    {
        // PayPal integration
    }
}

public class CreditCardPayment : IPaymentMethod
{
    public void Charge(Order order)
    {
        // Credit card integration
    }
}

public class AmountBasedPaymentStrategy : IPaymentStrategy
{
    private readonly IPaymentMethod _paypal;
    private readonly IPaymentMethod _creditCard;

    public AmountBasedPaymentStrategy(IPaymentMethod paypal, IPaymentMethod creditCard)
    {
        _paypal = paypal;
        _creditCard = creditCard;
    }

    public IPaymentMethod GetPaymentMethodFor(Order order)
    {
        return order.Amount > 1000 ? _paypal : _creditCard;
    }
}
namespace MyShop.Infrastructure.Notifications;

using System.Net.Mail;
using MyShop.Domain;

public class SmtpNotificationService : INotificationService
{
    private readonly string _smtpServer;
    private readonly string _fromAddress;

    public SmtpNotificationService(string smtpServer, string fromAddress)
    {
        _smtpServer = smtpServer;
        _fromAddress = fromAddress;
    }

    public void SendOrderConfirmation(Order order)
    {
        using var smtp = new SmtpClient(_smtpServer);
        smtp.Send(
            _fromAddress,
            order.CustomerEmail,
            "Order Confirmation",
            "Thanks for your order!");
    }
}
```

------

### 2.6 Komposition (fx i en ASP.NET Core `Program.cs`)

```csharp
builder.Services.AddScoped<IOrderRepository>(
    _ => new SqlOrderRepository(builder.Configuration.GetConnectionString("DefaultConnection")));

builder.Services.AddScoped<IPaymentMethod, PayPalPayment>();
builder.Services.AddScoped<IPaymentMethod, CreditCardPayment>(); // hvis du navngiver, kan du bruge flere registreringer

builder.Services.AddScoped<IPaymentStrategy, AmountBasedPaymentStrategy>();
builder.Services.AddScoped<INotificationService>(
    _ => new SmtpNotificationService("smtp.server.com", "shop@company.com"));
builder.Services.AddScoped<IOrderValidator, OrderValidator>();

builder.Services.AddScoped<OrderProcessor>();
```

------

Hvis du vil, kan vi næste skridt:

- gøre det mere DDD‑agtigt (aggregates, value objects, domain events)
- tilføje async/await og EF Core
- eller tage en konkret use case og skrive unit tests til Application‑laget.

Hvad har du mest lyst til at bygge videre på herfra?