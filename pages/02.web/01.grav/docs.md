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
sudo bin/grav clear-cache --all
sudo chown -R www-data:www-data /var/www/cylindric.net/html/
```
