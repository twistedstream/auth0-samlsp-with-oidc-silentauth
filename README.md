# SAML SP with OIDC Slient Authentication

## Overview

This is an example of a simple SAML Service Provider using Auth0 as an IDP. After logging into the Service Provider it allows the end user to use OIDC [silent authentication](https://auth0.com/docs/api-auth/tutorials/silent-authentication) to get an `access_token` in the frontend which can be used to fetch the user profile and call their own API. 

Essentially we're embedding a frontend (JavaScript-based) client within the HTML that's being rendered by the SAML SP, and its that frontend client that's making calls to an API, not the SP. Therefore the server-based SAML SP and the fontend client are technically separate apps. For this reason, we have you set up separate clients in Auth0 so they can be configured independently.

## Setup

### SAML SP App setup in Auth0

1. Go to [Clients tab](https://manage.auth0.com/#/clients) in the Auth0 Dashboard and create a new **Regular Web Application** client with the name `SAML SP App`
2. Go to **Settings**
3. Make a note of the **Client ID** of the client
4. Add the following to the **Allowed Logout URLs**:

   ```
   http://sp.myapp.local:5000/
   ```

3. Click on **Addons** and enable **SAML2 Web App**
4. Set the **Application Callback URL** to: `http://sp.myapp.local:5000/assert`
5. Under **Settings** add:

    ```json
    {
      "audience": "http://sp.myapp.local/metadata.xml",
      "recipient": "http://sp.myapp.local/assert",
      "nameIdentifierFormat": "urn:oasis:names:tc:SAML:1.1:nameid-format:unspecified",
      "nameIdentifierProbes": [
        "http://schemas.xmlsoap.org/ws/2005/05/identity/claims/nameidentifier",
        "http://schemas.xmlsoap.org/ws/2005/05/identity/claims/emailaddress",
        "http://schemas.xmlsoap.org/ws/2005/05/identity/claims/name"
      ],
      "signatureAlgorithm": "rsa-sha256"
    }
    ```

    and save the settings

6. Under **Connnections** enable `Username-Password-Authentication` as a connection for the client
7. Under the **SAML2 Web App** addon go to Usage and download the IDP certificate for your account

### Frontend App setup in Auth0

1. Create a new **Single Page Web Application** client in Auth0 with the name `Frontend App`
2. Go to **Settings**
3. Make a note of the **Client ID** of the client
4. Under the **Settings** tab, set the  **Allowed Callback URLs** to:

   ```
   http://sp.myapp.local:5000/silentauth-callback
   ```

### Resource Server setup in Auth0

1. Go to https://manage.auth0.com/#/apis
2. Create a new API with following settings
   - Identifier : `urn:gateway:api`
   - Allow Skipping User Consent: true
   - Signing Algorithm: RS256
   - Scopes: `read:foo`

3. Add the following scope to the API:
   - Name: `read:foo`
   - Description: `Read your foo`

### Local Setup

1. Clone this repo

2. Copy the IDP certificate file you downloaded under [Auth0 IDP Setup](#auth0-) Step 7 to the root folder of the app and rename it to `idp.pem`

3. Add a hosts file entry into your computer: 

    ```
    127.0.0.1       sp.myapp.local
    ```

4. Run the following command to install all the dependencies:

    ```bash
    npm install
    ```

5. Create a local `.env` file with the following values:

    ```
    SP_DOMAIN=sp.myapp.local
    AUTH0_DOMAIN=your-account.auth0.com
    AUTH0_SP_CLIENT_ID=client_id-of-SAML-SP-App
    AUTH0_FRONTEND_CLIENT_ID=client_id-of-Frontend-App
    API_AUDIENCE=urn:gateway:api
    ```

## Run

1. Run the node application by running:

    ```bash
    npm start
    ```

2. Open your browser and go to: http://sp.myapp.local:5000/
3. It redirects to auth0 for login and you can enter a username/password from the connection you activated for this app
4. On sign in the SAML assertion is posted to: http://sp.myapp.local:5000/assert 
5. On this page you can click on the button and get a token for your API
