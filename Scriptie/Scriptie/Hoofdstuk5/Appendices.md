# Appendices
In dit deel bevinden zich meer details over bepaalde zaken.

### Code
De belangrijkste code is gegroepeerd per actie en hieronder terug te vinden.

* ChangeRoles
```
        public void OnGet()
        {
            Customers = m_userManager.GetUsersInRoleAsync("Customer").Result.ToList();
            Agents = m_userManager.GetUsersInRoleAsync("Agent").Result.ToList();
            Admins = m_userManager.GetUsersInRoleAsync("Admin").Result.ToList();
        }
        //Delete User
        public IActionResult OnPost(string id)
        {
            var user = m_userManager.FindByIdAsync(id).Result;
            //first update security stamp van de gebruiker die we verwijderen
            m_userManager.UpdateSecurityStampAsync(user);
            m_userService.DeleteUser(id);

            
            // Clear the existing external cookie to ensure a clean login process
            //HttpContext.SignOutAsync(IdentityConstants.ExternalScheme);

            return RedirectToPage("../Admin/AdminChangeRoles");

        }
```

* Companies

```
public void OnGet(string searchString)
        {
            Companies = m_exactOnlineService.GetCompanies();

            ViewData["CurrentFilter"] = searchString;
            if (!String.IsNullOrEmpty(searchString))
            {
                Companies = Companies.Where(x => x.Name.ToLower().Contains(searchString.ToLower())).ToList();

                if (Companies.Count == 0)
                {
                    ModelState.AddModelError(string.Empty, "No matching results");
                }
            }
            
        }
```

* CompanyDetails

```
[Authorize(Roles = "Admin")]
    public class AdminCompanyDetailsModel : PageModel
    {
        public class InputModel
        {
            public Models.ContractLevel ContractLevel { get; set; }

        }

        [BindProperty]
        public InputModel Input { get; set; }

        private ExactOnlineService m_exactOnlineService;
        private UserInCompanyService m_userInCompanyService;
        private ApplicationDbContext m_context;
        private UserManager<ApplicationUser> m_userManager;

        public Companies Companies { get; set; }

        public List<UserInCompany> UserInCompany {get; set;}
        public List<ApplicationUser> UsersList { get; set; }

        public AdminCompanyDetailsModel(ExactOnlineService exactOnlineService, UserInCompanyService userInCompanyService, ApplicationDbContext context, UserManager<ApplicationUser> userManager)
        {
            m_exactOnlineService = exactOnlineService;
            m_userInCompanyService = userInCompanyService;
            m_context = context;
            m_userManager = userManager;
            this.UsersList = new List<ApplicationUser>();
        }
        public void OnGet(int id)
        {
            Companies = m_exactOnlineService.GetCompanyById(id);

            UsersInCompany(id);
        }

        public List<ApplicationUser> UsersInCompany(int id)
        {
            UserInCompany = m_userInCompanyService.GetUsersInCompany().FindAll(x => x.CompanyId == id).ToList();

            foreach (var item in UserInCompany)
            {
                var Gebruiker = m_userManager.FindByIdAsync(item.UserId).Result;
                UsersList.Add(Gebruiker);
            }

            return UsersList;
        }

        public IActionResult OnPostUpdateCompany(int id)
        {

            var updateCompany = m_context.Company.FirstOrDefault(x => x.Id == id);

            updateCompany.ContractLevel = Input.ContractLevel;
            m_context.SaveChanges();

            return RedirectToPage("../Admin/AdminCompanies");
        }
    }
```

* CreateUser

```
public IActionResult OnPost()
        {
            var user = new ApplicationUser { UserName = Input.Email, Email = Input.Email , Firstname =Input.Firstname, Lastname = Input.Lastname};
            var result = m_userManager.CreateAsync(user).Result;
            if (result.Succeeded)
            {
                //add user to role
                var test2 = m_userManager.AddToRoleAsync(user, Input.Role).Result;
                //add notifcation settings
                var userNotifications = m_userManager.Users.Include(x => x.Notifications).FirstOrDefault(x => x.UserName == user.UserName);
                userNotifications.Notifications.Add(new Models.Notifications()
                {
                    TicketCreated = true,
                    TicketUpdate = true,
                    NewArticle = true,
                    NewDocument = true
                });
                m_context.SaveChanges();

                //new account has been created
                //stuur een Email confirmatie mail  ....
                var code = m_userManager.GenerateEmailConfirmationTokenAsync(user).Result;
                var callbackUrl = Url.EmailConfirmationNewTicketLink(user.Id, code, Request.Scheme);

                m_mailmanager.SendEmailConfirmationAsync(Input.Email, callbackUrl, Input.Email);

                // zorg ervoor dat de gebruiker hier ook zijn passwoord kan setten
            }
            else
            {
                return RedirectToPage("../Error");
            }

            return RedirectToPage("../Admin/AdminChangeRoles");
        }
```

