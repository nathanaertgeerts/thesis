# Visual Studio Team Services Integratie
Visual Studio Team Services is een cloudservice specifiek voor het samenwerken aan de ontwikkeling van software. VSTS beschikt over ingebouwde functionaliteit zoals een versie controle systeem, agile scrum methodes en meer. Om connectie te maken met Visual Studio Team Services heeft een support medewerker een persoonlijke API-key nodig die hijzelf kan aanmaken in het webportaal van VSTS. Wanneer een ticket behandeld wordt moet de support medewerker kunnen zien of hier reeds gekoppelde bugs of features aan toebehoren. Een support medewerker kan zelf een bug of feature aanmaken vanuit het customer service portaal.

Visual Studio Team Services beschikt over een uitgebreide en goed gedocumenteerde API-bibliotheek waar ik mezelf eerst in moest verdiepen. Vervolgens hebben we samen de gewenste functionaliteiten samengesteld gebaseerd op de mogelijkheden met behulp van de API’s. Als eerste moet er een API-token worden aangevraagd om op een veilige manier GET en POST Request te kunnen sturen met een VSTS account. Ten tweede is het antwoord van een API-call een JSON, dit moet omgezet worden in een object zodat  deze data kan worden uitgelezen en toegekend. Met behulp van de NewSoft JSON library kan de JSON worden gedeserialized in een object en de data worden opgeslagen in de database. 

### API calls
Elke support medewerker moet een eigen token aanmaken op VSTS om toegang te krijgen tot zijn projecten. De token moet dus instelbaar zijn per gebruiker, in het dashboard is daarom een invoerveld voorzien waarbij een token kan worden ingevoerd en gekoppeld via de include functionaliteit van ASP.NET Core aan onze ApplicationUser. De gebruiker kan zijn API-key ook verwijderen wanneer gewenst en een nieuwe toe voegen wanneer deze vervallen is. Dit is efficiënt gemaakt door in de user interface links toe te voegen waardoor het voor de gebruiker eenvoudig is om te navigeren naar de instellingen.

Om een bug of feature aan te maken wordt er gebruik gemaakt van een POST request. Een Bug of feature moet in VSTS een titel en  beschrijving bevatten. Dit wordt geserialized naar een JSON object en vervolgens meegestuurd in de POST request header. Het is ook van belang om een link te kunnen leggen tussen een ticket en een bug of feature. Hiervoor is een bug model aangemaakt en het ticket model aangepast zodat deze objecten van een bug kan bevatten. Wanneer een koppeling wordt gemaakt wordt in de database het ticket ID opgeslagen bij de bijhorende bug zodat deze eenvoudig kan worden opgevraagd en getoond worden bij het juiste ticket. De API-calls zijn eerst getest via postman en nadien met behulp van een C# voorbeeld van Visual Studio Team Services geschreven in C# in het asp.net project. De onderstaande code is een voorbeeld van een POST request om een feature of bug aan te maken.
```
var token = item.VSTStoken;

string _personalAccessToken = token;
string _credentials = Convert.ToBase64String(System.Text.ASCIIEncoding.ASCII.GetBytes(
string.Format("{0}:{1}", "", _personalAccessToken)));

Object[] patchDocument = new Object[2];
patchDocument[0] = new { op = "add", path = "/fields/System.Title", value = Title };
patchDocument[1] = new { op = "add", path = "/fields/System.Description", value = Description };

using (var client = new HttpClient())
{
      client.DefaultRequestHeaders.Accept.Clear();
      client.DefaultRequestHeaders.Accept.Add(new 
      System.Net.Http.Headers.MediaTypeWithQualityHeaderValue("application/json"));
      client.DefaultRequestHeaders.Authorization = new AuthenticationHeaderValue("Basic", _credentials);

      var patchValue = new StringContent(JsonConvert.SerializeObject(patchDocument), 
      Encoding.UTF8, "application/json-patch+json");

      var method = new HttpMethod("PATCH");
      var request = new HttpRequestMessage(method, 
      "https://intation-development.visualstudio.com/" + Project + "/_apis/wit/workitems/$Feature?api-version=2.2") 
      { Content = patchValue };
      var response = client.SendAsync(request).Result;

      if (response.IsSuccessStatusCode)
      {
            var result = response.Content.ReadAsStringAsync().Result;
            var rawResponse = JsonConvert.DeserializeObject<Bug>(result);
            AddBug(rawResponse.VstsBugId, TicketId);
       }
}

```
Om de feature ook gebruiksnut te geven was het essentieel dat de bugs worden weergegeven bij het ticket met hun onderwerp, ID, ‘Assigned To’ en door hier op te klikken eenvoudig genavigeerd kan worden naar de juiste VSTS-bug of feature. Als je op verschillend bugs klikt moet je ook het project meegeven wat verschillend kan zijn, dit heb ik opgelost door dit op te vragen wanneer een bug of feature wordt aangemaakt. 
```
 <a href="https://intation-development.visualstudio.com/@Bug.ProjectName/_workitems?id=@Bug.VstsBugId">@Bug.VstsBugId</a>
```
Het kan zijn dat een ticket betrekking heeft tot een reeds bestaande bug of feature. Daarom is de optie voorzien om deze manueel te koppelen door het ID van een bestaande bug mee te geven. De ID van bugs en features zijn te vinden in VSTS.


### Resultaat
We kunnen met behulp van REST API's met deze service communiceren, hiervoor moeten we eerst een token genereren. Deze token is specifiek voor een gebruiker en je kan hieraan verschillende permissies toekennen. De functionaliteit die we geïmplementeerd hebben in de webapplicatie zorgt ervoor dat tickets eenvoudig gekoppeld kunnen worden aan bugs of features. Wanneer een support medewerker een ticket behandelt kan hij eenvoudig de status en andere gegevens van de bug of feature bekijken om een gedetailleerd antwoord te kunnen geven.

<img src="/Images/portal/bug.png" height="400"/>
<img src="/Images/portal/ticketbug.png" height="400"/>

