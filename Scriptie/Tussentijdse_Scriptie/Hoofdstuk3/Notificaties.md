## Notificaties
Notificaties zijn zeer belangrijk voor een gebruiker, zo kan hij of zij onmiddelijk op de hoogte worden gesteld wanneer er een antwoord is op een vraag, updates of nieuwigheden hebben plaats gevonden. Intation wil dat een gebruiker een email ontvangt wanneer er een ticket is aangemaakt of aangepast en wanneer er documenten of artikels zijn toegevoegd. De gebruiker moet zelf zijn notificatieinstellingen kunnen aanpassen. Wanneer nieuwe documenten worden toegevoegd is het vanzelfsprekend dat enkel de gebruikers die toegang hebben tot deze documenten een mail zullen ontvangen.

### Implementatie
De notificaties en meldingen bestaan voor Intation uit emails op maat. Er is geopteerd om elke gebruiker de mogelijkheid te geven om emails te ontvangen over volgende zaken: Ticket Created, Ticket Update, New Article in KB, New Document. De gebruiker kan deze instellingen aanpassen in zijn persoonlijke menu. Wanneer er dus een nieuw ticket wordt aangemaakt of aangepast door of voor de gebruiker krijgt hij hiervan een mail. Wanneer een nieuw document wordt toegevoegd aan het portaal wordt er eerst gekeken welke gebruikers er in een bedrijf zitten en of zij toegang hebben tot deze documenten. Indien dit zo is en de gebruiker zijn meldingen aanstaan zal hij hiervan een mail ontvangen.

Hiervoor is er een nieuwe view voorzien waarbij elke gebruiker in zijn persoonlijke instellingen enkele checkboxen kan aanduiden of hij al dan niet meldingen wilt ontvangen van een bepaalde verandering. De checkboxen zijn gekoppeld aan een boolean die wordt gecontroleerd alvorens een melding wordt verstuurd. Er wordt gecontroleerd op de boolean van de notificatie van de gebruiker voordat de mailmanager de mail zal opstellen. 

```
foreach (var notifications in User.Notifications)
{
     SendMail = notifications.NewDocument;
}
if (SendMail == true)
{
    //mailmanager send mail           
}

```

### Resultaat
De notifcaties zijn te vinden in het persoonlijke menu van de gebruiker en de instellingen kunnen eenvoudig worden aangepast.
