# <a name="top"></a>SparkOAuth.py

- [Overview](#overview)
- [Creating a Spark Application](#app_create)
- [Configuring the Sample](#config)
- [Using the Sample](#usage)
- [Code Walkthrough](#code_walkthrough)
- [Editing the Sample](#sample_edit)


## <a name="overview"></a>Overview

**SparkOAuth.py** is a sample Python application that demonstrates how to use the [Spark authentication APIs](http://docs.sparkauthentication.apiary.io/) to perform the OAuth 2.0 authentication required for accessing the Spark HTTP service 3D printing APIs.<br/>
Note that the generated authentication tokens are valid only for a single session.


## <a name="app_create"></a>Creating a Spark Application

To use the sample, you must first create (register) an application on the Spark developer portal:

1. Sign up to the [Spark developer portal](https://spark.autodesk.com/developers/).

2. Create a new application [here](https://spark.autodesk.com/developers/getStarted).

3. <a name="cb_url_set"></a>In the app settings, set your *Callback URL* to http://localhost:8089/callback.

4. Copy and save the <a name="app_key"></a>*App Key* and <a name="app_secret"></a>*App Secret* values from the app-registration *API Keys* tab.


## <a name="config"></a>Configuring the Sample

Before using the sample, you must configure it to use your custom App Key and App Secret, by editing the **SparkOAuth.py** sample as follows:

1. Replace "*YOUR_CONSUMER_KEY*" in the **CONSUMER_KEY** variable assignment with your [App Key](#app_key).

2. Replace "*YOUR_CONSUMER_SECRET*" in the **CONSUMER_SECRET** variable assignment with your [App Secret](#app_secret).

**NOTE:** The sample uses `TODO` comments to mark required configuration steps.<br/>
You can delete these comments once you performed the relevant steps.


## <a name="usage"></a>Using the Sample

To start using the sample, simply browse http://localhost:8089/signin.


## <a name="code_walkthrough"></a>Code Walkthrough

The authentication logic is implemented in the **MyHandler** class, which demonstrates how to use the [Spark Authentication APIs](http://docs.sparkauthentication.apiary.io/) to perform OAuth 2.0 authentication and generate an [access token](http://docs.sparkauthentication.apiary.io/#reference/oauth-2.0/access-token):

1. <a name="code_login"></a>**Member log-in/registration** is handled in the sample's **do_GET** "**signin**" code segment, which is activated when browsing the [sign-in page](#usage). The **s.send_header** API call launches the log-in page (stored in the **loginUrl** variable). The sample uses the following log-in URL (where *{CONSUMER_URL}* is replaced with the App Key you assigned to the **CONSUMER_KEY** variable in the [Configuration Step](#config)): *https://sandbox.spark.autodesk.com/api/v1/oauth/authorize?client_id={CONSUMER_KEY}&response_type=code*

2. <a name="code_cb"></a>**Callback-URL processing and access-token generation** is handled in the sample's **do_GET** "**callback**" code segment, which is activated when Autodesk loads the registered callback URL, after the user logs in. The code performs the following actions:
  -  **Prepare the request's parameters string**: Callback parameters are passed to the app within a query string ('*?...*'); the parameters are separated by ampersands ('*&*'):<br/>
*{callback URL}?{param name}={param value}[&{param2 name}={param2 value}...]*<br/>
The authorization code that is required for completing the authentication is stored in the *code* parameter. For example, if the code is "*xyz*", the callback URL may look like this: *http://<span></span>localhost:8089/callback?code=xyz*.<br/>
  The handler parses the callback URL parameters string (see the **parsedParams** variable assignment), and extracts the value of the authorization-code parameter (see the **code** variable assignment). Then it prepares a parameters string  for the access-token generation request, using the extracted code (see the **payload** variable assignment). 

  - **Prepare the HTTP headers:** The handler encodes the [App Key](#app_key) and  [App Secret](#app_secret) using [base64 encoding](https://www.base64encode.org/), by grouping the values into a single *{App Key}:{App Secret}* string and passing it to the **base64.b64encode** API (see the **encodedAuthorizationHeader** variable assignment). The handler then prepares the required HTTP authorization and content-type headers, using the encoded string (see the **headers** variable assignment; see also the initial headers preparation in the **do_HEAD** code block.)

  - **Issue an access-token generation request** — see the **requests.post** API call. The result is assigned to the **res** variable.
  
  - **Extract the access token** from the result string — see the **json.loads** API call. The result is assigned to the **token** variable.


### <a name="sample_edit"></a>**Editing the Sample**

- **Callback URL** — The samples utilizes the [callback URL](#cb_url_set) that you defined in your app settings on the Spark developer portal. This URL is implicitly assigned to the **redirect_uri** parameters of the log-in URL and the callback's access-token generation request. To use a different callback URL, change your app settings.

- **Production** — To use the sample in the production stage, you will need to set the *Callback URL* in your production app settings on the Spark developer portal to the relevant URL; replace the sandbox URL prefix set in the sample's **SPARK_ENDPOINT** variable with your production URL; and replace the App Key and App Secret values in the sample's **CONSUMER_KEY** and **CONSUMER_SECRET** variable assignments with the production credentials provided on the developer portal.

- **Callback verification** — When using a custom callback URL, you may also want to set the optional **state** parameter by adding *&state={unique ID}* to the **loginUrl** string. You can then edit the callback code to check the value of the **state** parameter returned by the callback URL, and verify that the callback is linked to your original request.

- <a name="implicit_login"></a>**Implicit log-in** — If the value of App Secret in your app code (see the **CONSUMER_KEY** variable) cannot be hidden from the end user, you will need to modify the sample to implement implicit log-in. This is a less secure form of log-in, which does not require calling the access-token generation API.<br/>
  To implement implicit login, change the value of the **response_type** parameter in the **loginUrl** string from "*code*" to "*token*", and extract the access token from the callback URL's query string (see the **parsedParams** variable) instead of from the **requests.post** result.
  
- **Guest token** — You can generate a [guest token](http://docs.sparkauthentication.apiary.io/#reference/oauth-2.0/guest-token/generate-a-guest-token) instead of an access token to grant access to all public data on the Spark developer portal but not to private user data (as there's no user authentication). To generate a guest token you need to change the value of the *grant_type* parameter in the **payload** string from "*authorization_code*" to "*client_credentials*".

- **Token refreshing** — Access tokens are valid for 2 hours. You can edit the sample to call the Spark 
  [Access Token Refresh API](http://docs.sparkauthentication.apiary.io/#reference/oauth-2.0/access-token-refresh/refresh-an-access-token) to generate a new access token without requiring the user to log-in again. Refresh tokens are also valid for 2 hours and may be refreshed in the same manner. Note that [implicit log-in](#implicit_login) access tokens cannot be refreshed.
