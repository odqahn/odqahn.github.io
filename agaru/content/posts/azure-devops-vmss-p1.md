---
title: "Azure Devops & VMSS - part1"
date: 2021-03-04T20:29:11+01:00
draft: true
---

# Azure DevOps et VMSS agents
Bonne nouvelle, Azure DevOps permets maintenant d'utiliser des scale set pour les agents. Cela va apporter très facilement plus de flexibilités que les self hosted et nous permettre de faire des montages assez sympas.

Dans cette série de post nous allons :
* Voir en quoi un VMSS va nous apporter des choses (use cases)
* Créer un VMSS et le lier à Azure DevOps
* Explorer les différentes options
* Créer une image custo avec Docker pour pouvoir builder nos petits Dockerfiles
* Explorer le git des agents de build de Microsoft et builder l'image des self hosted pour notre VMSS
* Aborder quelques points de sécurité et faire un peu de custo avancée avec ces images

Commençons directement !

## Pourquoi les VMSS, c'est cool pour Azure DevOps?
### Ressources privées
A moins d'être une start up ou d'avoir foncé dans le cloud, votre infra est certainement composée d'un mélange de pleins de choses. On va y retrouver des VM, dans des VNet, des ressources publiques mais aussi des ressources privées. Depuis quelques temps, Azure propose de privatiser pas mal de ressources grâce aux [private endpoint](https://docs.microsoft.com/en-us/azure/private-link/private-endpoint-overview) par exemple.

Ce sont ces ressources privées et isolée d'internet qui nous intéressent. Pour l'instant, contacter celles-ci nécessitaient   de devoir créer un agent self hosted en créant un VM, en installant un agent. Il fallait ensuite monitorer celui-ci, le maintenir etc. 

Grâce au VMSS, on va disposer d'un potentiel élastique pour faire tourner nos pipelines et pouvoir rassembler le meilleur des deux mondes : des images construites dynamiquement dans une infra privée. Un seul pool d'agent qui va pouvoir builder du code sans devoir faire la maintenance de tous les framework, vous offrir la possibilité de réaliser des tests d'intégration directement sur votre non prod, de faire des releases sur des IIS internes,...

### Le cout
A noter aussi, dans un VMSS, vous ne payez que l'uptime de vos VMs (et un peu de stockage, de traffic,... mais c'est surtout l'utime). Grâce au VMSS et à l'intégration dans DevOps, on va pouvoir jouer là-dessus pour ne payer que ce qu'on utilise réellement, et pas pour 10 agents self hosted dispo 24/24 7/7 ou une VM qui doit tourner tout le temps.

Ici, c'est DevOps qui va piloter le scale set et provisionner des agents quand un build est lancé. On pourra lui donner quelques règles comme la possibilité de garder des agents en stand by ou l'idle time avant de tuer une instance par exemple. Nous explorerons les possibilités lors de la configuration.

### Custo!
Les selfs hosted ne sont pas facilement customisable. Bien sûr, vous pouvez ajouter des outils dans vos pipeline via une commande PowerShell ou bash mais cela représente un cout de temps, qui, pour certains outils, peut être assez long. 

Vous aimeriez peut-être que l'agent Ubuntu 20.04 utilise java 8 plutôt que java 11 par défaut? C'est possible (et ça sera d'ailleurs ce qu'on fera dans le dernier post).

## La suite?
Prochain post, création du VMSS et configuration dans DevOps avec une petite application d'exemple!