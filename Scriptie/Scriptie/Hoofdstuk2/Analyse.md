# Analyse
### Onderwerp
Het onderwerp van de bachelorproef had een vrij ruime betekenis aan de start van de stage. De opdracht was het voorzien van een customer service portaal, een webpagina waar de klant zichzelf kan helpen. Uit ervaring van projectopdrachten vroeg dit om een uitgebreid onderzoek. De eerste twee weken werden ingepland om een grondige analyse voor te bereiden.

Om het onderzoek te starten ben ik na advies van Daan Roks, de productowner, opzoek gegaan naar reeds bestaande oplossingen of 'tools' op de markt. De bestaande helpdesk tools bieden zeer veel features aan en zijn in grote lijnen met elkaar te vergelijken. De grootste spelers op de markt zijn aanhangsels van andere CRM of teamsupport aanbieders zoals Salesforce of Atlassian. Waardoor een ideale implementatie van deze software vaak steunt op deze aanbieder. Aangezien Intation gebruikmaakt van Office 365 en Visual Studio Team Services werd een eerste vereisten vastgesteld. 

*Het customer service portaal moet aansluiten bij de bestaande infrastructuur.*

### Wat is de huidige infrastructuur?
De website van Intation draait op wordpress, deze is extern opgezet en werd tot voor kort volledig uitbesteed. Wordpress is een Content Management Systeem (CMS) waarmee je zeer eenvoudig een website kan opzetten en onderhouden. Intation heeft intern een voorkeur voor C# en dat is duidelijk waarneembaar aan de huidige producten en projecten binnen Intation. Het meest recente product is in C# geschreven op een basis van ASP.NET Core 2.0 met het Entity Framework Core.

### Bestaande oplossingen en features
Features zijn eigenschappen die meestal een positieve toevoeging hebben aan een bepaald product of software.
Door features van grote spelers in de markt op te lijsten kunnen we nadien filteren op noodzaak van Intation. De grootste spelers op de markt van customer-support profiteren van hun aandeel en zijn zeer prijzig. Daarom heb ik mijn zoekopdracht wederom verfijnt en ben ik specifiek opzoek gegaan naar open-source mogelijkheden. De open-source mogelijkheden voorzien geen database hosting zodat zij zelf geen servicekosten hebben. Het grootste aantal open-source mogelijkheden zijn op een basis van PHP geschreven. 
*Aangezien het customer service portaal moet aansluiten bij de bestaande infrastructuur is er geen voorkeur voor PHP.*

Vervolgens ben ik de mogelijkheden gaan bekijken die aansluiten bij de huidige website. Wordpress biedt een tal van plug-ins om digitale downloads te verkopen, helpdesk-systeem toe te voegen of een knowledgebase op te zetten. De ontwikkelaars van deze plug-ins verdienen hier echter hun brood mee en al snel was het duidelijk dat ze vele features enkel aanbieden als een premium-service. Met de resultaten van mijn onderzoek gebundeld hebben we een eerste meeting gehouden. Tijdens deze meeting heb ik alle mogelijke oplossingen overlopen en eventuele uitblinkers zijn aan bod gekomen met een demo.

Op basis van de analyse en demo's hebben we tijdens de vergadering de richtlijnen van het project nauwkeurig te definiëren.

* Ticket Systeem 
* Email Integratie met ticket systeem 
* Integratie met Visual Studio Team Services 
* Integratie met Exact Online 
* Knowledge base met informatie  
* Downloads en documenten voor geregistreerde gebruikers 
* Notificaties voor updates

### Design
Aan de hand van de lijst met features kunnen we een eenvoudige maar duidelijke opdeling maken dat ons volgend schema oplevert:
<img src="/Images/functional-design.png" />

Er is een duidelijke scheiding tussen features die alleenstaand moeten geimplemnteerd worden en features waarvoor we gebruik konden maken van communicatie met externe services.

### De conclusie van de analyse
Er zijn verschillende oplossingen mogelijk: 
* Bestaande helpdesk-tools 
* servicedesk-tools
* wordpress plug-ins 
* IT-software Management-tools 

Elke mogelijkheid biedt min of meer voor de gevraagde eigenschappen een oplossing. Min of meer wilt zeggen dat sommige gecombineerd moeten worden om een resultaat te bekomen. 

Het was een zeer leerrijk onderzoek, voor zowel mezelf als Intation. We hebben features ontdekt, ondervonden en getest. We hebben bestaande tools op de markt gevonden die al dan niet geschikt zouden zijn, maar vooral een goede achtergrond opgebouwd om zelf een project te starten. Er is gekozen om een asp.net core (2.0) met Entity framework te gebruiken omdat dit sterk aanleunt bij de bestaande stack van Intation

Het resultaat dat Intation wil bereiken is een Customer Service Portaal waarop de klant kan registreren en inloggen. Daarna kan de klant support-tickets creëren en reeds aangemaakte tickets bekijken. Deze tickets worden op hetzelfde portaal door het supportteam behandeld. De ontwikkelaars kunnen met de visual studio team services integratie eenvoudig een bug of work-item aanmaken om hun software development te sturen en te voldoen aan de behoefte van de klant. Het support-ticket systeem moet een geïntegreerde email functionaliteit en een knowledgebase systeem bevatten, zodat klanten zichzelf kunnen helpen en op de hoogte worden gehouden van belangrijke updates. Met als doel dat het supportteam zijn tijd optimaal kan benutten. Het is noodzakelijke voorgaande features te implementeren en te streven naar een tool die kan ingezet worden naarmate het klantenbestand van Intation groeit.

