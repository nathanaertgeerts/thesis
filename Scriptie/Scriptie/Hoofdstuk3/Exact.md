# Exact Online Integratie
Het klantenbestand wordt vandaag beheerd in exact online, een uitgebreide business software - CRM tool voor bedrijven en zelfstandigen die enkele REST API's ter beschikking stelt. Deze tool wordt onder andere ook gebruikt voor de boekhouding en financiële gegevens. Wanneer een supportticket behandelt wordt geven we de supportmedewerker een overzicht in het portaal van de bijhorende bedrijfsgegevens. Om de bedrijfsgegevens te koppelen aan de juiste gebruikersaccounts van het customer service portaal wordt er een relatie opgezet in de database die kan worden ingesteld door de admin.

### REST API
Er wordt gecommuniceerd met exact online aan de hand van API-calls, De klantengegevens worden opgevraagd en hun details worden getoond. De klantobjecten van Exact worden gekoppeld aan accounts van het customer service portaal. Het is zeer belangrijk wanneer met dit repliceert om de redirect-url te veranderen. Dit moet je ook specifiëren wanneer je een applicatie toevoegd in het Exact Online portaal.

Het opvragen van de klantengegevens gaat in verschillende stappen:

#### 1) Autorisatie
Eerst moet je jezelf authentiseren met Exact Online, hiervoor gebruik je de redirect-url naar Exact Online. De redirect url moet je instellen bij het aanmaken van je token op het portaal van Exact Online. Vervolgens moet de webpagina in de applicatie worden omgeleid naar dezelfde url. 
```
public IActionResult OnPostOauth()
{
    return Redirect("https://start.exactonline.be/api/oauth2/auth?client_id=
    {****}&redirect_uri=http://192.168.5.20:5001/Admin/AdminExactOnline&response_type=code");
}
```
#### 2) Accesstoken
De response die verkregen wordt na het invullen van correcte identificatie gegegevens is een ACCESSTOKEN en Token Type (Bij een foute call vervalt de ACCESSTOKEN). De response is tevens een json object, om dit te kunnen uitlezen wordt dit gedeserialized met behulp van "Newtonsoft.Json", dit is een JSON framework voor .NET applicaties.

```
public void GetAPI(string Code)
{
      using (var client = new HttpClient())
      {
                var values = new Dictionary<string, string>
                {
                    { "code", Code },
                    { "redirect_uri", "http://192.168.5.20:5001/Admin/AdminExactOnline" },
                    { "grant_type", "authorization_code" },
                    { "client_id", "{******}" },
                    { "client_secret", "*****" }
                };
                var content = new FormUrlEncodedContent(values);

                var method = new HttpMethod("POST");
                var request = new HttpRequestMessage(method, "https://start.exactonline.be/api/oauth2/token") 
                { Content = content };
                var response = client.SendAsync(request).Result;

                if (response.IsSuccessStatusCode)
                {
                    var result = response.Content.ReadAsStringAsync().Result;
                    var rawResponse = JsonConvert.DeserializeObject<ExactToken>(result);
                    var ACCESSTOKEN = rawResponse.access_token;
                    var RefreshToken = rawResponse.refresh_token;
                    var TokenType = rawResponse.token_type;

                    GetDivision(ACCESSTOKEN, TokenType);
                }
      }
}
```
#### 3) Divisie
Vervolgens moet je met behulp van de Accestoken de ‘Divisie’ opvragen, de divisie is eigen aan Exact Online en bepaald je toegangsniveau.
```
var method = new HttpMethod("GET");
var request = new HttpRequestMessage(method, "https://start.exactonline.be/api/v1/current/Me?$select=CurrentDivision");
```
De response is een nieuwe token en je divisie, deze token kan in combinatie met je divisie gebruikt worden voor meerdere API calls.

#### 4) API Calls

##### Klanten accounts
Vervolgens worden de klanten accounts van Exact Online opgevraagd, dit kan door in onze request een filter op te stellen. De response json wordt opnieuw gedeserialized en de klanten accounts worden toegekend aan het company object van het customer service portaal.

