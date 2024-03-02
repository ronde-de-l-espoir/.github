# La Ronde de l'Espoir !


In 2023, we developed a series of websites and utilities for the Ronde de l'Espoir :


| Name | Purpose | 2023 URL |
|---|---|---|
| Main | showcase | [ronde-de-l-espoir.fr]() |
| Paiment | donation platform | [paiement.ronde-de-l-espoir.fr]() |
| Inscription | Gala booking platform | [inscription.ronde-de-l-espoir.fr]() |
| App | WebView-based app | NaN |
| app-www | Website displayed in app | [app-www.ronde-de-l-espoir.fr]() |

Each of these repos will be detailed and explained :
* generally here, in documentation
* with code comments


## General information

### Hosting

In 2023, we were hosted by [o2switch.fr](), for 85€.

Despite the price, I think you should continue using o2switch because :
* the uptime is great
* they have basically a unique price, but with it you get everything (unlimited storage, unlimited email accounts...)
* the support is effective
* the interface used is cPanel : it is't hugely customizable, it looks old, but it does allow you to chnage quite a lot of settings

Cons :
* pricey
* it is shared hosting, so you are locked in your user `/home` folder. This means that you cannot edit the Apache or PHP configuration files (I'd recommend taking a VPS, as it allows for much more flexibility)

### Domain name problems

In the [o2switch.fr]() bundle was included the `ronde-de-l-espoir.fr` domain name (and subdomains domains).
But when the subscription ended in november 2023, the domain name was available, so somebody decided to use it for a... ahem... slightly different purpose.

I'm writing this in 2024, hopefully I will be able to retrieve the name in a year, when their subscription ends...

Otherwise, you will need to choose a different domain, such as `la-ronde-de-l-espoir.fr` or so...
But don't make the same mistake as us, keep the domain name between *rondes* ; it will only cost 12€ a year, and it's worth it, because every time Google destroys the SEO data if the website is down.

Talking about the website being down, maybe consider using a very cheap server, even a self-hosted one on a Raspberry Pi, just to keep the site running for the SEO...

### Git Usage

We have been using Git since the very beginning of the project.

At first, the idea was to have Git on the server, and communicate directly with it.
But we envied the ease of the GitHub GUI... so we used GitHub !

The system architecture is the following :

Changes aren't allowed in the `main` branch, so devlopers must create another branch to code. Once they finish, they must open a Pull Request (PR).
This allows for control on the standard `git merge` procedure.

Before being merged, the Pull Request goes through some checks but most importantly requires approval of other members of the organisation. The number of people required can be set in the repository settings where you can completely disable *branch protection*. This functionnality is only available in public repos.

After a few Pull Requests, we group them in a release.
Please follow the semantic guidelines for naming the versions.

The changes are sent over to the server via GitHub workflows/actions (other advantage of using GitHub) :
* main.yml : this sends the changes in the `main` branch to the server in the `dev.` directory
* release.yml : this sends the changes to the public URL at release creation.

These files are always located in the `./.github/workflows` directory of the project.

General `main.yml` (please remove the comments to reuse these scripts):

```yml
on: push
name: 🚀 Deploy website on push
jobs:
  web-deploy:
    name: Deploy
    runs-on: ubuntu-latest
    steps:
    - name: 🚚 Get latest code of main
      uses: actions/checkout@v3
      with :
        ref: main // gets the filed of the main branch
    - name: 📂 Sync files
      uses: SamKirkland/FTP-Deploy-Action@4.3.3         //this is the subaction we use to send the files via FTP
      with:
        server: <domain>
        username: ${{ secrets.username }}
        password: ${{ secrets.password }}
        server-dir: <path to the folder where the files must be saved>
```

General `release.yml` (please remove the comments to reuse these scripts) :

```yml
on: 
  release:
    types: [published]

jobs:
  release-latest-tag:
    name: Release Latest Tag
    runs-on: ubuntu-latest
    steps:
    - name: 🚚 Get latest release
      uses: actions/checkout@v3
      with :
        ref: ${{ env.GITHUB_SHA }}
    - name: 📂 Sync files
      uses: SamKirkland/FTP-Deploy-Action@4.3.3
      with:
        server: <domain>
        username: ${{ secrets.username }}
        password: ${{ secrets.password }}
        server-dir: <path to the folder where the files must be saved>
```


The `server-dir` must have a trailing slash and start with a slash (apparently the first slash means "at the root of the FTP root" and not "at the server's root").

For the secrets :
* for public repositories : you can set the secret in the organization settings
* for private repositories : you must set the secret individually, in the repository's settings
