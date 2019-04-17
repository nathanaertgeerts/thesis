# Logboek Week #10
### Dinsdag
* Error handling en lockout van verwijderde user. =>
* Update security stamp wanneer we deze verwijderen/aanpassen:
m_userManager.UpdateSecurityStampAsync(user);
en in startup: call naar database:
* services.Configure<SecurityStampValidatorOptions>(options =>
{
    // enables immediate logout, after updating the user's stat.
    options.ValidationInterval = TimeSpan.Zero;   
});
* opgepast kan zware load zijn voor DB => voert 5 queries uit (AspNetUser, AspNetUserClaims, AspNetUserRoles,AspNetRoles, AspNetRoleClaims) 
    
* Input velden required maken en error-handling
* Input velden maximum lenght beperken
* Forgot password layout
* Reply ticket text formateren
* Product icon aanpassen
* customer geen bug icon 
    
### Woensdag
* Pagina's beveiligen
* error handling met verwijderde user (security stamp vernieuwen)
* Link in navbar met home verwijderen
* Horizontale scrollbalk verwijderen op homepage
* verwijderen 2 factor auht
* Manage account phone number validation verwijderen
### Donderdag
* Sprintmeeting 
* Opzetten document management systeem
* Sync docs met local directory 
* Documenten/Directories uitlezen en laten zien
* Opzetten van Mogelijkheid om verschillende documenten met verschillende contract levels te laten zien
### Vrijdag
* Downloaden van documenten
* Verschillende paths kunnen uitlezen/toevoegen en opslaan in de database
* Admin DMS view, DMS front end
* Permissies voor verschillende users en documenten
