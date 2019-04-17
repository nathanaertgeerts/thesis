# Email Integratie
Email integratie in een ticketsysteem betekent dat een gebruiker kan antwoorden op zijn ticket via mail. Dit wil zeggen dat wanneer een ticket wordt aangemaakt op het portaal, de gebruiker een email zal ontvangen met als onderwerp een uniek identificatie nummer. Wanneer de gebruiker op de mail antwoord moet de context bij het juiste ticket verschijnen.
Als er een email wordt verstuurd van een onbekend emailadres of het uniek ID is incorrect dat moet het systeem hierop kunnen anticiperen. Tenslotte wilt Intation dat de emails een persoonlijke touch hebben en de vormgeving en stijl aansluiten bij de website en het customer service portaal

Een algemeen emailadres is noodzakelijk zodat emails steeds in dezelfde inbox terechtkomen, hiervoor is een nieuw emailaccount aangemaakt via office 365 (support@intation.eu) binnen het Intation domein.

### Nuget packages
Een NuGet package is een bestand met code gedeeld door een ontwikkelaar. Dankzij de ingebouwde NuGet tools in Visual Studio kan je eenvoudig NuGet packages downloaden en toevoegen aan je project, vervolgens kan je gebruik maken van de functionaliteit. 

Er zijn verschillende NuGet packages beschikbaar die de functionaliteit voor het ontvangen van mails in een ASP.NET Core applicatie ondersteunen. Ik heb gekozen voor Mailkit/Mimekit een NuGet Package met een MIT-licentie en een hoge rating, omdat deze beschikt over een goede documentatie en makkelijk uit te breiden is. 

### Background tasks
Aangezien het binnenhalen en uitlezen van emails een taak is die enkele seconden nodig heeft kan de applicatie vertragen. Om dit te vermijden kan je gebruik maken van een backgroundtask die deze opdracht zal uitvoeren in de achtergrond. Een tweede voordeel aan deze aanpak is dat de mails automatisch worden opgevraagd.

Een background task kan worden geïmplementeerd dankzij de IHostedService interface, dit is een singleton klasse dat geactiveerd wordt wanneer de applicatie opstart en proper wordt afgesloten wanneer de applicatie sluit. Er zijn 3 manieren om een hosted service op te stellen:
#### 1) Timed background tasks

Een getimede background task maakt gebruik van System.Threading.Timer klasse. De timer triggered de Execute methode en wordt gestopt wanneer StopAsync wordt aangeroepen.

#### 2) Consuming a scoped service in a background task

De hosted service creëert een scope om de scoped background task op te lossen en zijn Execute methode aan te roepen.

#### 3) Queued background tasks

De queued task worden gestart en gestopt volgens volgorde in de queu.

In dit project wordt er gebruik gemaakt van de timed background task omdat dit de mogelijkheid geeft telkens de mails op te halen na een bepaald aantal seconden. Onderstaande code verduidelijkt hoe de timed background task geïmplementeerd is.
```
protected abstract Task ExecuteAsync(CancellationToken cancellationToken);

        public virtual Task StartAsync(CancellationToken cancellationToken)
        {
            // Store the task we're executing
            _executingTask = ExecuteAsync(_stoppingCts.Token);

            // If the task is completed then return it,
            // this will bubble cancellation and failure to the caller
            if (_executingTask.IsCompleted)
            {
                return _executingTask;
            }

            // Otherwise it's running
            return Task.CompletedTask;
        }

        public virtual async Task StopAsync(CancellationToken cancellationToken)
        {
            // Stop called without start
            if (_executingTask == null)
            {
                return;
            }

            try
            {
                // Signal cancellation to the executing method
                _stoppingCts.Cancel();
            }
            finally
            {
                // Wait until the task completes or the stop token triggers
                await Task.WhenAny(_executingTask, Task.Delay(Timeout.Infinite,
                                                              cancellationToken));
            }
        }
```
### IMAP
Er wordt gebruik gemaakt van IMAP, Internet Message Access Protocol om de email berichten op te vragen over het internet. De emails worden niet gedownload op de computer, maar rechtstreeks gelezen van de emailserver. Er wordt een nieuw model aangemaakt in de database om deze mails te kunnen uitlezen en de juiste data op te slaan.

