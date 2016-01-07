# Authentication

## Principle
Discourse is configured to delegate SSO autentication to the UI.
- discourse passes a nonce to the UI authentication page (sso and sig parameters);
- the user enters its credentials and a request is sent by Angular to the API;
- the API checks the credentials and returns the token and user properties signed for discourse;
- the UI page stores the token in WEB storage and redirects to discourse, passing the signed properties;
- the user is logged on the UI and the forum; eventually, a matching user is automatically created in discourse.

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

