---
layout: post
title: Demonstrating OAuth 2.0 using Facebook's Graph API
date: 2018-05-10 13:55:37.000000000 +05:30
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories:
- SSO
- SSS
- Web Development
tags:
- Authorization
- Facebook Graph API
- Facebook PHP SDK
- Laravel
- OAuth
- OAuth 2.0
- PHP
meta:
  _thumbnail_id: '415'
  _rest_api_published: '1'
  _rest_api_client_id: "-1"
  _publicize_job_id: '17689229987'
  timeline_notification: '1525940738'
author: amodsachintha
excerpt: OAuth 2.0 is regarded as the industry-standard protocol for authorization. It
  works by delegating user authentication to the service that hosts the user account
  (trusted identity provider), and authorizing third-party applications to access
  the user account.
---
<h1>Introduction</h1>
<p>OAuth 2.0 is regarded as the industry-standard protocol for authorization. It works by delegating user authentication to the service that hosts the user account (<em>trusted identity provider</em>), and authorizing third-party applications to access the user account.</p>
<p>This post focuses on implementing the OAuth flow to authorize a Facebook app to access user data.</p>
<p><a href="https://github.com/amodsachintha/oauth-csrf-sss" target="_blank" rel="noopener">repo@github</a></p>
<p><a href="https://github.com/amodsachintha/oauth-csrf-sss/blob/master/app/Http/Controllers/FacebookController.php" target="_blank" rel="noopener">FacebookController</a></p>
<h2>OAuth Roles</h2>
<ul>
<li><strong>Resource Owner</strong> - A user who authorizes an application to access their data.</li>
<li><strong>Client</strong> - The application that wants to access protected resources.</li>
<li><strong>Resource Server</strong> - The service that holds user data</li>
<li><strong>Authorization Server</strong> - This service verifies the identity of a pending user.</li>
</ul>
<p>The <em>Resource Server</em> and <em>Authorization Server</em> roles can be (and usually is) combined to form an <strong>API</strong> that fulfills both service roles.</p>
<p><!--more--></p>
<h2>Protocol Flow</h2>
<p><img class="alignnone size-full wp-image-407" src="{{ site.baseurl }}/assets/oauth_flow.jpg" alt="OAuth 2.0 Flow" width="554" height="370" /> High level overview of OAuth protocol flow</p>
<ol>
<li>The application requests authorization to access protected resources from the User.</li>
<li>When the user authorizes the request, the application receives an authorization grant. (The grant types will be discussed briefly later on)</li>
<li>The application requests for an Access Token from the authorization server (API) in exchange for the authorization grant. (Application identity details too are sent along with the authorization grant to the API token endpoint)</li>
<li>The authorization server (API) issues an Access Token when both application identity and authorization grant are validated.</li>
<li>The Application requests the protected resource from the API while presenting the Access Token for authentication.</li>
<li>If the token is valid, the resource endpoint serves the resource to the application.</li>
</ol>
<h2>Application Registration</h2>
<p><img class="alignnone  wp-image-409" src="{{ site.baseurl }}/assets/app_registration_fb.png" alt="app_registration_fb" width="507" height="439" /> Application Registration fallout on Facebook Developers</p>
<p>Before using OAuth with an application, the application must be first registered with the service.</p>
<h4 id="client-id-and-client-secret">Client ID and Client Secret</h4>
<p>Once the application is registered, the service will issue "client credentials". It consists of a <em>client identifier </em>and a <em>client secret</em>. (A username-and-password-like combination) The Client ID is a publicly exposed string that is used by the service API to identify the application. The Client Secret is used to authenticate the identity of the application to the service API when the application requests to access a user's account. It must be kept private between the application and the API. Implementing client side code with the Client Secret in plain is a huge security risk.</p>
<h4>Callback URLs</h4>
<p><img class="alignnone size-full wp-image-410" src="{{ site.baseurl }}/assets/fb_callbacks.png" alt="fb_callbacks" width="713" height="323" /> Callback URLs for the application</p>
<p>Redirect URLs must be pre-configured into the service before exposing them via the client code. These will determine where a user is redirected upon the authorization result.</p>
<h2 id="authorization-grant">Authorization Grant</h2>
<p>OAuth 2.0 defines four grant types, each of which is useful in different cases:</p>
<ul>
<li><strong>Authorization Code</strong>: used with server-side Applications</li>
<li><strong>Implicit</strong>: used with Mobile Apps or Web Applications (applications that run on the user's device)</li>
<li><strong>Resource Owner Password Credentials</strong>: used with trusted Applications, such as those owned by the service itself</li>
<li><strong>Client Credentials</strong>: used with Applications API access</li>
</ul>
<h2>Authorization Code Grant Type</h2>
<p>The <em>authorization code</em> grant type is the most commonly used because it is optimized for <em>server-side applications.</em> Because the server-side source code is not publicly exposed, <em>Client Secret</em> confidentiality can be maintained.</p>
<h4>Authorization Code Link</h4>
<p><img class="  wp-image-412 aligncenter" src="{{ site.baseurl }}/assets/fb_login.png" alt="fb_login" width="380" height="256" /></p>
<p>The user is prompted to click on a button or link that resembles the following format.</p>
<pre>https://www.facebook.com/v2.12/dialog/oauth?client_id=161640204541883&amp;state=2081c5c62987cf1a760e3833295f3ea2&amp;response_type=code&amp;sdk=php-sdk-5.6.2&amp;redirect_uri=https%3A%2F%2Fwww.oauthtest.lk%2Ffbcallback&amp;scope=email%2Cpublic_profile%2Cuser_posts%2Cuser_friends%2Cpublish_actions</pre>
<p>The link components are as follows.</p>
<ul>
<li><strong>https://www.facebook.com/v2.12/dialog/oauth</strong>: the authorization endpoint of Facebook's graph API v2.12.</li>
<li><strong>client_id</strong>: the application's client ID</li>
<li><strong>state: </strong>a random string that serves CSRF protection</li>
<li><strong>response_type: </strong>code - specifies that the application is requesting an authorization code grant</li>
<li><strong>redirect_uri: </strong>where the service redirects the user-agent after an authorization code is granted</li>
<li><strong>scope: </strong>specifies the level of access and/or permissions to resource groups.</li>
<li><strong>sdk: </strong> php sdk version. This is optional</li>
</ul>
```php
    public function show()
    {
        $fb = new Facebook(
            'app_id' => env('FB_APP_ID'),
            'app_secret' => env('FB_APP_SECRET'),
            'default_graph_version' => env('API_VERSION'),
        ]);
        $helper = $fb->getRedirectLoginHelper();
        $permissions = ['email', 'public_profile', 'user_posts', 'user_friends', 'publish_actions',];
        $loginUrl = $helper->getLoginUrl('https://www.oauthtest.lk/fbcallback', $permissions);
        $url = htmlspecialchars($loginUrl);
        return view('index')->with('url', $url);
    }
```
<h4>User Authorizes the application</h4>
<p>When the user clicks the link, the user-agent (browser) is redirected to the identity service (i.e. Facebook) to authenticate the user's identity. The user must login to the service if they're not logged in already.</p>
<p><img class=" size-full wp-image-414 aligncenter" src="{{ site.baseurl }}/assets/authentication.png" alt="authentication" width="610" height="662" /></p>
<h4 id="step-3-application-receives-authorization-code">Application Receives Authorization Code</h4>
<p>When the user authorizes the application, the service redirects the user-agent to the application redirect URL, which was specified during the client registration, along with an <em>authorization code </em>and the <em>state </em>parameter. The redirect would look something like this.</p>
<p><img class="alignnone size-full wp-image-415" src="{{ site.baseurl }}/assets/callback_img.png" alt="callback_img" width="751" height="187" /></p>
<h4 id="step-4-application-requests-access-token">Application Requests Access Token</h4>
<p>The application requests an access token from the API, by passing the authorization code along with authentication details, including the <em>client secret</em>, to the API token endpoint.</p>
<pre class="_5s-8 prettyprint lang-code prettyprinted"><span class="pln">GET https</span><span class="pun">:</span><span class="com">//graph.facebook.com/</span><code><span class="com">v2.12</span></code><span class="com">/oauth/access_token?</span><span class="pln"> client_id</span><span class="pun">={</span><span class="pln">app</span><span class="pun">-</span><span class="pln">id</span><span class="pun">}</span> <span class="pun">&amp;</span><span class="pln">redirect_uri</span><span class="pun">={</span><span class="pln">redirect</span><span class="pun">-</span><span class="pln">uri</span><span class="pun">}</span> <span class="pun">&amp;</span><span class="pln">client_secret</span><span class="pun">={</span><span class="pln">app</span><span class="pun">-</span><span class="pln">secret</span><span class="pun">}</span> <span class="pun">&amp;</span><span class="pln">code</span><span class="pun">={</span><span class="pln">code</span><span class="pun">-</span><span class="pln">parameter</span><span class="pun">} </span></pre>
<h4></h4>
<h4 id="step-5-application-receives-access-token">Application Receives Access Token</h4>
<p>If the authorization is valid, the API will send a response containing the access token  to the application.</p>
<p><img class="alignnone size-full wp-image-416" src="{{ site.baseurl }}/assets/access_token.png" alt="access_token" width="724" height="128" /></p>

```php
     $helper = $fb->getRedirectLoginHelper();
     $_SESSION['FBRLH_state'] = $_GET['state'];
     try {
        $accessToken = $helper->getAccessToken(); //the SDK does a post to the token endpoint
        } catch (FacebookSDKException $e) {
            echo 'Facebook SDK returned an error: ' . $e->getMessage();
            exit;
        }
```
    
