---
title: "Static website using Hugo"
date: 2020-08-02T19:15:15+02:00
draft: false
---

# Bienvenu chez moi! 

Pour mon premier post, parlons de l'hébergement de ce site.

## Intro
Les outils utilisés sont:
- [Hugo](https://gohugo.io/)
- [Azure Statis Web Apps](https://azure.microsoft.com/fr-fr/services/app-service/static/)
- [Github](https://github.com)
- [VScode](https://code.visualstudio.com/)
- [Markdown](https://daringfireball.net/projects/markdown/)

Hugo permets de générer un site statique sur base de configurations et de fichiers Markdown. L'avantage de ce système est d'avoir besoin de très peu de ressources pour servir le site, il n'y aura que très peu (voir pas) de calcul à faire, autant du côté serveur que du côté client.

Concernant Markdown, il s'agit d'un language balisé plutôt simple à utiliser. Il a pour but de fournir un document facile à lire et à interpréter et transformable en html, pdf,... C'est ce language qui est utilisé traditionnellement dans les repos git pour composer le fichier README.md (et tout autre doc faisant partie du source control).

## Hugo

Hugo est assez simple à installer: https://gohugo.io/getting-started/installing

Une fois en place, il vous suffira de créer un nouveau site à l'aide de votre terminal préféré: `hugo new site newsite`.

Pour l'utilisation basique de Hugo, je vous renvoi au [tuto quick start](https://gohugo.io/getting-started/quick-start/) du projet.

## Azure

Concernant l'hébergement, j'utilise une web app dans Azure spécialement crééee pour du contenu dynamique. Innutile de payer pour le support de framework innutilisé, il n'y aura ici que de l'htlm et un peu de javascript.

Cette technologie est actuellement en preview et gratuite. Il suffit d'avoir un abonnement Azure pour se lancer. 

La configuration de cet artefact est super simple. On pourra lui ajouter un domaine custo (ici agaru.be) en suivant l'assistant. Le certificat https sera automatiquement ajouté.

Une fois en place, vous pouvez déposer des fichiers htlm ou mieux... Automatiser le déploiement!

## Git workflow & deploy

Mon code est hébergé dans github, ce qui va me permettre d'automatiser la publication des nouvelles versions. Contrairement à un site web dynamique comme on à l'habitude d'en croiser (Wordpress, Drupal,...), chaque mise à jour du contenu va m'obliger à publier le site. Et comme chaque action manuelle, cela peut vite devenir redondant, sans parler des erreurs humaines.

Sur les principes d'automatisations, qu'on peut retrouver dans la philosphie devops par exemple, j'ai utilisé workflow github afin de pousser automatiquement mon code dans l'app service.

Pour se faire, après avoir créé mon repo dans github, il me suffit de me connecter à mon compte github pendant la création de mon app service:
![screenshot du portail azure](img/hosting/webappstatic.png)

Après avoir choisi son organisation, le repo et la branche, Azure va automatiquement ajouter un dossier .github contenant votre workflow dans le source control.

Il ne reste maintenant qu'à générer vos pages avec la CLI de hugo (voir le [tuto quick start](https://gohugo.io/getting-started/quick-start/)), commiter et pusher vos changements. Tout sera déployé automatiquement sur votre site quelques minutes plus tard.