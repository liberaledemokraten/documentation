---
title: 'Basic Configuration'
---

Some basic configuration should be set for the LD Cloud. Those will be described in the following.

## Recommendations

### Setting the Maximum Memory Size
The maximum memory size of the PHP configuration should be set to 512 MB.

### PHP Memory Cache
Configuring a memory cache is recommended for better performance. For further information, you may read [this documentation](https://docs.nextcloud.com/server/20/admin_manual/installation/server_tuning.html).

### Manually adding DB Indices
Nextcloud does not add all indices automatically to not keep alot of time during setup. However the admin can trigger the indexation by running `occ db:add-missing-indices`.

### Adding Recommended PHP Modules
Adding the following PHP modules are recommended by Nextcloud:

* bcmath
* gmp
* imagick

## Optional Configurations

### Maintenance Mode
When maintaining the LD Cloud and changing some things, we may want nobody to use the LD Cloud and enter the maintenance mode. To do so, add or modify the following entry in the `config.php`

```
'maintenance' => true,
```

### Remove Advertisement Links
Nextcloud per default advertises to Nextcloud. That is especially visible when sharing files with users that are not logged in. But we can safely disable that. To do so, go to your web root directory `public_html` and add the following line to `config/config.php`:

```
'simpleSignUpLink.shown' => false,
```

### Default Language & Locale
Nextcloud may set the language and locale not the way we'd like. As a German party, it is our primary choice to go for German and Germany as the default value. So we may add the follwoing two lines into the `config.php`

```
'default_language' => 'de',
'default_locale' => 'de',
```

### Default App
The default app is the one app that loads first with each login. Nextcloud defaults to "dashboard", but as we are using the LD Cloud mainly as a file storage, we may want to default to the "files" app as well. To do so, add the following to the `config.php`
```
'defaultapp' => 'files',
```