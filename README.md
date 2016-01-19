# Authentication

## Principle
Discourse is configured to delegate SSO authentication to the UI.
There are two paths to authenticate, depending on the domain where :
- Initiated by the forum:
  - the user clicks on the login button in the forum;
  - discourse passes a nonce to the UI authentication page (sso and sig parameters);
  - the user enters its credentials and a request is sent by Angular to the API;
  - the API checks the credentials and returns the token and user properties signed for discourse;
  - the UI page stores the token in WEB storage and redirects to discourse, passing the signed properties;
  - the user is logged on the UI and the forum; a user is automatically created in discourse.
- Initiated by the UI:
  - the user clicks on the connection button in the UI;
  - the user enters its credentials and a request is sent by Angular to the API;
  - the API checks the credentials and prepares the token;
  - a request is sent to the forum to retrieve a nonce (sso and sig);
  - user properties are signed for discourse and is returned to the browser together with the token;
  - an iframe is created to have the forum set its cookies, a user is automatically created in discourse;
  - the user is logged on the UI and the forum; the user is redirected to some UI page.

## Configuration

This is done in the discours administration page, with:
```
SiteSetting.enable_sso = true
SiteSetting.sso_url = "https://c2corgv6.demo-camptocamp.com/auth"
SiteSetting.sso_secret="some secret string"
```

Users are automatically created in discourse, from the information provided
by the c2c UI (actually retrieved from the c2c API) during login.

Users are prevented to change their username or email in discourse:
```
SiteSetting.sso_overrides_username = true
SiteSetting.sso_overrides_name = true
SiteSetting.sso_overrides_email= true
```

