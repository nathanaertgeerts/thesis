## Ticket Systeem

### Wat is een ticket systeem
Normaler wijze contacteer je een bedrijf via mail of telefoon om een antwoord te krijgen op je vraag. Wanneer een bedrijf of organisatie groeit zal het aantal klanten mee groeien en het aantal vragen oplopen. Om de 'support-vragen' te behandelen en een overzicht te bewaren wordt er gevraagd aan de klant om een formulier in te vullen op de website. Dit formulier wordt omgezet in een 'support-ticket', het ticket krijgt een uniek identificatie nummer zodat de klant en de support medewerker eenvoudig kunnen terugvallen op een bepaald ticket. Wanneer een ticket wordt aangemaakt krijgt dit de status 'open' mee, als het ticket in behandeling is wordt deze status omgezet in 'pending'. Vervolgens is er communicatie mogelijk tussen de klant en de support medewerker, wanneer de vraag opgelost is krijgt het ticket de status 'closed'.

<img src="/Images/ticket-flow.png" />


#### Create, read, update en delete
Om van start te gaan met ticket objecten wordt er gebruik gemaakt van basis CRUD-operaties: create, read, update en delete. Aangezien deze acties op verschillende plaatsen worden aanroepen, worden deze samen gebundeld in een ticket service. Om data te kunnen meegeven vanuit de razor pages moeten we invoervelden voorzien binnen een form zodat deze met behulp van een button en databinding de data van de ingevoerde velden kunnen doorgeven aan onze backend. 

In onderstaande code wordt er een variable aangemaakt van het type ticket, met behulp van databinding kan er data worden doorgegeven uit een webformulier van de razor pages. De datum van het ticket wordt bepaald op het moment dat het ticket wordt aangemaakt, alsook de status bij een nieuw ticket zal steeds 'open' zijn. Vervolgens wordt het ticket meegegeven aan de methode AddTicket van de ticket service. 
```
var ticket = new Models.Ticket()
{
     TicketSubject = Input.TicketSubject,
     TicketDetails = Input.TicketDetails,
     TicketRequestor = User.Identity.Name,
     TicketDate = DateTime.Now,
     PriorityType = Input.TicketPriority,
     StatusType = Models.Status.Open,
     ProductType = Input.TicketProduct,
};

m_ticketService.AddTicket(ticket);
```
De ticket service zal op zijn beurt het ticket aanmaken en opslaan in de database.
```
m_context.Ticket.Add(ticket)
m_context.SaveChanges();
```
Bij het updaten van een ticket wordt het gekoppelde ID meegegeven, met het ID kan het juiste ticket object worden opgevraagd en eventueel worden bewerkt. Om een object te zoeken in de database kan er gebruik worden gemaakt van verschillende methodes. 

* FirstOrDefault()
* Where()
* Find()

FirstOrDefault en Where kunnen gebruikt worden op zowel een lijst, array als op een collectie. De methode Find kan enkel gebruikt worden op een lijst en zal opzoek gaan naar een specifieke voorwaarde. FirstOrDefault zal enkel het eerste element uit de lijst of collectie retourneren dat voldoet aan de voorwaarden. De methode Where zal alle items retourneren dat voldoen aan de voorwaarden.

Het object dat in de database gevonden wordt met het juiste ID zal toegekend worden aan de variabele UpdateTicket.
```
var UpdateTicket = m_context.Ticket.Find(id);
```
De gegevens van dit ticket worden geupdate.
```
UpdateTicket.PriorityType = ticketPriority;
UpdateTicket.StatusType = ticketStatus;
```
De aangepaste gegevens moeten worden opgeslagen in de database en het vernieuwde ticketobject wordt terug meegegegeven.
```
m_context.SaveChanges();
return UpdateTicket;
```
Dezelfde flow wordt gevolgd bij het lezen of verwijderen van een object uit de database. 


#### Enums
Om functionele redenen zijn het status- en prioriteitenveld als Enums opgeslagen. Omdat de variabelen maar uit een kleine set van mogelijkheden kan bestaan, zo kunnen er geen fouten insluipen via strings.

