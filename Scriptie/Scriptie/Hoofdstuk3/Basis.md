# Basis
De basis voor het project is een ASP.NET Core 2.0 applicatie waarbij een gebruiker eenvoudig kan inloggen en registreren. Vervolgens moeten er rollen aangemaakt worden en gebruikers aan bepaalde rollen worden toegekend. Deze basisfunctionaliteit moet uitgebreid worden met verificatie van het account met behulp van een bestaand emailadres. Nadien kunnen de gebruikers en rollen permissies worden opgelegd. Om de opstart van de applicatie te vereenvoudigen en de database te vullen met data moet deze automatisch gevuld en gecontroleerd worden bij de opstart van het project.

### Start project
Wanneer je in Visual Studio een project aanmaakt heb je de mogelijkheid om gebruik te maken van een set basis tools, hierdoor is het opzetten van een eenvoudige webapplicatie en het toevoegen van individuele useraccounts een klein werk.

### Registreren
In onderstaande code wordt een variabele aangemaakt van het type ApplicationUser, een klasse dat overerft van IdentityUser. IdentityUser is een standaard uit het Entity Framework dat gebruikt kan worden om gebruikers aan te maken en op te slaan in de database. De gegevens van de nieuwe gebruiker die registreert op het customer service portaal worden met behulp van een een webformulier en databinding doorgegeven.

```                 
var user = new ApplicationUser { UserName = Input.Email, Email = Input.Email, Firstname = ... };
```
Vervolgens wordt de asynchrone methode CreateAsync aangeroepen om een gebruiker aan te maken via de userManager, dit voorziet de mogelijkheid om gebruikers te beheren in ASP.NET Core.

```
var result = await _userManager.CreateAsync(user, Input.Password);
``` 

### Rollen
De ingebouwde rolemanager van ASP.NET zorgt ervoor dat er eenvoudig rollen kunnen aangemaakt worden. Er worden in de applicatie 3 rollen aangemaakt: Admin, Agent en Customer. Vanzelfsprekend kan een admin profiel alle functionaliteit uitoefenen die beschikbaar is op het webportaal. Een Agent is beperkt in het verwijderen en aanpassen van zaken maar kan alles bekijken. Een customer is nog beperkter en kan enkel en alleen zijn eigen tickets bekijken. De restricites worden opgelegd aan een profiel of meerdere profielen met behulp van authorisatie. De toegang tot een controller kan worden geweigerd en de rol bepaald welke html code de gebruiker te zien krijgt.

```
var res = roleManager.CreateAsync(admin).Result;
```
De gebruiker wordt toegevoegd aan een bepaald profiel. Aangezien registratie volgens het portaal enkel voor klanten is, zullen deze allemaal worden toegekend aan het customer-profiel.
```
var AddToRole = _userManager.AddToRoleAsync(user, "Customer").Result;
```

</br>

### Login
Tenslotte kan de gebruiker inloggen op het portaal met behulp van de SignInManager, deze standaard behandeld de autorisatie van de gebruikers. Er wordt gebruik gemaakt van de asynchrone methode PasswordSignInAsync om te controleren of de login gegevens correct zijn.

```                 
var result = await _signInManager.PasswordSignInAsync(Input.Email, Input.Password);
```

### Email verificatie
Het is vanzelfsprekend dat een gebruiker zich moet verifiëren op het portaal met een bestaand emailadres. Hiervoor was het noodzakelijk om een klasse op te stellen die zich enkel en alleen focust op het behandelen van emails. Aangezien de functionaliteit van deze klasse op andere plaatsen zal worden aangesproken, wordt de code gebundeld in een service.

Vooraleer een methode uit de custom service MailManager aangeroepen wordt, genereert het systeem een code van het type string met behulp van de methode GenerateEmailConfirmationTokenAsync. Aan de hand van de code, user-ID en url van het portaal wordt er een EmailConfirmatieLink opgesteld. Deze unieke link wordt verstuurd naar het emailadres van de geregistreerde gebruiker.
```
var code = await _userManager.GenerateEmailConfirmationTokenAsync(user);
var callbackUrl = Url.EmailConfirmationLink(user.Id, code, Request.Scheme);
m_mailmanager.SendEmailConfirmationAsync(Input.Email, callbackUrl, Input.Email);
```
De MailManager maakt gebruik van de SendEmailConfirmationAsync methode. Deze methode leest de opgestelde template uit en vervangt de placeholders met de juiste gegevens. Door gebruik van System.Net.Mail wordt de mail via SMTP vanuit de ASP.NET Core applicatie verstuurd.
```
string body = File.ReadAllText(@"Templates\confirm_mail.html");
body = body.Replace("{url}", HtmlEncoder.Default.Encode(link));
body = body.Replace("{username}", username);

m_client.SendMailAsync(new MailMessage("support@intation.eu", to)
{
    Subject = "InSupport: New User Registration",
    IsBodyHtml = true,
    Body = body
});
```
### Enkele problemen die ondervonden zijn:
Gmail blokkeert inlogpogingen van applicaties of apparaten die oudere securityinstellingen gebruiken. De oplossing hiervoor was deze instellingen aanpassen zodat Gmail hier wel toegang tot verleent. 

De emailconfiguratie gegevens moeten correct zijn, de smtp service is afhankelijk van je email provider. Wanneer je mails wil versturen van een office 365 emailadres zullen deze instellingen anders zijn dan bij Gmail (smtp.gmail.com).

### Database seeden
Om niet telkens opnieuw gebruikers aan te maken, rollen te creëren en gebruikers hieraan toe te kennen heb ik beslist om de database te seeden wanneer er nog geen data in de database zit. Hiervoor is er een DBInitializer klasse aangemaakt die controleert of er al data in de database zit. Indien er geen data is gevonden worden er als eerst rollen aangemaakt, vervolgens worden gebruikers gecreëerd waarvan de emailverificatie reeds op true wordt gezet, tenslotte worden de gebruikers toegevoegd aan een bepaalde rol.
```
context.Database.EnsureCreated();
```
```
if (!context.Roles.Any(r => r.Name == "Admin"))
{...}
```
```
if (!context.Users.Any(u => u.UserName == "nathan.aertgeerts@intation.eu"))
{...}
```

### resultaat
Het resultaat is een webpagina waarop de gebruiker kan inloggen en registreren, de layout van de homepage is aangepast aan de Intation vormgeving en stijl. De basisgegevens van de gebruiker worden opgeslagen in de database en de gebruiker kan zijn emailadres bevestigen aan de hand van de bevestigingsmail die wordt verstuurd naar zijn emailadres.
<img src="/Images/portal/basis.png" height="400"/>

