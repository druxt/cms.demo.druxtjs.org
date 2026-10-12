# Drupal CMS demo

[Drupal CMS](https://www.drupal.org/project/cms) with the Haven site template and its sample content, the backend of the Druxt Drupal CMS demo.

Haven is the non-profit template. Its sample organisation works on climate and the environment, and its landing pages are built in Canvas around projects, blog posts and people. For now Drupal renders the site itself. A Druxt frontend comes later and reads the same content over JSON:API.

## Run it locally

You need PHP 8.3 or later with SQLite, and Composer.

```sh
cd drupal
composer install
.devtools/provision
cd web && php -S 127.0.0.1:8888 .ht.router.php
```

Provision installs a fresh site into `/tmp/cms-demo.sqlite` (set `DB_FILE` to change it). Sign in with `vendor/bin/drush user:login`.

## What provision does

- Installs Drupal CMS from the Haven template, which brings its content.
- Removes Automatic Updates, Project Browser and Package Manager. The codebase is built into an image and never changes from the site.
- Enables JSON:API and the Druxt module, for the frontend to come.
- Builds the search index.

`.devtools/demo-snapshot` keeps the fresh site and `.devtools/demo-reset` puts it back.

## Lagoon

One environment runs Drupal, its database and the daily reset. Each deploy installs the site fresh and snapshots it, and a cron job resets the demo to that snapshot every day. Production answers on api.cms.demo.druxtjs.org, and on cms.demo.druxtjs.org until the frontend takes that domain.

Set `DRUPAL_ACCOUNT_PASS` on the Lagoon project to give the admin account a known password.

## License

MIT
