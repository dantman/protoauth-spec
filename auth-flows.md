User Auth Flows
===============
This page does not directly standardize any part of ProtoAuth but instead defines the high-level user flows that ProtoAuth is built to support. This is used as a reference when deciding what types of low-level authorization flows to support, what token grant types we need, and what kind of extra parameters should be included in the spec. This page is also intended to help implementors understand how to work the low-level authorization flows into the many actual high-level flows -- something that both versions of OAuth appear to be really quiet and unhelpful about.

Web Server Flow [Universal]
---------------
Pattern:
* User visits client application's website and is sent to the authorization server.
* User authorizes the client.
* Site redirects user to the client's redirection endpoint.
* Client exchanges data passed back to it for tokens.

Notes:
* OAuth 2 provides this with an "Authorization Code Flow".
* Are there any cases where a public (non-confidential) client would want to use this flow?
* The redirect URI MUST only be `http://` or `https://` as clients using custom protocols should use the implicit grant.
* This is one of the only flows that can have permanent authorizations with refresh tokens.
* Codes are not granted until authorization. So there is no loop to close if the user just closes the browser.
* Codes are exchanged directly after an automatic redirection. So there is no loop to close with authorized codes and the can expire very quickly.
* Exchanging authorization code for access token MUST require a request_uri parameter that matches the one used in the original authorization request. See http://hueniverse.com/2011/06/oauth-2-0-redirection-uri-validation/

Pure-JS Web App Flow [Universal]
--------------------
Pattern:
* User visits client application's website and is sent to the authorization server.
* User authorizes the client.
* Site redirects user to the client's redirection endpoint with a token in the fragment.

Notes:
* OAuth 2 provides this with an "Implicit Grant Flow".
* redirect_uri MUST be known beforehand.
* Flows like this cannot be secure if the authorization server does not use HTTPS.
* Tokens from this method should not last long and will require re-authorization.
* Tokens are not granted until authorization. So there is no loop to close if the user just closes the browser.
* The authorization server and resource server require CORS support for this method to work.

Device Redirection Flow [Universal]
-----------------------
Pattern:
* User opens up client app on the device and the app opens up the browser pointing to the authorization server.
* User authorizes the client.
* Site redirects user to the client's redirection endpoint with a token in the fragment.
* Device allows client to intercept the URL of the redirection endpoint.
* Client gets the token from the fragment of the URL.

Notes:
* OAuth 2 provides this with an "Implicit Grant Flow".
* redirect_uri MUST be known beforehand.
* Flows like this cannot be secure if the authorization server does not use HTTPS.
* This is the same flow as the Pure-JS Web App Flow. However we typically want device authorizations to last longer. How do we handle this?
  * One option might be to make a mixed method where an authorization code rather than a token is returned inside of the fragment and then the device app exchanges that for an auth code.
* Tokens are not granted until authorization. So there is no loop to close if the user just closes the browser.

Application Token Copy Flow [Rejected?]
---------------------------
Pattern[A:Token]:
* User opens up desktop application on their computer and clicks on a button that opens up their browser pointing to the authorization server.
* User authorizes the client.
* Authorization server displays an access token to the user in a Copy & Paste-able box.
* User copies access token and pastes it into client.
* Client uses the token that was pasted into it.

Note:
* Tokens are not granted until authorization. So there is no loop to close if the user just closes the browser.
* This method directly exposes the token to the clipboard. Malicious apps will easily gain access to it. And also exposes the token to non-malicious apps that log the clipboard.
* With this method the application cannot know when the user explicitly rejected the application.

Pattern[B:Code]:
* User opens up desktop application on their computer and clicks on a button that opens up their browser pointing to the authorization server.
* User authorizes the client.
* Authorization server displays an authorization code to the user in a Copy & Paste-able box.
* User copies authorization code and pastes it into client.
* Client exchanges the authorization code for a token.

Note:
* Codes are not granted until authorization. So there is no loop to close if the user just closes the browser.
* This method directly exposes the authorization code to the clipboard without a client_secret to restrict it to the client. A malicious app can easily try to exchange the authorization code for a token itself. Since the malicious app would be polling the clipboard it probably will also get the authorization code faster than the application itself.
* With this method the application cannot know when the user explicitly rejected the application.

