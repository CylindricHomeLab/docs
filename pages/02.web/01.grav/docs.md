---
title: 'Grav'
taxonomy:
    category:
        - Web
    tag:
        - grav
        - highlight
---

Various Grav notes.

===

# Requirements

Now needs PHP 7.4.

```bash
sudo su -
apt-get update
apt -y install software-properties-common
add-apt-repository ppa:ondrej/php
apt-get update
apt -y install php7.4
apt-get install -y php7.4-{mbstring,curl,ctype,zip,fpm,dom,gd,xml}
```

Update `/etc/caddy/Caddyfile` to use the `7.4-fpm`.

Plugins being used (`./bin/gpm install <package>`):

* admin
* cookie-consent
* ganalytics
* git-sync
* highlight
* markdown-notices
* mermaid-diagrams
* page-toc
* tagcloud

# Updating Grav
```bash
cd /var/www/cylindric.net/html
bin/gpm self-upgrade
bin/gpm update
chown -R www-data:www-data .
```

# Updating the Highlighter plugin
```bash
git clone https://github.com/isagalaev/highlight.js.git
cd highlight.js
npm install
```
The full list of languages is available [from therepository](https://github.com/isagalaev/highlight.js/tree/master/src/languages).

* Common languages only: `node tools/build.js :common`
* All languages: `node tools/build.js -t node`
* Specific languages: `node tools/build.js -n python ruby`
* Common + other: `node tools/build.js -t webbrowser :common powershell`
* CDN: `node tools/build.js -t cdn -`

Install the new script:

```bash
sudo cp build/highlight.pack.js /var/www/cylindric.net/html/user/plugins/highlight/js/
```

Clear the cache:

```
cd /var/www/cylindric.net/html
sudo bin/grav clearcache --all
sudo chown -R www-data:www-data /var/www/cylindric.net/html/
```