* DMS

```
        public IActionResult OnPostDeleteDirectory(int id)
        {
            m_documentManager.DeleteTargetDirectory(id);
            return RedirectToPage();
        }
        public IActionResult OnPostAdd()
        {
            //Save Path van directory voor documenten

            m_context.TargetDirectory.Add(new TargetDirectory
            {
                DirectoryPath = Input.NewPath
            });
            m_context.SaveChanges();
            return RedirectToPage("/Admin/AdminDMS");

        }
```

* Documents

```
        public void OnGet(string id)
        {
            DirectoryPath = id;

            string[] filesindirectory = Directory.GetFiles(DirectoryPath);

            FilePaths = filesindirectory.ToList();


        }

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

* DocumentSettings

```
        public void OnGet(int Id)
        {
            TargetDirectory = m_documentManager.GetTargetDirectoryByID(Id);

            string[] filesindirectory = Directory.GetDirectories(TargetDirectory.DirectoryPath);

            FilePaths = filesindirectory.ToList();
            Docs = m_context.DMS.ToList();
            //for elke filepath kijken of deze al bestaat, anders toevoegen aan DB
            foreach (var item in FilePaths)
            {
                var newPath = Docs.Find(x => x.FilePath == item);
                if (newPath == null)
                {
                    TargetDirectory.FilePaths.Add(new Models.DMS
                    {
                        FilePath = item,
                        DocumentLevel = Models.DocumentLevel.Invisible
                    });
                    //add new => document always invisible
                    m_context.SaveChanges();
                }
                else
                {
                    //path already in DataBase => content will still be available
                }
            }


            Docs = TargetDirectory.FilePaths.ToList();
        }

        public IActionResult OnPostUpdate()
        {
            //Update de Document levels van de bestaande paths
            foreach (var item in Docs)
            {
                var Document = m_context.DMS.FirstOrDefault(x => x.Id == item.Id);
                OldLevel = Document.DocumentLevel;

                var newLevel = m_context.DMS.FirstOrDefault(x => x.Id == item.Id);
                newLevel.DocumentLevel = item.DocumentLevel;

                m_context.DMS.Update(newLevel);
                m_context.SaveChanges();

                //controleren op nieuwe document wijzigingen??
                if (newLevel.DocumentLevel != OldLevel)
                {
                    if (newLevel.DocumentLevel == DocumentLevel.None)
                    {
                        LevelNone = LevelNone + 1;
                    }
                    if (newLevel.DocumentLevel == DocumentLevel.Bronze)
                    {
                        LevelBronze = LevelBronze + 1;
                    }
                    if (newLevel.DocumentLevel == DocumentLevel.Silver)
                    {
                        LevelSilver = LevelSilver + 1;
                    }
                    if (newLevel.DocumentLevel == DocumentLevel.Gold)
                    {
                        LevelGold = LevelGold + 1;
                    }
                }
            }

            //stuur mail naar contacten uit bedrijf waarop verandering document van toepassing is.
            CheckfornewItems(LevelNone, LevelBronze, LevelSilver, LevelGold);

            return RedirectToPage();
        }

        public void CheckfornewItems(int none, int bronze, int silver, int gold)
        {
            //Als document levels geupdate worden moeten we mails sturen naar users in companies die hun meldingen aan hebben staan
            var link = Url.Documents(Request.Scheme);
            //alle gebruikers die gekoppeld zijn aan een bedrijf
            UserInCompanyList = m_userInCompanyService.GetUsersInCompany();
            foreach (var GebruikersInBedrijf in UserInCompanyList)
            {
                //user object oprvagen voor elke user in bedrijf
                var UserObject = m_userManager.FindByIdAsync(GebruikersInBedrijf.UserId).Result;
                var CompanyObject = m_context.Company.FirstOrDefault(x => x.Id == GebruikersInBedrijf.CompanyId);

                if (CompanyObject.ContractLevel == ContractLevel.None)
                {
                    if (none != 0)
                    {
                        m_mailmanager.SendEmailNewDocument(UserObject.Email, link);
                    }
                }
                if (CompanyObject.ContractLevel == ContractLevel.Bronze)
                {
                    if (none != 0 || bronze != 0)
                    {
                        m_mailmanager.SendEmailNewDocument(UserObject.Email, link);
                    }
                }
                if (CompanyObject.ContractLevel == ContractLevel.Silver)
                {
                    if (none != 0 || bronze != 0 || silver != 0)
                    {
                        m_mailmanager.SendEmailNewDocument(UserObject.Email, link);
                    }
                }
                if (CompanyObject.ContractLevel == ContractLevel.Gold)
                {
                    if (none != 0 || bronze != 0 || silver != 0 || gold != 0)
                    {
                        m_mailmanager.SendEmailNewDocument(UserObject.Email, link);
                    }
                }

            }
        }
