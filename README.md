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

There are two paths to logout:
- Initiated by the forum:
  - the user clicks the "Log Out" button;
  - discourse redirects to the UI passing the '?logout' parameter;
  - the UI forget the token and redirect back to the forum using the referrer.
- Initiated by the UI:
  - a request is sent by Angular to the API;
  - the token is invalidated;
  - a logout request is sent by the API to the forum;
  - the user is logged out of the forum and the UI.

## Configuration

This is done in the discours administration page, with:
```
SiteSetting.enable_sso = true
SiteSetting.sso_url = "https://c2corgv6.demo-camptocamp.com/auth"
SiteSetting.logout_redirect = 'https://c2corgv6.demo-camptocamp.com/auth?logout'
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

## Import from v5

Loading the v5 database
```
sudo -u postgres createdb c2corg
sudo -u postgres psql -d c2corg < c2corg_v5_dump.sql
```

Add missing columns:
```
ALTER TABLE punbb_topics ADD COLUMN first_post_id integer default 0;
UPDATE punbb_topics SET first_post_id = (select MIN(p.id) from punbb_posts as p where punbb_topics.id = p.topic_id);
```

A discourse installation in developer mode is required:
https://github.com/discourse/discourse/blob/master/docs/DEVELOPER-ADVANCED.md

```
git clone git@github.com:c2corg/discourse
cd discourse
gem install bundle --user-install
bundle install --path ~/gempath
```

A database user with enough rights is necessary:
```
sudo -u postgres createuser -s -P discourse_import
```

A database must be chosen:
Modify `config/database.yml` then export the database name.
```
export THEDB="discourse_development"
```

Tables need to be created:
```
PGHOSTADDR=127.0.0.1 PGUSER=discourse_import  PGPASSWORD=discourse_import bundle exec rake db:create db:migrate
```

Migration:
```
DATABASE_URL=postgres://discourse_import:discourse_import@127.0.0.1/$THEDB bundle exec ruby script/import_scripts/punbb.rb
```


## Testing (with standalone, non sso instance)

Set 'c2cc2cc2c' password to everyone:
```
update users set password_hash = 'a9f9571107c43a7554293c0537beb0a6467863204b3420bee464da165fc690b6', salt = '7b5e9c84645272ade56967e0db501f69';
```

Activate all emails:
`update email_tokens set confirmed = true;`

Promote a user admin:
`update users set admin = true where username = 'XXX';`

Promote a user moderator:
`update users set moderator = true where username = 'XXX';`


# Restore discourse data (for testing)
See https://meta.discourse.org/t/advanced-manual-method-of-manually-creating-and-restoring-discourse-backups/18273

Save:
```
sudo -u postgres psql -d discourse_development -f backup.sql
```


Restore:
```
sudo -u postgres psql discourse_development <<END
DROP SCHEMA public CASCADE;
CREATE SCHEMA public;
ALTER SCHEMA public OWNER TO discourse_import;
CREATE EXTENSION IF NOT EXISTS hstore;
CREATE EXTENSION IF NOT EXISTS pg_trgm;
END

sudo -u postgres psql -d discourse_development -f backup.sql
```


# Launch dev on 0.0.0.0
`PGHOSTADDR=127.0.0.1 PGUSER=discourse_import  PGPASSWORD=discourse_import bundle exec rails server -b 0.0.0.0 -p 3000`



# Setup discourse_development_full instance

Instructions to set up the full instance on the demo server.
The full instance contains the whole import from punbb.
It uses database discourse_development_full and runs outside docker.

git clone git@github.com:c2corg/discourse discourse_import
Change database in config/database.yml to discourse_development_full

DATABASE_URL=postgres://discourse_import:discourse_import@127.0.0.1/discourse_development_full bundle exec rake db:create db:migrate
DATABASE_URL=postgres://discourse_import:discourse_import@127.0.0.1/discourse_development_full bundle exec ruby script/import_scripts/punbb.rb

Create an admin account
DATABASE_URL=postgres://discourse_import:discourse_import@127.0.0.1/discourse_development_full bundle exec rake admin:create

Run with
DATABASE_URL=postgres://discourse_import:discourse_import@127.0.0.1/discourse_development_full bundle exec rails server -b 0.0.0.0 -p 2000

Open http://c2corgv6-demo.gis.internal:2000


(delete database: DATABASE_URL=postgres://discourse_import:discourse_import@127.0.0.1/discourse_development_full  bundle exec rake db:purge:all && bundle exec rake db:create db:migrate)