Pattern[C:Poll]:
* User opens up desktop application on their computer.
* Client makes a request to the authorization server for a temporary token.
  * Client continually polls / pings the authorization server at the rate specified to keep the temporary token alive.
* Potentially after the user clicking on something the client opens up the user's browser pointing to an authorization server URL.
* User authorizes the client.
* Authorization server displays an authorization code to the user in a Copy & Paste-able box.
* User copies authorization code and pastes it into client.
* Client uses the temporary token to exchange the authorization code for a token.

Note:
* In this method the application continually polls the server. As a result we can close the loop on the server by expiring the temporary token when the application is no longer polling the server.
* Secrets have already been exchanged between the client and server in this method. Is there any reason anymore to force the user to Copy & Paste a code?
* The client is polling the server so it can be told when the user rejects the application.

Application Polling Flow [Universal]
------------------------
Pattern:
* User opens up desktop application on their computer.
* Client makes a request to the authorization server for a temporary token.
  * Client beings to continually poll the authorization server at the rate specified waiting for the token to be authorized.
* Potentially after the user clicking on something the client opens up the user's browser pointing to an authorization server URL.
* User authorizes the client and is told they can then close the page.
* Client stops polling when the authorization server responds that the user has given authorization and is given an access token.

Todo:
* OAuth 1 had a phishing issue that caused the need for a verifier. Double check that we didn't fall into that trap with this flow.

Device and Application Password Flow [Universal]
------------------------------------
Pattern:
* User opens up client app on the device or application on their computer.
* User enters their username and password into the device or application.
* Client uses username and password to ask authorization server for access token.
* Authorization server validates the username and password and then hands over an access token.
* Device or application discards the password and exclusively used the access token.

Notes:
* Use of this flow may be useful for trusted device apps and desktop apps which can probably log keystrokes anyways.
* This flow gives full rights to the application.
* Malicious apps can still screen scrape your site and log in with a username and password there so there is no real advantage to restricting or not offering this flow. This flow is still better than normal password auth because the application can discard the password after logging in.
* <del>Access tokens returned via this method should not be restrained to short times. As forcing short times would create the incentive to store the password which would be insecure.</del>
* <del>An expires parameter should be used to let the client define when the access token should expire. This will allow indefinitely authorized in clients while also allowing clients that don't want indefinite tokens to let their own token expire.</del>
* An alternative use case for password flows may be a replacement for HTTP Authorization and/or something in browser JS that allows a login form to use our auth system in place of cookies.

Spare Key Application Flow
--------------------------
Pattern:
* User opens up application on their computer.
* User opens up a special page on the authorization/resource server and requests a valet key.
* User pastes valet key into application.
* Client exchanges one-time valet key for authorization token.

Notes:
* This flow still gives full rights to the application but it has the advantage that the user is not typing their password into the desktop application.
* The utility of a token like this might be debatable considering most situations it would be used can probably use one of the other token flows which can define scope and client.

Provider Facilitated Application Download Flow [Proprietary]
----------------------------------------------
Pattern:
* User logs into service provider website and looks at a list of 3rd party applications and then initiates a download for one of them.
* Provider creates a long one-time request token valid for a reasonable period of time [define recommendation!].
* Provider bakes this request token into the application's download code and hands the application data over to be downloaded.
* User installs the downloaded application.
* On first-opening the application the client exchanges the one-time request token for an authorization token.

Notes:
* 
* Most of this cannot be made to work generically for everything and will be provider specific and only work for proprietary APIs. Our goal here will simply be to specify the method that a client exchanges the one-time request token for an authorization token.

Pending Approval Queue Flow [Rejected]
---------------------------
Pattern:
* User opens up client app on the device and enters their username.
* Client notifies the authorization server that the user is waiting on authorization of an application.
* Authorization server returns temporary credentials to the client.
* User opens up the providers website and logs in.
* Website displays a notification to the user saying something like "You have an application waiting for approval." and follows the link.
* User authorizes the app.
* The client which is polling for authorization is given an access token by the authorization server.

