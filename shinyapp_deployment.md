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
3) `cd [nom_de_l_application]`
4) `sftp DNUM@prod18-ars-diamant.integra.fr`
5) `get incoming/shinyapp_to_deploy/[nom_de_l_archive].zip`
6) `quit`
7) `unzip [nom_de_l_archive].zip`
8) `rm [nom_de_l_archive].zip`
9) `touch Dockerfile`

***
Exemple de Dockerfile : 

\# Base R Shiny image
FROM rocker/shiny:4.3

\# Make a directory in the container
RUN mkdir /home/shiny-app

RUN apt update && apt install -y libssl-dev libudunits2-dev libxml2-dev librsvg2-dev libpq-dev libgdal-dev && apt autoclean

\# Install R dependencies
RUN R -e "install.packages(c('dplyr', 'shinydashboard', 'leaflet', 'leaflet.extras', 'htmlwidgets', 'sf', 'stringr', 'shinycssloaders',   'janitor', 'DT', 'shinyjs', 'shinyWidgets', 'openxlsx', 'here'), repos='https://cloud.r-project.org/')"
#RUN R -e "install.packages(c('plotly'), repos='http://cran.rstudio.com/', dependencies=TRUE)"

\# Copy the Shiny app code
COPY . /home/shiny-app/

\# Expose the application port
EXPOSE 3838

\# Run the R Shiny app
CMD Rscript /home/shiny-app/app.R

***

10) `sudo docker build -t [nom_de_l_application] .`
12) `vim /etc/shinyproxy/application.yml`
13) Ajouter l'application :

***

Exemple :

  - id: tru_app
    display-name: Shiny App TRU
    description: A sample Shiny application du turfu au mois d'aout
    container-image: tru_app
    container-cmd: ["R", "-e","shiny::runApp('/home/shiny-app', host = '0.0.0.0',port = 3838)"]
    access-groups: TRU_APP
    access-users: hassan.hireche@cegedim.com

***

13) `sudo systemctl restart shinyproxy`
 
