# Researching Google Drive search and indexability in .NET

Resarch as of 10/11. 

## Overview 

- Initially design looked at tracking changes in GDrive into Az Cognitive search index
    - Ran into a number of problems, but is still possible. 
    - Ultimately abandonned due to cost, and complexity of ensuring access rights & security. 
- New direction looking into combining search results from Az Cognitive search and GDrive directly via GAPI calls 
    - Security guarenteed as we are direstly relying on signed in user accounts accessing Google's services 
- Looking into authentication: 
    - Initially looked at methods of silently supplying a MS supplied claim to GAPI, which would verify with MS and return a claim for the federated user in Workspace
    - User federation and OAuth2/OIDC lacked support for this
        - Federation only really designed to benefit sysadmins with managing all users in one place 
        - No benefit for app integrations 
    - Now looking to combine both MSAL and Google authentication schemes in a single sign in session to call respective APIs
- Looking at other available APIs beyond the simple drive files API in the Google API suite for a richer searching experience
- Combining results indexing
    - Possible easiest way is to have different sections under search for diffrent areas where results are found
        - Reduces the need to wait on all results to be returned and can return results from each API asynchronously as they resolve 
    - Discussion with MS placeholder meeting to discuss other methods scheduled for 15/11/23. 

## Auth 

The bulk of the research up until this point has focused on how to generate an authenticaion token to call the Google APIs on behalf of the user. 

The final application will have all users in Azure Entra ID (Azure AD) federated in Google Workspsace, but will primarily will be authenticated in the application using MSAL and Entra ID; it would in theory be possible to pass our use claims token to Google, for them to authenticate with Entra ID, and return a token on behalf of the corresponding federated user in Workspace, completely silently (without client involvement). 

<img src="https://tinypic.host/images/2023/11/14/DesiredAuthFlow.png" />

A compromised version of this:

<img src="https://tinypic.host/images/2023/11/14/Compromised-Auth-Flow.png" />


Unfortunately it became aparent that the auth libraries didn't support this, further reaching out on relevant forurms seemed to get a general consensus that OAuth2 and OIDC doesn't really support this, and that user federation is only really setup for the benefit of sysadmins of managing all user data in one place not for the benefit of application integrations. 

Thus the best way to move forward would be for a user, once signed in via MSAL and Entra ID, to then also be presented with the Google authentication scheme, thus triggering a redirect to Google sign in, if we can specify the email in this redirect (obtained via the MSAL claim), Google auth would know to then also forward on to MS SSO, which the user has already signed in so would automatically redirect back to Google sign in and then back to out app, creating a somewhat longer but mostly silent auth flow for the user. 

<img src="https://tinypic.host/images/2023/11/14/Current-Auth-Flow.png" />