```
var method = new HttpMethod("GET");
var request = new HttpRequestMessage(method, 
"https://start.exactonline.be/api/v1/" + Division + "/crm/Accounts?$filter=Status eq'C'");
            
```
##### Main contacts
Als laatste worden de hoofdcontact personen van de klant opgevraagd ook met behulp van een nieuwe filter. De hoofdcontact personen wordt toegevoegd aan het klantenobject
```
var method = new HttpMethod("GET");
var request = new HttpRequestMessage(method, 
"https://start.exactonline.be/api/v1/" + Division + "/crm/Contacts?$filter=ID eq guid'" + MainContact + "'");
          
```
De klantengegevens kunnen manueel gekoppeld worden door een button actie zodat de juiste informatie kan worden weergeven bij de tickets. 


#### Problemen
Voor de integratie met Exact Online was eerst gevraagd om te kijken naar API-mogelijkheden aangezien deze documentatie ook beschikbaar is op de website van Exact Online. Helaas kan een gegenereerde token geen verschillende beperkingen opleggen dan het account waarop deze token gecreëerd is. Intation beschikt maar over enkele accounts bij exact online waarop de leidinggevende toegang tot alles hebben. Aangezien dit wil zeggen dat ik via de API-calls dan toegang heb tot alle vertrouwelijke data en deze ook kan verwijderen hebben we beslist om geen token te creëren met volledige toegang voor testing/development purposes. Het aanmaken van eender welke extra account met al dan niet eventuele beperkingen resulteert in een maandelijkse extra fee voor deze account. Om deze extra kost te vermijden bij het ontwikkelen van een applicatie hebben we geopteerd om gebruik te maken van de exportfunctie van Exact Online naar een XML of CSV-file. 

### XML-upload
De data die hieruit verkregen wordt is te vergelijken met de data die je van een API-call zouden terugkrijgen, waardoor de functionaliteit verder kon worden uitgewerkt. Het verschil is dat er gebruik wordt gemaakt van een statisch dataformaat dat we handmatig moeten updaten. 

Het XML-bestand bevat veel klanteninformatie dat we moeten filteren op noodzaak in onze applicatie. Hiervoor is het XML-bestand onderzocht, dit bestand bevat Accounts, deze Accounts zijn bedrijven of klanten en hebben soms een lijst van contact objecten. ASP.NET Core heeft een ingebouwde functionaliteit om XML-bestanden te parsen. 

Ik heb een object aangemaakt dat de data van deze file kan opslaan en vervolgens een lijst weergeven van alle bedrijven. Om het XML bestand te uploaden wordt dit eerst omgezet in een ByteArray met behulp van de IFormFileExtension, de extensie zorgt ervoor dat je via databinding de file kan uploaden in de razor pages en vervolgens omzetten in een ByteArray om op te slaan. De array kan echter niet zomaar uitgelezen worden, daarom wordt deze omgezet in een stream, de stream valt perfect uit te lezen en zodat de data kan worden opgeslagen in onze database. Het bestand wordt vervolgens geserialized naar een JSON object zodat dit overeen zou komen met de data van onze API-calls, dit wordt gedeserialized om vervolgens de data aan het object toe te kennen en op te slaan in de database. 

```
var data = InputFile.ToByteArray();
Stream stream = new MemoryStream(data);

XmlDocument doc = new XmlDocument();
doc.Load(stream);

string json = JsonConvert.SerializeXmlNode(doc);
var rawResponse = JsonConvert.DeserializeObject<Rootobject>(json);

```

Dit JSON object is ook eenvoudiger om mee te werken dan het XML-bestand. Om het bedrijf en zijn data te kunnen koppelen aan een gebruiker heb ik een extra tabel aangemaakt die het companyID en UserID opslaat. Dit valt te vergelijken met hoe de rollen worden opgeslagen in ASP.NET Core. 

*De exact online integratie was succesvol afgerond met het uploaden van een XML-file, maar aan het einde van deze sprint is er beslist om toch een apart development account aan te maken en met de API te werken. De omvorming was niet geheel eenvoudig. Het waren maar kleine aanpassingen om de data te beheren, maar de moeilijkheid zat hem in de connectie met de API.*

### Resultaat
<img src="/Images/portal/exact2.png" height="400" />
<img src="/Images/portal/exact3.png" height="400" />
