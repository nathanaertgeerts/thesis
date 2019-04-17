# Document Management Systeem
Document management is hoe iemand zijn elektronische documenten beheerd. Een document management systeem zorgt ervoor dat deze documenten eenvoudig beheerd kunnen worden. Intation wilt elektronische documenten beschikbaar stellen voor zijn klanten aan de hand van hun contractlevel. De documenten worden opgeslagen in sharepoint, sharepoint is een platform van Microsoft dat het mogelijk maakt om informatie te delen en samenwerking binnen een organisatie online te bevorderen.

Om dit te voltooien moeten er een integratie worden voorzien zodat de documenten vanuit Sharepoint beschikbaar zijn. Vervolgens is het noodzakelijk dat de beschikbare downloads gelimiteerd kunnen worden aan de hand van het contractlevel.

### Beschikbare documenten
Eerst en vooral moeten de documenten die aangeboden worden als digitale download beschikbaar zijn op de server. Hiervoor is een lokale sharepoint map opgezet die automatisch synchroniseert met de online wijzigingen. Dit heeft als gevolg dat onze documenten altijd lokaal beschikbaar en up-to-date zullen zijn. 

Vervolgens moet een administrator een path kunnen opslaan in de Database. Er kan wegens security redenen geen 'folder-picker' mogelijkheid voorzien worden vanuit het portaal. Dit zou wil zeggen dat de gebruikers de volledige mappen structuur kunnen onderzoeken van de server. Er wordt dus een suggestie gedaan aan de administrator van het lokale path, waarin de documenten zich bevinden. Binnen Intation wordt steeds dezelfde structuur voor documenten van producten en projecten gehanteerd.

### Locaties van bestanden 
Nu het path naar de juiste folder beschikbaar is, kan er voor elk document in deze map het path opgevraagd worden. Vervolgens wordt het filepath opgeslagen in de database.
```
string[] filesindirectory = Directory.GetDirectories(TargetDirectory.DirectoryPath);

FilePaths = filesindirectory.ToList();

foreach (var item in FilePaths)
{
     ...
}
```

### Downloads
ASP.NET beschikt over File providers waardoor we toegang kunnen geven tot lokale bestanden en deze retourneren als download in de browser aan de gebruiker. Het bestands-type en de naam voor de download worden gespecifieerd en het bestand wordt opgehaald met behulp van de File provider en het filepath dat is opgeslagen in de database. 
```
public FileResult OnPostDownload()
{
     var fileName = Input.fileName;
     var DirectoryPath = Input.DirectoryP;
     var filePath = DirectoryPath + "/" + fileName;
     var fileExists = System.IO.File.Exists(filePath);
     var mimeType = System.Net.Mime.MediaTypeNames.Application.Octet;
     return PhysicalFile(filePath, mimeType, fileName);
}
```
Een volledige map stellen we beschikbaar als zip-bestand. Er wordt lokaal op de server een map aangemaakt met een zip extensie waarin de gevraagde documenten worden geplaatst.
```
public ActionResult OnPostDownload(string path)
{
      ZipFile.CreateFromDirectory(path, "support.zip");
      var mimeType = System.Net.Mime.MediaTypeNames.Application.Octet;
      return PhysicalFile(contentRootPath + "/support.zip", mimeType, "support.zip");
}
```
### Restricties
Voorlopig zijn alle documenten zichtbaar en beschikbaar voor iedereen. Om dit te limiteren is er geopteerd om de documenten een level toe te kennen aan de hand van een Enum zodat deze nadien kunnen worden vergeleken met het contractlevel van een bedrijf. Dit geeft ons de mogelijkheid om restricties op te leggen en te specifieren welke downloads al dan niet beschikbaar zijn voor bepaalde klanten. Wanneer een map of filepath wordt opgeslagen krijgen deze de enum 'invisible' toegekend, dit zorgt ervoor dat in de razorpages de documenten onzichtbaar kunnen gemaakt worden voor de gebruiker. 

Nadien kan er per document een level worden ingesteld, de beschikbare levels zijn zoals de contractlevels: 'None = zichtbaar voor iedereen, Bronze = zichtbaar vanaf een bronze contractlevel, Silver = zichtbaar vanaf een silver contractlevel en Gold = enkel zichtbaar met een gouden contractlevel'. 
```
SilverMaps = m_context.DMS.Where(x => x.DocumentLevel == DocumentLevel.None ||
                                      x.DocumentLevel == DocumentLevel.Bronze ||
                                      x.DocumentLevel == DocumentLevel.Silver).ToList();
```

```
if (CompanyDetails.ContractLevel == ContractLevel.Silver)
{
    Documents = SilverMaps;
}
```

Dit wil zeggen alvorens een gebruiker op het portaal toegang heeft tot een document hij gekoppeld moet zijn aan een bedrijf. De administrator kan gebruikers koppelen aan een bedrijf in het admin dashboard. Hiervoor is een one-to-many relatie opgesteld tussen een bedrijf en gebruikers (een bedrijf kan meerdere gebruikers bevatten). Er wordt gebruik gemaakt van een nieuwe database tabel die het user-ID en het company-ID opslaat. Deze werking is te vergelijken met gebruikers en rollen.
```
m_context.Add(new UserInCompany()
{
      UserId = UserId,
      CompanyId = CompanyId
});
```

### Resultaat
Klanten kunnen bestanden individueel downloaden of mappen als een gecomprimeerd bestand. De administrators kunnen documenten toevoegen door het juiste path op te slaan. Vervolgens kunnen restricties worden opgelegd aan de hand van contract- en documentlevels.

<img src="/Images/portal/dms.png" height="400" />
