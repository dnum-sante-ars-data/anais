# Déploiement d'une Shiny app sur la plateforme ANAIS

## Prérequis

* avoir accès aux VM de l'environnement de développement
* avoir accès au SFTP (facultatif, seulement dans le cas où l'application est transmise via France Transfert)

## Transfert de l'application Shiny

Le développeur de l'application Shiny doit fournir le `app.r` ainsi que tous les fichiers de l'application (data, images, etc.) dans une archive `.zip` : 
* via France Transfert s'il n'a pas encore de compte et d'accès au SFTP
* via le SFTP dans le répertoire `ARS_BDD/[nom_de_sa_region]`

Si l'archive a été envoyée via France Transfert, une étape d'upload sur le SFTP est indispensable : 
1) En local, se placer dans le répertoire où l'archive a été téléchargée, par ex `cd Downloads`
2) `sftp DNUM@prod18-ars-diamant.integra.fr`
3) `put incoming/shinyapp_to_deploy/[nom_de_l_archive].zip`
5) `quit`

## Sur anais.app : 

Chaque application Shiny est conteneurisée via Docker et l'accès via Keycloack est configuré via Shiny Proxy. 
* les conteneurs de chaque application sont dans : `/opt/shinyapp`
* le fichier de configuration Shiny proxy est situé dans : `/etc/shinyproxy`

1) `cd /opt/shinyapp`
2) `mkdir [nom_de_l_application]`
3) `sftp DNUM@prod18-ars-diamant.integra.fr`
4) `get incoming/shinyapp_to_deploy/[nom_de_l_archive].zip`
5) `quit`