Notes:
* This falls into the temporary credentials trap in a case where we cannot easily close the loop.
* This flow makes it very easy to spam/harass a user. Anyone can make an application request so all you need is someone's username and you can flood their approval queue spamming them notifications about it.

Device PIN Input Flow [Rejected?]
---------------------
Pattern:
* User opens up client app on the device and is given instructions.
* User visits the URL on the authorization server from the instructions using another computer.
* User authorizes the client from there.
* Authorization server displays a PIN to the user.
* User types the PIN into their device.
* Client exchanges code for a token.

Notes:
* OAuth 1 can do this using an oob callback. eg: https://dev.twitter.com/docs/auth/pin-based-authorization
* A PIN MUST NEVER be less than 4 digits.
* For the optimum usability and security balance either 6 or 8 digits is recommended.
* Authorization servers should consider shortening the expiration time for shorter codes. (?)
* Endpoint needs params that let the client tell the authorization server how many digits are supported<del> and if letters</del> are supported.
* Flaw: We need to know what scope the client wants before the user can authorize the client. As a result under this flow the url that the user enters ends up being a complex and ugly URL rather than a simple web page URL that any part of the site can link to.
  * Use of something like OAuth 1's temporary credentials can reduce the URL data but it the URL still remains complex.
  * Because the user is entering the complex URL they are practically already entering a code into the webpage beforehand. If we use temporary credentials then there isn't really any advantage to the PIN.

Device PIN Display Flow [Universal]
-----------------------
Pattern:
* User opens up client app on the device.
* Client makes a request to the authorization server for a temporary token and a PIN.
* Client app displays the PIN to the user.
* User visits an authorization server URL using another computer.
* User enters PIN into a form.
* User authorizes the client.
* Client either polls the authorization server or waits for the user to press a button on the device.
* Client now gets a real token from the authorization server.

Notes:
* This falls back to the downsides of temporary credentials.
  * That said that trap is probably unavoidable to do PINs in a user friendly way.
  * If polling is used we could perhaps time out temporary credentials really quickly but use the polling to extend that time while the device is still waiting.
    * Though devices that go to sleep while the user is doing the authorization could be an issue.

Pure-JS Mobile Web App PIN Flow?
-------------------------------
[Meant for devices like B2G]

[Idea: QR Code Device Flow; Dead-End]
-------------------------------------
Pattern:
* User opens up client app on the device and is given instructions.
* User visits an authorization server URL using another computer.
* Authorization server displays a QR Code to the user. (This could actually include discovery information)
* User uses client app to scan the QR code.
* Client app makes a request to the authorization server for temporary credentials attached to the QR Code's authorization.
* User authorizes the client.
* Client either polls the authorization server or waits for the user to press a button on the device.
* Client now gets a real token from the authorization server.

Notes:
* Thinking about it this would require both the device to poll for when the credentials are authorized and the authorization to poll itself to know when the QR Code has been scanned.

[Idea: Hybrid Flows]
--------------------
For some of these flows there is a potential to introduce a "Web Server Flow" style web server into the mix even when we are using things like device flows. The idea being that the web server can have a client_secret and permit the use of refresh tokens. This would let us have a device flow for the user but do a server side grant and get a refresh token then pass an access token back to the client. The client could then talk to the server whenever it needs another access token.

Then again this idea may be a complete dead end. Needs more thought.

It's possible that an issue may moot this idea making it's security no good. Though at the same time we should take into consideration an other case. In flows like the "Device PIN Display Flow" the client ends up needing to poll the server. An alternate to this is to; Have an extra server into the mix. After the authorization is done redirect the web browser to a special redirect_uri. From there the server for the device client's provider would use Android or iOS' cloud messaging system to send a notification to the device that it has been authorized.

@todo Browser Add-on Flows
--------------------------
I need to look over what interfaces Chrome and Firefox provide to their add-ons and identify the most user friendly flows to use for them.

@todo Client Credentials Grant
------------------------------
...
