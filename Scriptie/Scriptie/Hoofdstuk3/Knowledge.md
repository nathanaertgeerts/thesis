# Knowledge Base
Een knowledge base of kennisbank is een collectie van informatie of 'kennis'. Een knowledge base gaat vaak samen met een ticket systeem doordat herhaalde vragen kunnen omgezet worden in een artikel waardoor mensen sneller geholpen kunnen worden.
De support medewerkers moeten folders en artikels kunnen aanmaken en bewerken. De klanten moeten de knowledgebase kunnen doorzoeken.

### Artikels en folders
Om folders aan te maken en nadien artikels te kunnen toevoegen wordt er zoals bij het ticket systeem gebruik gemaakt van de CRUD-operaties. De mappen en artikels hebben een one-to-many relatie zoals de antwoorden op tickets, waardoor er eveneens gebruik kan worden gemaakt van de include property in ASP.NET Core. 

Mappen en artikels moeten aangemaakt kunnen worden door Agents en Administrators, hiervoor is een nieuwe razorpage gekoppeld aan de knowledgebase sectie op het portaal. Om artikels een rijke context te kunnen meegeven maken we gebruik van dezelfde summernote editor als bij de tickets. Hierdoor kan men afbeeldingen en allerlei eenvoudig opslaan als string in de database.

### Knowledgebase doorzoeken
De knowledgebase kan eenvoudig doorzocht worden dankzij de methode Contains(). De Contains methode maakt het mogelijk om na te gaan of een string een bepaalde string bevat. De gebruiker kan dus op de webpagina een bepaalde zoekterm ingeven en de zoekterm wordt omgezet in kleine letters met de methode ToLower(). Vervolgens kijken we of de titel van een map of de content van een artikel de zoekterm bevat. Indien de zoekterm voorkomt in een titel of tekst zullen we enkel de gevonden folders of artikels tonen.
```
ViewData["CurrentFilter"] = searchString;
if (!String.IsNullOrEmpty(searchString))
{
    Maps = Maps.Where(x => x.Title.ToLower().Contains(searchString.ToLower())).ToList();

    Articles = Articles.Where(x => x.Content.ToLower().Contains(searchString.ToLower())).ToList();
}
```

### Resultaat
Support medewerkers kunnen artikels en folders aanmaken. Artikels kunnen allerlei informatie bevatten en gebruikers kunnen eenvoudig zoeken tussen de folders of gebruik maken van de zoekfunctie.
<img src="/Images/portal/knowledge2.png" height="400"/>
