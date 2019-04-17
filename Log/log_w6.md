# Logboek Week #6

### Maandag
* Laatste updates aan de UI van ticketview
* Opzoeken naar de mogelijkheden om mails te ontvangen in de applicaties
* Keuze gemaakt voor een nuget package (Mailkit/Mimekit) 
* Inkomende mails op inbox (support@intation.eu) opslaan in huidige directory
* een nieuwe model aangemaakt zodat we deze mails kunnen opslaan in de database
* We halen de juiste data uit die we nodig hebben uit de mail
* MailReceiver service aangemaakt waarin dit allemaal gebundeld is
* De gebruiker kan het aantal nieuwe mails bekijken en beslissen of hij/zij een ticket aanmaakt van de mail

### Dinsdag 
* Een mail naar ticketconverter aangemaakt
* De inkomende mails worden uitgelezen en de details worden automatisch overgedragen naar een ticket
* De agent kan dan beslissen of de mail wordt bijgehouden of enkel het ticket
* De admin/agent pagina's aanpassen en vereenvoudigen. De ticketservice beperken tot enkel functies met tickets
* Op de create/convert Ticket pagina controleren of de gebruiker reeds gekend is
* Indien ja, mailtje sturen dat er een ticket is aangemaakt
* Indien nee, mailtje sturen om account aan te maken en dat hij ticket kan bekijken.

### Donderdag
* Afmaken van de mailing-mogelijkheden voor de aangemaakte tickets
* Onderzoek naar VSTS, mogelijkheid om api's te gebruiken. 
* Testen met voorbeelden van VSTS api's in C#
* Get request op projects en de projecten laten zien in een lijst

### Vrijdag
* POST request op Bug of Feature
* extra view om deze aan te maken, radio button selectie voor bugs 
* linken aan tickets
* Api calls testen via postman 
* API calls schrijven in C#