```
public enum Status
{
     Open = 1,
     Pending = 2,
     Closed = 3
}
```

Om tickets nog eenvoudiger te kunnen behandelen is er na overleg geopteerd om de bestaande producten van intation eveneens toe te voegen als enum (InControl, InWave, InDocumentation).

#### Categorieën
Een wijziging die later is toegevoegd is het toekennen van categorieën aan tickets, aangezien de categorieën nog niet gedefinieerd zijn willen we deze zelf kunnen toevoegen en instellen. De categorieën worden op dezelfde manier aangemaakt als een ticket en zijn enkel instelbaar door de administrators. Een klant kan bij het aanmaken van zijn ticket dankzij een dropdown de bijpassende categorie selecteren.

#### One-to-many relatie
Een ticket op zichzelf verteld veel over het probleem maar heeft weinig nut als er niet op geantwoord kan worden. Hiervoor moeten de tickets kunnen voorzien worden van antwoorden, dit is een one-to-many relatie. Er wordt hiervoor een nieuw Reply model gecreëerd en het ticket model wordt aangepast door te zeggen dat deze een lijst van objecten bevat van Reply. 

```
public List<Reply> Replies { get; set; }
```

Om een antwoord toe te voegen wordt opnieuw het ticket object opgevraagd aan de hand van zijn ID. Er wordt gebruik gemaakt van de Include() functionaliteit in ASP.NET Core om de data van Reply mee op te vragen uit de database.

```
var getTicketById = m_context.Ticket.Include(x => x.Replies).FirstOrDefault(x => x.TicketId == id);
var dateNow = dateTime;

getTicketById.Replies.Add(new Reply()
{
      Content = ReplyContent,
      ReplyDate = dateNow,
      ReplyRequestor = user,
});

m_context.SaveChanges();
```

Vervolgens worden de antwoorden voorzien van de CRUD-operaties. Om antwoorden te verwijderen kan er ook gebruik worden gemaakt van de ingebouwde Include property in ASP.NET Core waardoor we geen rekening moeten houden met een cascade delete. Een customer kan enkel zijn eigen antwoorden verwijderen, in razor pages kan dit door te verbergen wanneer de identiteit gekoppeld aan het antwoord niet gelijk is aan de identiteit van de customer.

#### HTML-tekst editor
Om een ticket nog verder te verrijken met informatie is het handig om tekst een bepaalde structuur mee te geven en een afbeeldingen of screenshot in te dienen. Summernote wat gebruikt maakt van javascript laat ons toe om tekst als html door te sturen en afbeeldingen als bit64. Dit kan worden opgeslaan als een string en dankzij de razorpages functionaliteit kan je met HTML.raw() de string terug omvormen. Vervolgens wordt de HTML als afbeelding getoond bij het ticket. Tenslotte is dit ook toegepast op de antwoorden. 

Om summernote te implementeren volstaat het om in de layout een stylesheet en javascript referentie te plaatsen
```
<link href="https://cdnjs.cloudflare.com/ajax/libs/summernote/0.8.9/summernote-bs4.css" rel="stylesheet">
<script src="https://cdnjs.cloudflare.com/ajax/libs/summernote/0.8.9/summernote-bs4.js"></script>
```


Daarna plaats je volgend script in je razor page:
```
<script>
        $(document).ready(function () {
            $('#summernote').summernote({
                height: 150
            })
            if ($('#summernote').summernote('maximumImageFileSize', '1048576')) {
                alert('File is too large.')
            };
        });
</script>
```
De hoogte van de textarea wordt gedefinieerd met height, vervolgens limiteren we de uploadbare filesize. Het is belangrijk dat je in het input veld dat voorzien is voor de editor, het id meegeeft zoals je specifieert in je script.
```
id="summernote"
```

### Resultaat
Tickets kunnen op het portaal worden aangemaakt door de gebruiker of support medewerker. Er kunnen antwoorden geplaatst worden op het ticket en deze kunnen rijkelijk gevuld worden met allerlei informatie.

<img src="/Images/portal/newticket.png" height="400"/>
<img src="/Images/portal/ticketview.png" height="400"/>
<img src="/Images/portal/alltickets.png" height="400"/>