Met onderstaande code wordt er verbinding gemaakt met de mailserver en de emails uit de inbox worden uitgelezen. De host, port en Secure Socket Layer (veilige communicatie) moeten gedefinieerd worden. De nieuwe inkomende mails worden in een lijst gestoken zodat er over de lijst kan worden geïtereerd.
```
using (var client = new ImapClient())
{
     client.Connect("outlook.office365.com", 993, SecureSocketOptions.SslOnConnect);
     client.Authenticate("nathan.aertgeerts@intation.eu" + @"\support@intation.eu", "******");
     client.Inbox.Open(FolderAccess.ReadWrite);

     var uids = client.Inbox.Search(SearchQuery.All);
     foreach (var uid in uids)
     {
          ...
     }
}
```
De mail wordt vervolgens aangeduid als gelezen en verwijderd uit de inbox. Dit is essentieel om telkens nieuwe emails uit te lezen en niet de volledige inbox.

```
client.Inbox.AddFlags(uid, MessageFlags.Deleted, true);
client.Inbox.Expunge();
```
Intation heeft beslist dat niet voor elke inkomende mail een ticket moet worden aangemaakt, dit is om spam of andere praktijken te vermijden. Een ticket kan dus alleen worden aangemaakt op het portaal. 

Eerst kijkt het systeem na of de gebruiker gekend is.
```
if (m_userManager.FindByEmailAsync(message.From.Mailboxes.FirstOrDefault().Address) != null)
{
    ...
}
```
Wanneer een mail van een onbekende afzender wordt ontvangen stuurt het systeem een antwoord om eerst een account aan te maken op het portaal en vervolgens een ticket aan te maken. 
```
m_mailmanager.SendEmailRegisterFirst(...);
```
Als een gebruiker reeds gekend is wordt er gevraagd om een ticket aan te maken via het portaal. 
```
m_mailmanager.SendEmailCreateFirst(...);
```
Wanneer een ticket wordt aangemaakt krijgt dit een uniek identificatie nummer, dit nummer gebruiken we eveneens als onderwerp in de mails zodat deze eenvoudig worden teruggevonden door gebruiker en supportmedewerker. Om dit unieke ID uit het onderwerp van de mail te halen kijken we of het subject de voorafgaande code bevat, vervolgens bepalen we de positie van het unieke ID en halen we het ID uit de string.
```
if (message.Subject.Contains("#INT-"))
{
        int positionOfINT = message.Subject.IndexOf("#INT-") + 5;
}
```
Om het ID uit de string te halen maken we gebruik van Regex, regular expretion.
```
string subjectIsId = Regex.Match(Substring, @"\d+").Value;
```
Als de afzender, het subject en unieke ID gekend zijn zal de mail verder worden uitgelezen en wordt de context opgeslagen als antwoord op het ticket in de database. 
```
m_ticketService.AddReply(...);
m_mailmanager.SendEmailTicketUpdate(...);
```

### Email templates
Om persoonlijke emails te sturen maakt de applicatie gebruik van een ontwerp dat zich baseert op de stijl en vormgeving van de Intation website. Emails bestaan uit html code en om consistentie te garanderen werken we met een tabel in html code zodat de mails die we willen versturen steeds een identieke vormgeving hebben. Vervolgens stijlen we de tabel aan de hand van de Intation kleuren. De email templates bevatten placeholders afhankelijk van hun onderwerp, deze placeholders worden ingevuld het moment dat ze verstuurd worden. Zo kunnen we de naam van de gebruiker, url van de webpagina, ticket onderwerp en vele andere zaken meesturen waardoor de gebruiker een duidelijk overzicht in zijn mailbox krijgt.

Een voorbeeld van de templates kan u vinden in de appendices.

### Resultaat

Nieuwe mails worden uitgelezen en automatisch door het systeem behandeld. Wanneer bepaalde zaken in het portaal een mail triggeren wordt de mail verstuurd met bijhorende data. De email templates werken naar behoren.

<img src="/Images/portal/email.png" height="400"/>
