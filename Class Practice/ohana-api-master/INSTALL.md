# Running Ohana API on your computer

## Fork and clone

[Fork this repository to your GitHub account][fork].

Clone it on your computer and navigate to the project's directory:

    git clone https://github.com/<your GitHub username>/ohana-api.git && cd ohana-api

[fork]: http://help.github.com/fork-a-repo/

## Docker Setup (recommended, especially for Windows users)

1. Download, install, and launch [Docker]

1. Set up the Docker image and start the app:

        $ script/bootstrap

Once the docker images are up and running, the app will be accessible at
[http://localhost:8080](http://localhost:8080).

### Verify the app is returning JSON

[http://localhost:8080/api/locations](http://localhost:8080/api/locations)

[http://localhost:8080/api/search?keyword=food](http://localhost:8080/api/search?keyword=food)

We recommend the [JSONView][jsonview] Google Chrome extension for formatting
the JSON response so it is easier to read in the browser.

[jsonview]: https://chrome.google.com/webstore/detail/jsonview/chklaanhfefbnpoihckbnefhakgolnmc

### More useful Docker commands

* Stop this running container: `docker-compose stop`
* Stop and delete the containers: `docker-compose down`
* Open a shell in the web container: `docker-compose run --rm web bash`

[Docker]: https://docs.docker.com/engine/installation/

## Local Setup

Before you can run Ohana API, you'll need to have the following software
packages installed on your computer: Git, Postgres, Ruby 2.3+,
a Ruby version manager such as chruby, rvm, rbenv, or asdf, and Google Chrome.
If you're on a Linux machine, you'll also need Node.js and `libpq-dev`.

If you don't already have all the prerequisites installed, there are two ways
you can install them:

- If you're on a Mac, the easiest way to install all the tools is to use
Moncef's [Ruby on Mac] script.

- Install everything manually: [Build tools], [Ruby with RVM], [Postgres], and
[Node.js][node] (Linux only).

[Ruby on Mac]: https://rubyonmac.dev
[Build tools]: https://github.com/codeforamerica/howto/blob/master/Build-Tools.md
[Ruby with RVM]: https://github.com/codeforamerica/howto/blob/master/Ruby.md
[Postgres]: https://github.com/codeforamerica/howto/blob/master/PostgreSQL.md
[node]: https://github.com/codeforamerica/howto/blob/master/Node.js.md

### PostgreSQL Accounts

On Linux, PostgreSQL authentication can be [set to _Trust_](http://www.postgresql.org/docs/9.1/static/auth-methods.html#AUTH-TRUST) [in `pg_hba.conf`](https://wiki.postgresql.org/wiki/Client_Authentication) for ease of installation. Create a user that can create new databases, whose name matches the logged-in user account:

    $ sudo -u postgres createuser --createdb --no-superuser --no-createrole `whoami`

On a Mac with Postgres.app or a Homebrew Postgres installation, this setup is
provided by default.

### Install the dependencies and populate the database with sample data:

    bin/setup

_Note: Installation and preparation can take several minutes to complete!_

### Run the app

Start the app locally on port 8080:

    puma -p 8080

### Verify the app is returning JSON

[http://localhost:8080/api/locations](http://localhost:8080/api/locations)

[http://localhost:8080/api/search?keyword=food](http://localhost:8080/api/search?keyword=food)

We recommend the [JSONView][jsonview] Google Chrome extension for formatting
the JSON response so it is easier to read in the browser.

[jsonview]: https://chrome.google.com/webstore/detail/jsonview/chklaanhfefbnpoihckbnefhakgolnmc

## Set up the environment variables & customizable settings

#### Configure environment variables
Inside the `config` folder, you will find a file named `application.yml`.
Read through it to learn how to customize it to suit your needs.

#### Adjust customizable settings
Inside the `config` folder, you will also find a file called `settings.yml`.
In that file, there are many settings you can, and should, customize.
Read through the documentation to learn how you can customize the app to suit
your needs.

To customize the text the appears throughout the website
(such as error messages, titles, labels, branding), edit `config/locales/en.yml`.
You can also translate the text by copying and pasting the contents of `en.yml`
into a new locale for your language. Find out how in the
[Rails Internationalization Guide](http://guides.rubyonrails.org/i18n.html).

## Uploading and validating your own data

- [Prepare your data][prepare] in a format compatible with Ohana API.

- Place your CSV files in the `data` folder.

- From the command line, run `script/reset` to reset the database.

- Run `script/import` to import your CSV files, but first read the notes below.

If your data doesn't already include a taxonomy, and if you want to use the Open
Eligibility taxonomy, you can create the categories with this command:
```
bin/rake create_categories
```

If your Location entries don't already include a latitude and longitude, the
script will geocode them for you, but this can cause the script to fail with
`Geocoder::OverQueryLimitError`. If you get that error, set a sleep time to
slow down the script:
```
script/import 0.2
```

Alternatively, cache requests and/or use a different geocoding service that
allows more requests per second. See the [geocoding configuration][geocode]
section in the Wiki for more details.

If any entries contain invalid data, the script will output the CSV row
containing the error(s):
```
Importing your organizations...
Line 2: Organization name can't be blank.
```

Open the CSV file containing the error, fix it, save it to the `data` folder,
then run `script/import`. Repeat until your data is error-free.

[prepare]:https://github.com/codeforamerica/ohana-api/wiki/Populating-the-Postgres-database-from-the-Human-Services-Data-Specification-%28HSDS%29-compliant-CSV-files
[geocode]: https://github.com/codeforamerica/ohana-api/wiki/Customizing-the-geocoding-configuration

### Export the database

Once your data is clean, it's a good idea to save a copy of it to make it easy
and much faster to import, whether on your local machine, or on Heroku.
Run this command to export the database:

```
script/export_prod_db
```
This will create a filed called `ohana_api_production.dump` in the data folder.
This will also automatically remove all test users and admins before the export.

### Import the database locally

To restore your local database from your clean data:
```
script/restore_prod_db
```

### User and Admin authentication (for the developer portal and admin interface)

To access the developer portal, visit [http://localhost:8080/](http://localhost:8080/).

To access the admin interface, visit [http://localhost:8080/admin/](http://localhost:8080/admin/).

The app automatically sets up users and admins you can sign in with.
Their username and password are stored in [db/seeds.rb][seeds].

If you deleted these test users and admins (by running `script/export_prod_db`
for example), you can restore them by running `script/users`.

[seeds]: https://github.com/codeforamerica/ohana-api/blob/master/db/seeds.rb

The third admin in the seeds file is automatically set as a Super Admin. If you
would like to set additional admins as super admins, you will need to do it
manually for security reasons.

#### Setting an admin as a Super Admin

##### Locally:

    psql ohana_api_development
    UPDATE "admins" SET super_admin = true WHERE email = 'masteradmin@ohanapi.org';
    \q

Replace `masteradmin@ohanapi.org` in the command above with the email of the
admin you want to set as a super admin.

##### On Heroku:
Follow the same steps above, but replace `psql ohana_api_development` with
`heroku pg:psql -a your-heroku-app-name`.