```

* ExactOnline

```
public void OnGet(string code)
        {

            Code = code;

            if (Code != null)
            {
                GetAPI(Code);
            }
        }
        public IActionResult OnPostOauth()
        {
            return Redirect("https://start.exactonline.be/api/oauth2/auth?client_id={*******
            }&redirect_uri=http://192.168.5.20:5001/Admin/AdminExactOnline&response_type=code");
        }
        public void GetAPI(string Code)
        {
            using (var client = new HttpClient())
            {
                var values = new Dictionary<string, string>
                {
                    { "code", Code },
                    { "redirect_uri", "http://192.168.5.20:5001/Admin/AdminExactOnline" },
                    { "grant_type", "authorization_code" },
                    { "client_id", "{208b2aab-272c-4123-a35d-fc872079bceb}" },
                    { "client_secret", "FcjRzukOQvEf" }
                };
                var content = new FormUrlEncodedContent(values);

                var method = new HttpMethod("POST");
                var request = new HttpRequestMessage(method, "https://start.exactonline.be/api/oauth2/token") { Content = content };
                var response = client.SendAsync(request).Result;

                //if the response is successfull, set the result to the workitem object
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
        public void GetDivision(string Token, string Type)
        {

            using (var client = new HttpClient())
            {
                //set our headers
                client.DefaultRequestHeaders.Accept.Clear();
                client.DefaultRequestHeaders.Accept.Add(new System.Net.Http.Headers.MediaTypeWithQualityHeaderValue("application/json"));
                client.DefaultRequestHeaders.Authorization = new AuthenticationHeaderValue("Bearer", Token);

                var method = new HttpMethod("GET");
                var request = new HttpRequestMessage(method, "https://start.exactonline.be/api/v1/current/Me?$select=CurrentDivision");
                var response = client.SendAsync(request).Result;

                //if the response is successfull, set the result to the workitem object
                if (response.IsSuccessStatusCode)
                {
                    var result = response.Content.ReadAsStringAsync().Result;

                    var rawResponse = JsonConvert.DeserializeObject<ExactDivision>(result);

                    foreach (var item in rawResponse.d.results)
                    {
                        var division = item.CurrentDivision;
                        GetAccountGUID(Token, division);
                    }

                }
            }
        }
        public void GetAccountGUID(string Token, int Division)
        {
            using (var client = new HttpClient())
            {
                //set our headers
                client.DefaultRequestHeaders.Accept.Clear();
                client.DefaultRequestHeaders.Accept.Add(new System.Net.Http.Headers.MediaTypeWithQualityHeaderValue("application/json"));
                client.DefaultRequestHeaders.Authorization = new AuthenticationHeaderValue("Bearer", Token);

                var method = new HttpMethod("GET");
                var request = new HttpRequestMessage(method, "https://start.exactonline.be/api/v1/" + Division + "/crm/Accounts?$filter=Status eq'C'");

                //Working API calls
                //Juiste !!!
                //crm/Accounts?$filter=Status eq'C'


                ///api/v1/{division}/crm/Contacts?$filter=ID eq guid'00000000-0000-0000-0000-000000000000'&$select=Account,AccountIsCustomer = lijst nog te groot, geeft klanten contact personen
                ///crm/Contacts?$filter=AccountIsCustomer eq true = contacten is customer geeft alle contacten van bedrijf terug
                ///crm/Accounts?$select=Type = geeft type A, accounts terug
                ///crm/Accounts?$select=AccountManager
                ///////////////////////////////////
                ///crm/Contacts?$filter=IsMainContact eq true and AccountIsCustomer eq true === te weinig data??? 
                //////////////////////////////////

                var response = client.SendAsync(request).Result;

                //if the response is successfull, set the result to the workitem object
                if (response.IsSuccessStatusCode)
                {
                    //als we tot hier geraakt zijn, gaan we ervan uit dat we alle contacten en companies hebben kunnen binnenhalen, 
                    //dus hier verwijderen we de contacten lijst om nadien een clean add te doen voor contacten per bedrijf.
                    if (m_context.Contacts.ToList() != null)
                    {
                        var OLDContactList = m_context.Contacts.ToList();
                        foreach (var contact in OLDContactList)
                        {
                            m_context.Contacts.Remove(contact);
                            m_context.SaveChanges();
                        }
                        
                    }



                    var result = response.Content.ReadAsStringAsync().Result;
                    var rawResponse = JsonConvert.DeserializeObject<ExactCompany>(result);

                    var list = m_context.Company.ToList();
                    foreach (var Company in rawResponse.d.results)
                    {

                        var result2 = list.FirstOrDefault(x => x.Name == Company.Name);
                        if (result2 == null)
                        {
                            m_exactOnlinService.AddCompany(Company.Name, Company.AddressLine1, Company.Email, Company.Code, Company.Website, Company.Phone, Company.Postcode, Company.CountryName, Company.City);
                        }
                        else
                        {
                            var updateCompany = m_context.Company.FirstOrDefault(x => x.Name == Company.Name);

                            updateCompany.Phone = Company.Phone;
                            updateCompany.Address = Company.AddressLine1;
                            updateCompany.Email = Company.Email;
                            updateCompany.Code = Company.Code;
                            updateCompany.Postalcode = Company.Postcode;
                            updateCompany.Country = Company.CountryName;
                            updateCompany.City = Company.City;

                            m_context.SaveChanges();

                        }
                        GetMainContact(Token, Division, Company.MainContact, Company.Name);
                    }
                }
            }
        }
        public void GetMainContact(string Token, int Division, string MainContact, string CompanyName)
        {
            using (var client = new HttpClient())
            {
                //set our headers
                client.DefaultRequestHeaders.Accept.Clear();
                client.DefaultRequestHeaders.Accept.Add(new System.Net.Http.Headers.MediaTypeWithQualityHeaderValue("application/json"));
                client.DefaultRequestHeaders.Authorization = new AuthenticationHeaderValue("Bearer", Token);

                var method = new HttpMethod("GET");
                var request = new HttpRequestMessage(method, "https://start.exactonline.be/api/v1/" + Division + "/crm/Contacts?$filter=ID eq guid'" + MainContact + "'");
                var response = client.SendAsync(request).Result;

                //if the response is successfull, set the result to the workitem object
                if (response.IsSuccessStatusCode)
                {
                    var result = response.Content.ReadAsStringAsync().Result;

                    var rawResponse = JsonConvert.DeserializeObject<ExactMainContact>(result);

                    foreach (var item in rawResponse.d.results)
                    {
                        m_exactOnlinService.AddContact(CompanyName, item.FullName, item.FirstName, item.LastName, item.Email, item.BusinessEmail, item.Phone, item.BusinessPhone, item.Mobile, item.BusinessMobile, item.JobTitleDescription);

                    }
                }
            }
        }
```

* Settings

```
        public void OnGet()
        {
            ApiKeys = m_userService.GetApiKeys(User.Identity.Name);
        }

        public IActionResult OnPostSetVSTSToken()
        {
            m_userService.AddAPIKey(Models.Source.VSTS, User.Identity.Name, Input.Key, DateTime.Now);
            // userService ophalen en de functie Add VSTS token aanroepen
            return RedirectToPage("../Admin/AdminSettings");
        }

        public IActionResult OnPostDeleteToken(int id)
        {
            m_userService.DeleteAPIKey(Input.id);
            return RedirectToPage("../Admin/AdminSettings");
        }
```

* Tickets

```
//onGet ticket by ID binnenhalen
        public IActionResult OnGet(int id)
        {
            CategoryList = m_ticketService.GetCategories();
            ApiKey = m_userService.GetApiKeys(User.Identity.Name);

            
            var APIkey = m_userManager.Users.Include(x => x.ApiKeys).FirstOrDefault(x => x.UserName == User.Identity.Name).ApiKeys.ToList();
            if (APIkey.Count() == 0)
            {
                //hier kan je nog error handling voorzien
            }
            else
            {
                foreach (var item in APIkey)
                {
                    if (item.Source == Source.VSTS)
                    {
                        var token = item.VSTStoken;
                        m_VSTSService.GetAllBugs(token);
                    }
                    else
                    {
                        //hier kan je nog error handling voorzien
                    }
                }
                
            }

            Ticket = m_ticketService.GetTicketById(id);
            Input = new InputModel()
            {
                TicketId = id
            };

            if (Ticket == null)
            { return RedirectToPage("../Tickets/MyTickets"); }


            UserDetails = m_userService.GetUserByMail(Ticket.TicketRequestor);
            if (m_userInCompanyService.GetUsersInCompany().FirstOrDefault(x => x.UserId == UserDetails.Id) == null)
            {
                Page();
            }
            else
            {
                CompanyDetails = GetCompanyDetails(UserDetails.Id);
            }
            

            return Page();

        }

        //Delete ticket
        public IActionResult OnPostDelete(int id)
        {
            Ticket = m_ticketService.DeleteTicket(id);

            return RedirectToPage("../Tickets/MyTickets");

        }

        //Update ticket
        public IActionResult OnPostUpdate(int id)
        {
           
            Ticket = m_ticketService.UpdateTicket(id, Input.TicketPriority, Input.TicketStatus);
            m_ticketService.AddCategoryToTicket(Ticket.TicketId, Input.Category);

            SendMailTicketUpdate(id);

            return RedirectToPage();
        }

        //Add reply
        public IActionResult OnPostReply(string ReplyContent)
        {
            SendMailTicketUpdate(Input.TicketId);

            m_ticketService.AddReply(User.Identity.Name, DateTime.Now, Input.ReplyContent, Input.TicketId);

            return RedirectToPage();
        }

        //Delete Reply
        public IActionResult OnPostDeleteReply(int id)
        {
            Reply = m_ticketService.DeleteReply(Input.ReplyId);

            return RedirectToPage();

        }

        //Add bug
        public IActionResult OnPostLink(string WorkItemId)
        {
            m_VSTSService.AddBug(Input.WorkItemID, Input.TicketId);

            return RedirectToPage();
        }

        public void SendMailTicketUpdate(int Id)
        {
            var callbackUrl = Url.TicketLink(Request.Scheme);
            m_mailmanager.SendEmailTicketUpdate(Input.TicketRequestor, callbackUrl, Id);
        }

        //public void GetUsersInCompany(string userId)
        //{
        //    var CompanyFromUser = m_userInCompanyService.GetUsersInCompany().FirstOrDefault(x => x.UserId == userId);
        //}
        public Companies GetCompanyDetails(string userId)
        {
            var CompanyFromUser = m_userInCompanyService.GetUsersInCompany().FirstOrDefault(x => x.UserId == userId);
            var company = m_context.Company.Include(x => x.Contacten).FirstOrDefault(x => x.Id == CompanyFromUser.CompanyId);
            return company;
        }
```

* CreateBug

```
        public IActionResult OnPostCreateBug()
        {
            var description = Input.Description + " " + Url.TicketLinkID(Request.Scheme);
            m_VSTSService.CreateBug(User.Identity.Name, Input.Title, description, Input.TicketId, Input.Project);

            return RedirectToPage("../Tickets/MyTickets");
        }
        
       //Service
       //Create Bug
        public void CreateBug(string name, string Title, string Description, int TicketId, string Project)
        {
            var user = m_userManager.Users.Include(x => x.ApiKeys).FirstOrDefault(x => x.UserName == name).ApiKeys.ToList();
            foreach (var item in user)
            {
                if (item.Source == Source.VSTS)
                {
                    var token = item.VSTStoken;


                    string _personalAccessToken = token;
                    string _credentials = Convert.ToBase64String(System.Text.ASCIIEncoding.ASCII.GetBytes(string.Format("{0}:{1}", "", _personalAccessToken)));

                    Object[] patchDocument = new Object[2];

                    patchDocument[0] = new { op = "add", path = "/fields/System.Title", value = Title };
                    patchDocument[1] = new { op = "add", path = "/fields/Microsoft.VSTS.TCM.ReproSteps", value = Description };

                    //use the httpclient
                    using (var client = new HttpClient())
                    {
                        //set our headers
                        client.DefaultRequestHeaders.Accept.Clear();
                        client.DefaultRequestHeaders.Accept.Add(new System.Net.Http.Headers.MediaTypeWithQualityHeaderValue("application/json"));
                        client.DefaultRequestHeaders.Authorization = new AuthenticationHeaderValue("Basic", _credentials);

                        //serialize the fields array into a json string
                        var patchValue = new StringContent(JsonConvert.SerializeObject(patchDocument), Encoding.UTF8, "application/json-patch+json");

                        var method = new HttpMethod("PATCH");
                        var request = new HttpRequestMessage(method, "https://intation-development.visualstudio.com/" + Project + "/_apis/wit/workitems/$Bug?api-version=2.2") { Content = patchValue };
                        var response = client.SendAsync(request).Result;

                        //if the response is successfull, set the result to the workitem object
                        if (response.IsSuccessStatusCode)
                        {
                            var result = response.Content.ReadAsStringAsync().Result;

                            var rawResponse = JsonConvert.DeserializeObject<Bug>(result);

                            AddBug(rawResponse.VstsBugId, TicketId);
                        }
                    }

                }

            }

        }
```

* CreateFeature

```
        public IActionResult OnPostCreateFeature()
        {
            var description = Input.Description + " " + Url.TicketLinkID(Request.Scheme);
            m_VSTSService.CreateFeature(User.Identity.Name, Input.Title, description, Input.TicketId, Input.Project);

            return RedirectToPage("../Tickets/MyTickets");
        }
        
        // Service
        public void CreateFeature(string name, string Title, string Description, int TicketId, string Project)
        {
            var user = m_userManager.Users.Include(x => x.ApiKeys).FirstOrDefault(x => x.UserName == name).ApiKeys.ToList();
            foreach (var item in user)
            {
                if (item.Source == Source.VSTS)
                {
                    var token = item.VSTStoken;

                    string _personalAccessToken = token;
                    string _credentials = Convert.ToBase64String(System.Text.ASCIIEncoding.ASCII.GetBytes(string.Format("{0}:{1}", "", _personalAccessToken)));

                    Object[] patchDocument = new Object[2];

                    patchDocument[0] = new { op = "add", path = "/fields/System.Title", value = Title };
                    patchDocument[1] = new { op = "add", path = "/fields/System.Description", value = Description };

                    //use the httpclient
                    using (var client = new HttpClient())
                    {
                        //set our headers
                        client.DefaultRequestHeaders.Accept.Clear();
                        client.DefaultRequestHeaders.Accept.Add(new System.Net.Http.Headers.MediaTypeWithQualityHeaderValue("application/json"));
                        client.DefaultRequestHeaders.Authorization = new AuthenticationHeaderValue("Basic", _credentials);

                        //serialize the fields array into a json string
                        var patchValue = new StringContent(JsonConvert.SerializeObject(patchDocument), Encoding.UTF8, "application/json-patch+json");

                        var method = new HttpMethod("PATCH");
                        var request = new HttpRequestMessage(method, "https://intation-development.visualstudio.com/" + Project + "/_apis/wit/workitems/$Feature?api-version=2.2") { Content = patchValue };
                        var response = client.SendAsync(request).Result;

                        //if the response is successfull, set the result to the workitem object
                        if (response.IsSuccessStatusCode)
                        {
                            var result = response.Content.ReadAsStringAsync().Result;

                            var rawResponse = JsonConvert.DeserializeObject<Bug>(result);

                            AddBug(rawResponse.VstsBugId, TicketId);

                        }
                    }
                }
            }  
```

* CreateTicket

```
ublic IActionResult OnPost()
        {
            

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
            m_ticketService.AddCategoryToTicket(ticket.TicketId, Input.Category);

            var user = new ApplicationUser { UserName = Input.TicketRequestor, Email = Input.TicketRequestor };
            var result = m_userManager.CreateAsync(user).Result;
            if (result.Succeeded)
            {
                //add user to role
                var test2 = m_userManager.AddToRoleAsync(user, "Customer").Result;
                //add notifcation settings
                var userNotifications = m_userManager.Users.Include(x => x.Notifications).FirstOrDefault(x => x.UserName == user.UserName);
                userNotifications.Notifications.Add(new Models.Notifications()
                {
                    TicketCreated = true,
                    TicketUpdate = true,
                    NewArticle = true,
                    NewDocument = true
                });
                m_context.SaveChanges();
                //new account has been created
                //stuur een Email confirmatie mail  ....
                var code = m_userManager.GenerateEmailConfirmationTokenAsync(user).Result;
                var callbackUrl = Url.EmailConfirmationNewTicketLink(user.Id, code, Request.Scheme);

                m_mailmanager.SendEmailConfirmationAsync(Input.TicketRequestor, callbackUrl, Input.TicketRequestor);

                // zorg ervoor dat de gebruiker hier ook zijn passwoord kan setten
            }
            else
            {
                //account already in DB
                //stuur mail dat er een ticket is aangemaakt
                SendMailTicketCreate(ticket.TicketId);
            }

           

            //na post van ticket, redirect naar my ticket lijst
            return RedirectToPage("../Tickets/MyTickets");
        }

        public void SendMailTicketCreate(int Id)
        {
            var callbackUrl = Url.TicketLink(Request.Scheme);
            m_mailmanager.SendEmailTicketCreate(Input.TicketRequestor, callbackUrl, Id);
        }
```

* Upload

```
public IActionResult OnPost()
        {
            if (InputFile == null)
            {
                return Page();
            }
            if (!InputFile.FileName.EndsWith(".xml"))
            {
                return Page();
            }
            else
            {
                var data = InputFile.ToByteArray();
                Stream stream = new MemoryStream(data);

                XmlDocument doc = new XmlDocument();
                doc.Load(stream);

                string json = JsonConvert.SerializeXmlNode(doc);

                var rawResponse = JsonConvert.DeserializeObject<Rootobject>(json);

                var test = rawResponse.eExact.Accounts.Account.ToList();

                var list = m_context.Company.ToList();

                foreach (var Company in test)
                {

                    var result = list.FirstOrDefault(x => x.Name == Company.Name);
                    if (result == null)
                    {
                        m_exactOnlineService.AddCompany(Company.Name, Company.Address.AddressLine1, Company.Email, Company.code, Company.HomePage, Company.Phone, Company.Address.PostalCode, Company.Address.Country, Company.Address.City);
                    }
                    else
                    {
                        var updateCompany = m_context.Company.FirstOrDefault(x => x.Name == Company.Name);

                        updateCompany.Phone = Company.Phone;
                        updateCompany.Address = Company.Address.AddressLine1;
                        updateCompany.Email = Company.Email;
                        updateCompany.Code = Company.code;
                        updateCompany.Postalcode = Company.Address.PostalCode;
                        m_context.SaveChanges();
                    }



                }

                return RedirectToPage("../Admin/AdminCompanies");
            }
```

* ManageArticle

```
public void OnGet(int id)
        {
            Article = m_knowledgeService.GetArticleById(id);
            AllCategories = m_knowledgeService.GetMaps();
            foreach (var item in AllCategories)
            {
                if (item.Articles == null)
                {
                    //no articles
                }
                else
                {
                    foreach (var Article in item.Articles)
                    {
                        if (Article.Id == id)
                        {
                            Category = m_knowledgeService.GetMapByID(item.Id);
                        }
                    }
                }
                
            }

        }

        public IActionResult OnPostUpdateArticle()
        {
            m_knowledgeService.UpdateArticle(Input.Id, Input.Title, Input.Content, Input.CategoryID);
            return RedirectToPage("../Knowledge/ManageArticles");
        }

        public IActionResult OnPostDelete()
        {
            m_knowledgeService.DeleteArticle(Input.Id);
            return RedirectToPage("../Knowledge/ManageKnowledge");
        }
```

* ManageArticles

```
public IActionResult OnPostCreateArticle()
        {
            m_knowledgeService.CreateArticle(Input.Title, Input.Content, Input.Id);
            var link = Url.Knowledgebase(Request.Scheme);

            var Customers = m_userManager.GetUsersInRoleAsync("Customer").Result.ToList();

            foreach (var Customer in Customers)
            {
                m_mailManager.SendEmailNewArticle(Customer.Email, link);
            }
           


            return RedirectToPage("../Knowledge/ManageArticles");
        }
```

* ManageKnowledge

```
    [Authorize(Roles = "Admin, Agent")]
    public class ManageKnowledgeModel : PageModel
    {
        public class InputModel
        {
            public string Title { get; set; }
        }
        [BindProperty]
        public InputModel Input { get; set; }

        public List<Models.KnowledgeMap> Maps { get; set; }
        

        private KnowledgeService m_knowledgeService;

        public ManageKnowledgeModel(KnowledgeService knowledgeService)
        {
            m_knowledgeService = knowledgeService;
        }
        public void OnGet()
        {
            Maps = m_knowledgeService.GetMapsAndArticles();
        }

        public IActionResult OnPostMap()
        {
            m_knowledgeService.CreateMap(Input.Title);
            Maps = m_knowledgeService.GetMapsAndArticles();

            return RedirectToPage("../Knowledge/ManageKnowledge");
        }
    }
```

* NotifcationSettings

```
public IActionResult OnPost()
        {
            UserNotifications = m_userManager.Users.Include(x => x.Notifications).FirstOrDefault(x => x.UserName == 
            User.Identity.Name);

            //delete settings
            m_context.Remove(UserNotifications.Notifications.First());

            //set new settings
            UserNotifications.Notifications.Add(new Models.Notifications()
            {
                NewArticle = Input.NewArticle,
                NewDocument = Input.NewDocument,
                TicketCreated = Input.TicketCreated,
                TicketUpdate = Input.TicketUpdate,
            });
            m_context.SaveChanges();

            return RedirectToPage();

        }
```

## Email Template

```
<div height="400">
<body style="margin: 0; padding: 0; max-width:400px;">
    <table border="0" cellpadding="0" cellspacing="0" style="font-family: Verdana; background:#F4F5F7">
        <tr>
            <td style="width:40px"></td>
            <td style="text-align:center; padding-top:20px; padding-bottom:20px;">
            </td>
            <td style="width:40px"></td>
        </tr>
        <tr>
            <td></td>
            <td bgcolor="bf2341" style="height:12px;"></td>
            <td></td>
        </tr>
        <tr>
            <td></td>
            <td bgcolor="white" style="text-align:center;">
                <img src="https://www.intation.eu/wp-content/uploads/2018/03/inSupportlogo.jpg" width="150">
            </td>
            <td></td>
        </tr>
        <tr>
            <td></td>
            <td bgcolor="white" style="padding-bottom: 50px; padding-left: 50px; padding-right: 50px;">
                <h2 style="color:#424242">A new account has been created.</h2>
                <p style="color:#424242"><br/>Your current login information is now:</p>
                <ul style="color:#424242">
                    <li>Username: {username}</li>
                </ul>
                <p style="color:#424242">You have to confirm this email address. You can do this by clicking on 'Confirm Email'.<br/></p>
            </td>
            <td></td>
        </tr>
        <tr>
            <td></td>
            <td bgcolor="white" style="text-align: center;">
                <a href="{url}" style="text-decoration:none; background-color:#bf2341; color: white; font-size:18px; border: 10px solid #bf2341;">Confirm Email</a>
            </td>
            <td></td>
        </tr>
        <tr>
            <td></td>
            <td bgcolor="white" style="padding-bottom: 50px; padding-top: 50px; padding-left: 50px; padding-right: 50px;">
                <p style="color:#424242">Kind regards,<br/>Intation</p>
            </td>
            <td></td>
        </tr>
        <tr>
            <td></td>
            <td bgcolor="#D6D7D9" style="height:5px;">
            </td>
            <td></td>
        </tr>
        <tr>
            <td></td>
            <td style="text-align:center; padding-top:20px; padding-bottom:20px;">
                <p style="color:#677178; font-size:13px">One-time email from Incontrol for password confirmation.<br/>Koralenhoeve 15, 2160 Wommelgem, Belgium<br/>Please do not reply to this Email.</p>
            </td>
            <td></td>
        </tr>
        <tr>
            <td colspan="3" bgcolor="bf2341" style="text-align:center; margin-top:3px; margin-bottom:3px;">
                <a href="http://www.intation.eu/" title="Go to Intation website" align="center" style="color: white; background-color: transparent; text-decoration:wavy; font-size: 12px;color: white">&#169; Intation</a>
            </td>
        </tr>
    </table>
</body>
</div>


