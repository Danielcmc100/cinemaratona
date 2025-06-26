# CineMaratona

>[!NOTE]
| Nome | Matrícula | Email |
|------|-----------|-------|
| Kleber Daniel Mattos Viana | 06007199 | danielcmc100@gmail.com |
| Bernard Abreu Machado | 06006244 | benydepaull@hotmail.com |
| Hugo Norte | 06006259 | hugonorte@gmail.com |
| Gabriel Maciel de Aguiar Silva | 06006665 | gabriielmaciel17@gmail.com |
| Diego Eufrasio Martorana | 06006338 | diego18tere@gmail.com |

- Apresentação: [Cinemaratona Entrega Final](https://youtu.be/url)

## Tópico 1. Arquitetura Limpa (Pastas/Namespaces)
**É possível identificar pastas ou namespaces que separam apresentação, domínio e infraestrutura, evidenciando uma arquitetura limpa?**

Sim, o projeto `cinemaratona-backend` demonstra uma estrutura de pastas e namespaces que sugere uma arquitetura limpa, separando claramente as camadas de apresentação (Controllers), domínio (Models, Services) e infraestrutura (Repositories, Data).

**Exemplo de arquivo e trecho de código:**

**Arquivo:** 
[cinemaratona-backend/Cinemaratona/Controllers/EventController.cs]('https://github.com/Danielcmc100/cinemaratona-backend/blob/main/Cinemaratona/Controllers/EventController.cs')


Este arquivo representa a camada de apresentação (ou interface do usuário/API). Ele interage com a camada de serviço (`EventService`) para realizar operações de negócio, sem se preocupar com os detalhes de persistência de dados.

```csharp
using cinemaratona.Models;
using cinemaratona.Services;
using Mapster;
using Microsoft.AspNetCore.Authorization;
using Microsoft.AspNetCore.Mvc;

namespace cinemaratona.Controllers;

[ApiController]
[Route("api/[controller]")]
public class EventController(EventService eventService) : ControllerBase
{
    private readonly EventService _eventService = eventService;

    [Authorize]
    [HttpGet]
    public ActionResult<List<Event>> Get()
    {   
        var events = _eventService.List().Adapt<List<Event>>();
        return Ok(events);
    }

    // ... outros métodos ...
}
```

**Arquivo:** 
[cinemaratona-backend/Cinemaratona/Services/EventService.cs]('https://github.com/Danielcmc100/cinemaratona-backend/blob/main/Cinemaratona/Services/EventService.cs')

Este arquivo representa a camada de domínio (lógica de negócio). Ele orquestra as operações e utiliza o repositório para acessar os dados, mas não lida diretamente com a persistência.

```csharp
using cinemaratona.Models;
using cinemaratona.Repositories;

namespace cinemaratona.Services;

public class EventService(EventRepository eventRepository)
{
    private readonly EventRepository _eventRepository = eventRepository;

    public List<Event> List()
    {
        return _eventRepository.List();
    }

    // ... outros métodos ...
}
```

**Arquivo:**
[cinemaratona-backend/Cinemaratona/Repositories/EventRepository.cs]('https://github.com/Danielcmc100/cinemaratona-backend/blob/main/Cinemaratona/Repositories/EventRepository.cs')

Este arquivo representa a camada de infraestrutura (persistência de dados). Ele interage diretamente com o contexto do banco de dados (`CinemaratonaContext`) para realizar operações CRUD.

```csharp
using cinemaratona.Data;
using cinemaratona.Models;

namespace cinemaratona.Repositories;

public class EventRepository(CinemaratonaContext context)
{
    private readonly CinemaratonaContext _context = context;

    public List<Event> List()
    {
        return _context.Event.ToList();
    }

    // ... outros métodos ...
}
```

Essa separação em `Controllers`, `Services` e `Repositories` é um forte indicativo de uma arquitetura em camadas, que é um dos pilares da arquitetura limpa, promovendo a separação de preocupações e facilitando a manutenção e testabilidade do código.

## 2. Aplicação de Padrões de Projeto

**Verifique se foi implementado pelo menos um padrão de projeto, como Singleton, Strategy, Repository, ou qualquer outro, mostrando o padrão que foi usado.**

Sim, o padrão de projeto **Repository** foi implementado no projeto.

**Exemplo de arquivo e trecho de código:**

**Arquivo:** 
[cinemaratona-backend/Cinemaratona/Repositories/EventRepository.cs]('https://github.com/Danielcmc100/cinemaratona-backend/blob/main/Cinemaratona/Repositories/EventRepository.cs')

Este arquivo demonstra a implementação do padrão Repository. A classe `EventRepository` encapsula a lógica de acesso a dados para a entidade `Event`, abstraindo os detalhes de persistência do restante da aplicação. Isso permite que a camada de serviço (`EventService`) interaja com os dados sem conhecer os detalhes de como eles são armazenados (seja em um banco de dados, arquivo, etc.).

```csharp
using cinemaratona.Data;
using cinemaratona.Models;

namespace cinemaratona.Repositories;

public class EventRepository(CinemaratonaContext context)
{
    private readonly CinemaratonaContext _context = context;

    public List<Event> List()
    {
        return _context.Event.ToList();
    }

    public Event? Include(Event event_obj)
    {
        _context.Event.Add(event_obj);
        _context.SaveChanges();
        return event_obj;
    }

    public Event? Find(int id)
    {
        return _context.Event.FirstOrDefault(u => u.Id == id);
    }

    public Event? Delete(int id)
    {
        var event_obj = Find(id);
        if (event_obj == null) return null;
        _context.Event.Remove(event_obj);
        _context.SaveChanges();
        return event_obj;
    }

    public Event? Update(Event event_obj)
    {
        var event_objToUpdate = Find(event_obj.Id);
        if (event_objToUpdate == null) return null;
        _context.Entry(event_objToUpdate).CurrentValues.SetValues(event_obj);
        _context.SaveChanges();
        return event_objToUpdate;
    }
}
```

O `EventRepository` fornece métodos para operações CRUD (`List`, `Include`, `Find`, `Delete`, `Update`) que operam na entidade `Event`, isolando a lógica de acesso a dados e tornando-a reutilizável e testável independentemente da lógica de negócio.


## 3. Princípios SOLID em Prática

**Verifique se foi implementado pelo menos um dos princípios do SOLID, como por exemplo, o Single Responsibility ou Open/Closed.**

Sim, o princípio da **Responsabilidade Única (Single Responsibility Principle - SRP)** é evidente na estrutura do projeto.

**Exemplo de arquivo e trecho de código:**

**Arquivo:** [cinemaratona-backend/Cinemaratona/Services/EventService.cs]('https://github.com/Danielcmc100/cinemaratona-backend/blob/main/Cinemaratona/Services/EventService.cs')

Este arquivo demonstra o SRP. A classe `EventService` tem a única responsabilidade de gerenciar a lógica de negócio relacionada a eventos. Ela não se preocupa com a persistência de dados (que é responsabilidade do `EventRepository`) nem com a forma como as requisições HTTP são tratadas (que é responsabilidade do `EventController`).

```csharp
using cinemaratona.Models;
using cinemaratona.Repositories;

namespace cinemaratona.Services;

public class EventService(EventRepository eventRepository)
{
    private readonly EventRepository _eventRepository = eventRepository;

    public List<Event> List()
    {
        return _eventRepository.List();
    }

    public Event? Include(Event event_obj)
    {
        return _eventRepository.Include(event_obj);
    }

    public Event? Find(int id)
    {
        return _eventRepository.Find(id);
    }

    public Event? Delete(int id)
    {
        return _eventRepository.Delete(id);
    }

    public Event? Update(Event event_obj)
    {
        var existingEvent = _eventRepository.Find(event_obj.Id);
        if (existingEvent == null) return null;
        
        return _eventRepository.Update(event_obj);
    }
}
```

Cada classe (`Controller`, `Service`, `Repository`) possui uma única razão para mudar, o que está alinhado com o SRP. Se a lógica de negócio de eventos mudar, apenas `EventService` precisaria ser alterada. Se a forma de persistir eventos mudar, apenas `EventRepository` seria afetada. Essa separação de responsabilidades melhora a manutenibilidade e a testabilidade do código.


## 4. Convenções de Nomenclatura Claras

**Há Variáveis, métodos e propriedades em português claro, sem abreviações, que refletem o conteúdo dessas variáveis ou métodos?**

Sim, no entanto, muito embora a solicitação mencione o português, optamos pela padronização do desenvolvimento usando o idioma inglês. Sempre escrevendo de forma clara e descritiva, como uma boa prática em desenvolvimento de software. O projeto utiliza convenções de nomenclatura claras e descritivas. As variáveis, métodos e propriedades são nomeados de forma a refletir seu propósito e conteúdo, evitando abreviações desnecessárias.

**Exemplo de arquivo e trecho de código:**

**Arquivo:** [cinemaratona-backend/Cinemaratona/Models/Event.cs]('https://github.com/Danielcmc100/cinemaratona-backend/blob/main/Cinemaratona/Models/Event.cs')
Neste arquivo, as propriedades da classe `Event` são nomeadas de forma clara e autoexplicativa, como `Title`, `UsersId`, `CreatedAt`, `Description`, `Location`, `Date` e `MovieId`. Os nomes dos métodos em `EventService` e `EventRepository` também são descritivos (`List`, `Include`, `Find`, `Delete`, `Update`).

```csharp
namespace cinemaratona.Models;
public class Event {
    public int Id { get; set; }
    public required string Title { get; set; }
    public required int[] UsersId { get; set; }
    public DateTime CreatedAt { get; set; }
    public string? Description { get; set; }
    public required string Location { get; set; }
    public DateTime Date { get; set; }
    public int MovieId { get; set; }
}
```



## 5. Documentação Mínima de Código

**Há Comentários objetivos ou inteligentes em pontos estratégicos (como em métodos complexos) que ajudam a entender a lógica sem detalhar linha a linha?**

Não há uma documentação extensa em forma de comentários no código-fonte do repositório. A maioria dos arquivos não possui comentários que expliquem a lógica de métodos complexos ou a finalidade de classes. No entanto, a clareza da nomenclatura e a boa separação de responsabilidades (conforme observado nos itens anteriores) mitigam um pouco a necessidade de comentários excessivos para a compreensão básica do código.

**Exemplo de arquivo e trecho de código:**

Um exemplo onde a ausência de comentários pode ser notada é em `cinemaratona-backend/Cinemaratona/Services/PasswordService.cs` ou em métodos de `EventService` que realizam operações de negócio. Embora a lógica seja relativamente simples, a adição de comentários em métodos que envolvem criptografia ou validações complexas seria benéfica.

```csharp
using System.Security.Cryptography;
using System.Text;

namespace cinemaratona.Services;

public class PasswordService
{
    public byte[] GenerateSalt()
    {
        return RandomNumberGenerator.GetBytes(128 / 8);
    }

    public byte[] HashPassword(string password, byte[] salt)
    {
        // Rfc2898DeriveBytes: KDF (Key Derivation Function) based on PBKDF2
        // 10000 iterations is a common recommendation for password hashing
        using (var pbkdf2 = new Rfc2898DeriveBytes(password, salt, 10000, HashAlgorithmName.SHA256))
        {
            return pbkdf2.GetBytes(256 / 8);
        }
    }

    public bool VerifyPassword(string password, byte[] salt, byte[] hash)
    {
        var newHash = HashPassword(password, salt);
        return newHash.SequenceEqual(hash);
    }
}
```

Neste `PasswordService`, embora os nomes dos métodos sejam claros, um comentário explicando a escolha do algoritmo de hash (`Rfc2898DeriveBytes`) e o número de iterações (`10000`) seria um exemplo de documentação estratégica que agrega valor sem ser redundante.


## 6. Testes Automatizados

**Há a Presença de pelo menos um teste unitário usando xUnit?**

Sim, o projeto possui testes unitários, e o arquivo [TestCinemaratona/Repositories/ReviewRepositoryTests.cs]('https://github.com/Danielcmc100/cinemaratona-backend/blob/main/TestCinemaratona/Repositories/ReviewRepositoryTests.cs') demonstra o uso do framework xUnit para testar a funcionalidade do repositório.

**Exemplo de arquivo e trecho de código:**

**Arquivo:** [cinemaratona-backend/TestCinemaratona/Repositories/ReviewRepositoryTests.cs]('https://github.com/Danielcmc100/cinemaratona-backend/blob/main/TestCinemaratona/Repositories/ReviewRepositoryTests.cs')

Este arquivo contém um teste unitário para o `ReviewRepository`, verificando se uma avaliação é persistida corretamente no banco de dados em memória. O uso de `[Fact]` indica um método de teste xUnit.

```csharp
using Microsoft.EntityFrameworkCore;
using cinemaratona.Data;
using cinemaratona.Models;

public class ReviewRepositoryTests
{
    private DbContextOptions<CinemaratonaContext> BuildInMemoryOptions(string dbName) =>
        new DbContextOptionsBuilder<CinemaratonaContext>()
            .UseInMemoryDatabase(databaseName: dbName)
            .Options;

    [Fact]
    public void AddReview_Should_PersistToDatabase()
    {
        // arrange: cria options e instancia context
        var options = BuildInMemoryOptions("AddReviewDb");
        using (var context = new CinemaratonaContext(options))
        {
            var review = new Review {
                UserId = 1,
                MovieId = 42,
                Opinion = "Ótimo filme!",
                CreatedAt = DateTime.UtcNow,
                Rating = 5,
                Recommended = true,
                Watched = true
            };

            // act: adiciona e salva
            context.Review.Add(review);
            context.SaveChanges();
        }

        // assert: abre novo contexto para simular outra unidade de trabalho
        using (var context = new CinemaratonaContext(options))
        {
            var persisted = context.Review.Single();
            Assert.Equal(42, persisted.MovieId);
            Assert.True(persisted.Recommended);
            Assert.Equal("Ótimo filme!", persisted.Opinion);
        }
    }
}
```

A presença de testes unitários é um bom indicativo de preocupação com a qualidade do código, garantindo que as funcionalidades básicas do sistema operem conforme o esperado e facilitando futuras refatorações e adições de novas funcionalidades.

## 7. Refatorações Evidentes

**Há Trechos de código que passaram podem ter passado por refatoração, tais como: extração de métodos (refatoração “extrair método”) para remover duplicação e melhorar legibilidade?**

Sim, a estrutura geral do projeto, com a clara separação de responsabilidades entre `Controllers`, `Services` e `Repositories`, é fruto de refatoração. A extração da lógica de negócio para a camada de `Services` e a lógica de persistência para a camada de `Repositories` são exemplos clássicos de refatoração 'extrair método' e 'extrair classe' em um nível arquitetural, visando remover duplicação, melhorar a legibilidade e a manutenibilidade do código.

**Exemplo de arquivo e trecho de código:**

Considere o fluxo de uma requisição para listar eventos:

**Arquivo:** 
[cinemaratona-backend/Cinemaratona/Controllers/EventController.cs]('https://github.com/Danielcmc100/cinemaratona-backend/blob/main/Cinemaratona/Controllers/EventController.cs')

```csharp
    [HttpGet]
    public ActionResult<List<Event>> Get()
    {
        var events = _eventService.List().Adapt<List<Event>>();
        return Ok(events);
    }
```

O método `Get()` no `EventController` é conciso e foca apenas na responsabilidade de lidar com a requisição HTTP e retornar a resposta. A lógica de buscar os eventos é delegada ao `_eventService.List()`. Isso evita que o controller contenha lógica de negócio ou de acesso a dados, tornando-o mais limpo e focado.

**Arquivo:** `cinemaratona-backend/Cinemaratona/Services/EventService.cs`

```csharp
    public List<Event> List()
    {
        return _eventRepository.List();
    }
```

O método `List()` no `EventService` por sua vez, delega a responsabilidade de buscar os dados brutos ao `_eventRepository.List()`. Isso garante que o serviço se concentre na orquestração da lógica de negócio, sem se preocupar com os detalhes de como os dados são obtidos do banco de dados.

Essa cadeia de delegação é um resultado direto de refatorações para separar as preocupações, onde cada camada tem uma responsabilidade bem definida, contribuindo para um código mais modular, legível e fácil de testar e manter.

## 8. Tratamento de Erros e Exceções

**Há Uso consistente de blocos "try"/"catch" e mensagens de erro padronizadas, evidenciando a preocupação com confiabilidade e segurança?**


**Arquivo:**
[cinemaratona-backend/Cinemaratona/Repositories/EventRepository.cs]('https://github.com/Danielcmc100/cinemaratona-backend/blob/main/Cinemaratona/Repositories/EventRepository.cs')
```csharp

    public Event? Include(Event event_obj)
    {
        try
        {
            _context.Event.Add(event_obj);
            _context.SaveChanges();
            return event_obj;
        }
        catch (DbUpdateException ex) 
        {
            Console.WriteLine($"Erro ao incluir evento no banco de dados: {ex.Message}");
            return null; 
        }
        catch (Exception ex) 
        {
            Console.WriteLine($"Ocorreu um erro inesperado ao incluir evento: {ex.Message}");
            return null;
        }
    }
```

Neste exemplo, o bloco try/catch é adicionado para capturar exceções que podem ocorrer durante a adição e salvamento do evento no banco de dados. Capturar DbUpdateException permite tratar erros específicos do banco de dados, enquanto um catch genérico para Exception pode lidar com quaisquer outras falhas inesperadas. O log da exceção é crucial para depuração e monitoramento. O retorno de null mantém a consistência com a abordagem atual do projeto de indicar falha através de null.

Para um tratamento de erros mais abrangente, seria ideal implementar um middleware de tratamento de exceções global no ASP.NET Core (Program.cs) para capturar e logar exceções não tratadas e retornar respostas de erro padronizadas para o cliente, evitando que detalhes internos da aplicação sejam expostos.



## 9. Exemplos de Validação de Entrada

**Há Métodos ou filtros que checam parâmetros e garantem que dados inválidos não sejam processados, ou parâmetros adicionados a queries, evitando vulnerabilidades como SQL Injection ou XSS?**

**Arquivo:** [UserService]('https://github.com/Danielcmc100/cinemaratona-backend/blob/main/Cinemaratona/Services/UserService.cs')

No método Include do UserService (cinemaratona-backend/Cinemaratona/Services/UserService.cs), antes de persistir um novo usuário, poderíamos adicionar validações de formato para o email e regras de complexidade para a senha, além do hashing.

```csharp

public User? Include(User user)
    {
        if (!IsValidEmail(user.Email))
        {
            throw new ArgumentException("Formato de email inválido.");
        }

        if (!IsStrongPassword(user.Password))
        {
            throw new ArgumentException("A senha não atende aos requisitos de segurança.");
        }

        user.Password = _passwordService.HashPassword(user.Password);
        return _userRepository.Include(user);
    }

    private bool IsValidEmail(string email)
    {
        try
        {
            var addr = new System.Net.Mail.MailAddress(email);
            return addr.Address == email;
        }
        catch
        {
            return false;
        }
    }

    private bool IsStrongPassword(string password)
    {
        return password.Length >= 8
               && System.Text.RegularExpressions.Regex.IsMatch(password, "[A-Z]")
               && System.Text.RegularExpressions.Regex.IsMatch(password, "[a-z]")
               && System.Text.RegularExpressions.Regex.IsMatch(password, "\\d")
               && System.Text.RegularExpressions.Regex.IsMatch(password, "[^a-zA-Z0-9]");
    }
```
Essas validações adicionais no nível do serviço garantem que os dados estejam em um formato e complexidade aceitáveis antes de serem processados ou persistidos. Para prevenção de XSS, embora a sanitização seja geralmente feita no frontend, é uma boa prática também validar e, se necessário, sanitizar entradas de texto livre (como descrições de eventos ou opiniões de reviews) no backend antes de armazená-las, especialmente se esses dados forem exibidos diretamente em páginas web sem escape adequado. Bibliotecas de sanitização de HTML podem ser utilizadas para isso.

