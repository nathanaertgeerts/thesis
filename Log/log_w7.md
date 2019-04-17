# Logboek Week #7

### Maandag
* Aanmaken van een VSTS service met volgende functies
    * Create Bug
    * Create Feature
    * Get project list
* WorkItem (BugID) koppelen aan TicketId en weergeven in de ticketview
* Navigeren naar VSTS Bug/Feature door op de bug ID in het portaal te klikken
* Model aanmaken dat de JSON uitleest als er op een bugID wordt opgevraagd
* Probleem met het deserializen van een json met vele attributes, documentatie lezen van JSON .NET

### Dinsdag
* Weergeven van statussen en assigned to van de workItems
* Aanpassen van de layout van ticketview, te veel data op 1 pagina
* toevoegen van navtabs voor info en bugs
* het splitten van een string
* projectnaam (variabel) meegeven aan <a href>
* navigeren vanuit webaplicatie naar bug of feature in vsts
   
### Woensdag
* Sprint meeting met product owner
* Overleggen hoe we toegang krijgen tot exact online met de REST API
* Welke data is relevant om te zien in de applicatie?
* XML-file export gebruiken uit exact online
* nieuwe sprint samengesteld
* VSTS token kunnen configureren en aan user toekennen
* deze token opslaan en gebruiken om data te verkrijgen van VSTS

### Donderdag
* Opzoeken hoe we deze token kunnen koppelen aan de useraccounts in het ticketsysteem, 
* .Include(property) lukt niet, opzoeken hoe we dit kunnen includen.
* Opnieuw scope definieren voor exact online en sprint samenstellen
* API key configureerbaar maken per agent
* Mogelijkheid om API te verwijderen en nadien nieuwe toe te voegen
* Bugs opstellen en opnieuw efficient maken

### Vrijdag
* Applicatie beschikbaar maken op computer zodat andere dit via mijn IP adres kunnen bereiken
* Emails nieuwe registratie aanpassen, nieuwe account van ticket andere mailing voorzien
* Nieuwe regestratie webpagina token bevestigen
* rollen aanpasbaar maken voor admins
* opkuisen van admin dashboard
* enkele zaken vereenvoudigen bij het aanmaken/bewerken van een user (userview toegevoegd)
* Applicatie builden op de VSTS server, scripts toevoegen om dit op een een lokale server te zetten
