# En større mini‑applikation hvor du selv skal refaktorere

Her får du en **større mini‑applikation**, der *bevidst* bryder flere SOLID‑principper.
 Din opgave bliver at:

1. Identificere hvilke principper der brydes
2. Refaktorere applikationen, så den følger SOLID
3. (Hvis du vil) få mit løsningsforslag bagefter

Opgaven er lavet så realistisk som muligt, som noget man faktisk kunne møde i en virksomhed.

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

# En fuld SOLID‑refaktorering. Efterfulgt af en lagdelt arkitektur (Domain, Application, Infrastructure)

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

