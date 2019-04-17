# Conclusie
## Realisaties 
In hoofdstuk 3 worden de resultaten gedetailleerd beschreven. Er is een customer service portaal in ASP.NET Core 2.0 met Entity Framework gerealiseerd. 

De applicatie bevat volgende zaken: 
* Ticketsysteem
* Email integratie
* Integratie met Visual Studio Team Services met behulp van REST-API
* Integratie met Exact Online met behulp van REST-API
* Knowledgebase
* Document management systeem

#### Ticketsysteem
Een gebruiker kan zichzelf registreren op het portaal en kan vervolgens een ticket aanmaken. De agents of administrators kunnen antwoorden op het ticket. Het antwoord komt overzichtelijk bij het ticket te staan en de klant krijgt een email in zijn mailbox. Vervolgens kan de klant rechtstreeks op de mail of via het portaal te conversatie verderzetten. De tickets kunnen met behulp van een ordelijke tabel éénvoudig gerangschikt of opgezocht worden. De applicatie werkt naar behoren en de grote hoeveelheid aan informatie wordt duidelijk weergegeven. 

#### Email integratie
Emails worden automatisch verstuurd waardoor de gebruikers op de hoogte worden gehouden van updates. Ze kunnen dit op het portaal zelf instellen of ze al dan niet een email willen ontvangen over bepaalde zaken. De emails zijn afkomstig van het nieuwe emailadres "support@intation.eu". Mails die verstuurd worden naar dit adres worden automatisch in het systeem verwerkt, zodat tickets geupdate worden, gebruikers de juiste meldingen ontvangen en emails dat geen betrekking hebben tot het portaal worden gefilterd.

#### Integratie met Visual Studio Team Services met behulp van REST-API
Als agents hun eigen API-key aan maken in Visual Studio Team Services kunnen ze deze toevoegen aan het customer service portaal en zo zichzelf in directe verbinding brengen met de statussen van bugs en features binnen hun eigen projecten. Ze hebben ook de mogelijkheid om rechtstreeks uit het portaal bugs of features aan te maken in het juiste project. Hierdoor zal de efficientie positief worden beïnvloed. 

#### Integratie met Exact Online met behulp van REST-API
De gegevens en data van klanten kan met behulp van API automatisch geupdate worden in het customer service portaal. De data van het bedrijf en de bijhorende contactpersonen worden gekoppeld en zijn zichtbaar in het portaal. Vervolgens kunnen we gebruikers van het customer service portaal synchroniseren met een bedrijf. Wanneer een bedrijf een contractlevel heeft meegekregen zal dit bepalend zijn voor de informatie dat de gekoppelde gebruikers van dit bedrijf kunnen zien. 

#### Knowledgebase
Verschillende mappen of categorieën kunnen worden aangemaakt door de support medewerkers, zodat de knowledgebase artikels in de juiste folders terecht komen. Wanneer de knowledgebase groeit aan informatie kan de zoekfunctie worden gebruikt. De artikels kunnen gedetailleerd zijn doordat ze afbeeldingen, tabellen of code voorbeelden kunnen bevatten. Wanneer een nieuw artikel wordt toegevoegd zullen de gebruikers een mail ontvangen.

#### Document management systeem
Het Document management systeem zorgt ervoor dat support medewerkers documenten ter beschikking kunnen stellen en vervolgens restricties opleggen. Gebruikers kunnen documenten individueel downloaden of bepaalde folders als een gecomprimeerd bestand. De documenten zijn rechtstreeks beschikbaar uit Sharepoint vanop de server. Wanneer een nieuw document beschikbaar is worden alle gebruikers op wie dit van toepassing is op de hoogte gebracht.

## Aanbevelingen
ASP.NET Core heeft een stijle leercurve, wanneer de kennis de bovenhand grijpt word je al snel geprikkeld om te experimenteren met de mogelijkheden. Het framework en de community bieden een goede ondersteuning en zijn ongetwijfeld een goede basis om eender welke web applicatie te bouwen.

Entity Framework Core in combinatie met ASP.NET is ongetwijfeld de beste aanpak om eenvoudig een database te beheren. De simpliciteit en mogelijkheden van Entity Framework bieden een beginnende en ervaren ontwikkelaar de mogelijkheid om veel sneller en productiever te programmeren. 

Het gebruik van razor pages geeft de mogelijkheid om C# code te schrijven die achter de HTML pagina zit, waardoor de databinding zeer logisch is. Omgaan met user gegevens en input dat moet worden opgeslagen is in combinatie met Entity Framework een taak dat snel kan worden voltooid.

## Toekomstig werk of uitbreidingen 
De algemene stijl en user experience van het customer service portaal zou ongetwijfeld nog beter presteren aan de hand van verbeteringen door feedback van user testing. De interne performantie kan worden verbeterd door unit tests te implementeren.
De twee bovenstaande factoren zullen bijdragen aan de code quality van het gehele project. 

Om het ticket systeem uit te breiden kunnen we rekening gaan houden met de tijdsduur dat een ticket in behandeling is. Vervolgens hierop anticiperen en agents een melding sturen zodat er niets over het hoofd wordt gezien. Nadien kunnen we reports voorzien zodat de administrator een overzicht heeft van de efficientie.Een uitbreiding van Exact Online met een licentie module zal extra data opleveren, hierop moet dan opnieuw geanticipeerd worden zodat de support medewerkers continu de beste informatie hebben om de klant zo goed mogelijk te helpen. Om het document management systeem uit te breiden kunnen we gebruikers zelf een download map laten samenstellen. Support medewerkers een eenvoudigere oplossing bieden dan een statisch in te vullen bestandslocatie. De knowledgebase kunnen we optimaliseren door de zoekfunctie te optimaliseren. 

Om de mogelijkheid te bieden aan gebruikers om elkaar te helpen kan een forum een meerwaarde bieden aan het customer service portaal. Hier kunnen dan zowel de gebruikers als de support medewerkers oplossingen aanbieden. Tot slot kan een eigen vorm van een uitgebreide licentie module een meerwaarde bieden om in de toekomst gepersonaliseerde support mogelijkheden aan te bieden.

## Slot conclusie
We kunnen besluiten dat de opgelegde doelstellingen behaald zijn. De belangrijkste zaken voor Intation hebben hun plek gekregen in het project en zijn dankzij de consistente en positieve samenwerking allemaal geïmplementeerd. 

Het was voor mij een nieuwe ervaring om met razor pages te werken. Alsook was de kennis van .NET in het begin nog niet te vergelijken met het niveau dat er bereikt is op het einde van de stage. Het aantal features was niet min, waardoor het overzicht soms te wensen overliet. Uiteindelijk hebben de features allemaal hun plaats gekregen en zijn deze met behulp van sprints en sprintmeetings volledig gedefinieerd. Doordat de samenwerking en communicatie zeer vlot is verlopen kon een bepaalde visie of aanpak snel worden bijgestuurd en dit had tot gevolg dat er weinig tot geen tijd verloren is gegaan. 

Het geeft een enorme boost om terug te kijken op wat er verwezenlijkt is binnen de stageperiode, zowel op het vlak van kennis als persoonlijk. Ik ben tevreden met het eindresultaat, maar natuurlijk kijk ik er naar uit om in de toekomst zowel het project als mijn kennis van .NET te perfectioneren.
