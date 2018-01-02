# SP-ADFS-Customisations

Modules to change federation by SharePoint



Token Expiration and Sliding Sessions
One of the Adatum requirements was that the a-Portal application automatically sign users out after a defined period of inactivity, but allow them to continue working with the application without re-authenticating so long as they remain active. To achieve this, Adatum implemented a sliding-session mechanism in SharePoint that can renew the SharePoint session token. For performance reasons, Adatum wanted to be able to extend the lifetime of the session token without having to revisit ADFS (the federation provider) or the partner's token issuer.

A cookie (usually named FedAuth) that can exist either as a persistent or in-memory cookie represents the SharePoint session token. This cookie contains a reference to the SAML token that SharePoint stores in its token cache. The SAML token contains the claims issued to the user by any external identity and federation providers, and by the internal SharePoint security token service (STS).


## Sliding sessions 

Sliding Sessions in SharePoint
<P>
Adatum wanted to implement sliding sessions so that SharePoint can extend the lifetime of the session if the user remains active. Adatum wanted to be able to define an inactivity period, after which SharePoint forces the user to re-authenticate with ADFS. In other words, a user will only need to sign in again if the session is allowed to expire or if the SAML token expires. In this scenario, the session lifetime will be less than the SAML token lifetime.
To implement this behavior, Adatum first configured ADFS to issue SAML tokens with a lifetime of eight hours. The following Microsoft Windows® PowerShell® command-line interface script shows how you can configure this setting in ADFS for the SharePoint Adatum Portal relying party.
</P>
Add-PSSnapin Microsoft.ADFS.PowerShell
Set-AdfsRelyingPartyTrust –TargetName "SharePoint Adatum Portal" –TokenLifeTime 480

By setting the LogonTokenCacheExpirationWindow value to 470 minutes, Adatum can effectively set the session duration to 10 minutes.

$ap = Get-SPSecurityTokenServiceConfig <BR>
$ap.LogonTokenCacheExpirationWindow = (New-TimeSpan -minutes 470) <BR>
$ap.Update(); <BR>
IIsreset <BR> 

Adatum then modified the way that SharePoint manages its sessions. SharePoint now recreates a new session before the existing session expires (as long as the user visits the SharePoint web application before the existing session expires). A user can continue to recreate sessions up to the time that the SAML token finally expires; in this scenario, the user could continue using the a-Portal web application for eight hours without having to re-authenticate. If the user doesn't visit the web application before the session expires, then on the next visit he must sign in again. The Microsoft Visual Studio® development system solution, SlidingSessionModule, found in the 10SharePoint folder from http://claimsid.codeplex.com includes a custom HTTP module to deploy to your SharePoint web application that includes this functionality. The following code sample from the Adatum custom HTTP module shows the implementation.

This method first determines the valid from time and valid to time of the existing session, taking into account the value of the LogonTokenCacheExpirationWindow property. Then, if the existing session is more than halfway through its lifetime, the method uses the SPSessionAuthenticationModule instance to extend the session. It does this by creating a new session that has the same lifetime as the original, but which has a ValidFrom property set to the current time.
The sequence diagram in Figure 5 shows how SharePoint handles Adatum's sliding-sessions implementation.

## Reference 
https://msdn.microsoft.com/en-us/library/hh446526.aspx 

