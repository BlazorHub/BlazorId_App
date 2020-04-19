# BlazorId_App
Sample Blazor Server Solution with IdentityServer and API\n
This Example solution demonstrates the following features:
* Integrate a standard Blazor Server application with IdentityServer using auth code flow with PKCE protection.
* Configure a custom user claim in Identity Server and propagate it to the application cookie during user authentication.
* Include the custom user claim in the access token when calling the API.
* Use a shared authorization policy to secure the application and API.

# Blazor App Features
**This application provides two protected User features:**
* **View App Claims:** Displays the claims of the current User identity for the application.
* **View API Claims:** Calls a test API, which is protected by IdentityServer. The API will return the user claims it received with the request as JSON to the application. The application then display those claims to the User.

* **Blazor Server Application**
   * Scaffolded Blazor Server template project, protected by Identity Server using Authorization Code flow w/PKCE
   * Has protected views that display the User Claim set for the application or api
      * Identity features protected by Authorization policy **CanViewIdentity** from BlazorId_Shared. 
      * The policy requires custom claim type **appRole_claim** with value **identity**. 
      * **CascadingAuthenticationState** is declared in file App.razor to support UI authorization functions
      * **AuthorizeView** component is used in file NavMenu.razor to show / hide menu options based on authorization state
* **User Claims API** 
   * Returns current user claims as Json 
   * Requires IdentityServer client scope **identityApi**
   * Requires user claim type **appUser_claim** with value  **identity**
   * Authorization policy from BlazorId_Shared is the same one used by the application.
* **IdentityServer Demo** 
   * Identity Server samples, quickstart #6 AspNet Identity
   * Standard IdentityServer Demo project 
      * .Net Identity - Users: Alice (privileged), Bob (non-privileged)
      * Custom Api Resource **identityApi**
      * Custom Identity Resource **appUser_claim**
      
 ## Configuration
### Custom User Claim - appUser_Claim
#### Application user claim
   * Required configuration for the custom user claim to be added to the application cookie during IdentityServer login:
   1. The claim must be present for the user in the user store.
   2. A custom Identity Resource must configured in Identity Server for the custom claim:
         ```c#
           new IdentityResource("appUser_claim", new []{"appUser_claim"})
         ```
   3. The client config in IdentityServer must include this identity resource in the client's **'Allowed Scopes'**:
         ```c#
            client.AllowedScopes = {"openid","profile","email","identityApi","appUser_Claim"};
         ```
   4. The client must request this scope in the Oidc configuration settings in Startup.ConfigureServices:
         ```c#
             options.Scope.Add("appUser_claim"); 
         ```
   5. The client must map the IdentityServer Claim type to the Blazor App Claim type in the Oidc configuration settings in Startup.ConfigureServices:
      ```c#
       options.ClaimActions.MapUniqueJsonKey("appUser_claim", "appUser_claim");
      ```
 #### API user claim
 *Additional configuration requirements for the custom user claim to be included in the Access token for the API
   1.  The API Resource configuration in IdentityServer must include the custom claim type in the api resource's **ClaimTypes**. The following code creates the API resource and assigns the claim type of **appUser_claim**:
 ```c#
   new ApiResource("identityApi", 
                   "Identity Claims Api", 
                    new []{"appUser_claim"})
 ```

         ```
