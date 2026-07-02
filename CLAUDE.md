# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Projekt
Einfache Hello World Web-App (`index.html`) als Testvehikel für die Deployment-Pipeline. Kein Build-Schritt, kein Package-Manager — nur statisches HTML.

## Git-Remotes
Dieses Repo hat zwei Push-Ziele, ein fetch-Ziel:

```
fetch:  ssh://admin@192.168.178.23/share/MD0_DATA/dataGit/helloworld.git  (NAS lokal)
push:   ssh://admin@192.168.178.23/share/MD0_DATA/dataGit/helloworld.git  (NAS lokal)
push:   https://github.com/guanguido/helloworld.git                        (GitHub)
```

Ein `git push` schreibt automatisch auf beide Ziele.

## NAS-Besonderheit
Das NAS (Synology) hat `git-upload-pack` nicht im Standard-PATH. Der Pfad ist in `.git/config` hinterlegt:
```
[remote "origin"]
    uploadpack = /opt/bin/git-upload-pack
```
Dadurch funktioniert `git ls-remote origin master` ohne den `--upload-pack`-Parameter. Der Symlink `/usr/bin/git-upload-pack` auf dem NAS wird bei Updates automatisch gelöscht — nicht erneut anlegen, Config-Eintrag ist die robustere Lösung.

## Stand prüfen (alle drei Repos)
```bash
git rev-parse master
git ls-remote https://github.com/guanguido/helloworld.git master
git ls-remote origin master
```

## Deployment

Pipeline: `.github/workflows/deploy-strato.yml` — deployt via `rsync` + SSH auf Strato. Nur `index.html` wird übertragen. GitHub Secrets: `STRATO_SSH_KEY`, `STRATO_HOST`, `STRATO_USER`.

| Branch    | Umgebung   | Strato-Pfad                                                                  | URL                                    |
|-----------|------------|------------------------------------------------------------------------------|----------------------------------------|
| `master`  | production | `/mnt/web410/c1/52/59703252/htdocs/bullilab.com/helloworld/production/`     | http://helloworld.bullilab.com         |
| `develop` | staging    | `/mnt/web410/c1/52/59703252/htdocs/bullilab.com/helloworld/staging/`        | http://helloworld-staging.bullilab.com |

HTTPS ist noch nicht eingerichtet.

## Arbeitsablauf

Änderungen immer auf `develop` beginnen → auf Staging testen → erst dann in `master` mergen:

```bash
# Änderung entwickeln und testen
git checkout develop
# ... Änderungen machen, committen, pushen → prüfen auf http://helloworld-staging.bullilab.com

# Nach erfolgreichem Test: nach Production
git checkout master
git merge develop
git push
```

## Leitplanken
- Schreibweise durchgängig englisch: `production` / `staging`
