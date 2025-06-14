Pour ce projet, on m'a chargé de concevoir et mettre en oeuvre un traitement de données dans un environnement Big Data sur le cloud. Une start-up souhaite se faire connaître en mettant à disposition du grand public une application mobile qui permettrait aux utilisateurs de prendre en photo un fruit et d'obtenir des informations sur celui-ci. Il faut donc entraîner un modèle de classification d'images, seulement on s'attend à une augmentation massive du volume de données, donc on me demande d'utiliser PySpark, faire un test en local pour s'assurer que tout fonctionne, puis passer sur le cloud dans un environnement Big Data.

Le notebook a un double rôle : Exécution de la solution en local (partie 3), et exécution de la solution dans le cloud (partie 4). Tout en servant à la fois de guide d'utilisation pour les outils utilisés : Machine virtuelle (j'ai utilisé VMWare Workstation), Spark et tout l'environnement Big Data (dans mon cas AWS). Comme lors d'un projet précédent, je vais faire du Transfer learning avec un CNN, le MobileNetV2 cette fois, car plus faible dimensionnalité du vecteur de sortie et donc plus adapté à une application mobile que le VGG-16. Je n'ai pas grand chose à dire sur le déploiement en local alors je préfère développer sur le déploiement dans le cloud.

J'ai commencé par configurer une authentification multi-facteur et une clé d'accès SSH avec IAM, ensuite j'ai installé et configuré AWS Cli, l'interface en ligne de commande. A partir de celui-ci, j'ai stocké les données sur un bucket S3 avant de m'attaquer au plus gros morceau : l'EMR. Ceux-ci utilisent des frameworks tels qu'Hadoop et Spark, ainsi que des serveurs EC2 avec des applications pré-installées et configurées pour créer et gérer le cluster de calculs distribués. J'ai suivi ces étapes :
- Configuration logiciel, avec des paramètres personnalisés afin d'assurer la persistance des notebooks et du stockage sur S3
- Configuration matérielle, une instance maître et deux instances principales m5.xlarge
- Actions d'amorçage (ou bootstraping), pour installer l'ensemble des packages utiles à l'exécution du notebook sur l'ensemble des machines (et pas uniquement le driver)
- Sécurité, sélection de la paire de clés EC2, de la fonction de service et du profil d'instance

Le cluster est enfin créé, mais pas encore prêt à l'emploi. Il faut maintenant créer le tunnel SSH et proxy :
- Ouverture du port 22 sur lequel écoute le serveur SSH
- Connexion au noeud primaire à l'aide de SSH (dans le terminal)
- Configuration de l'extension FoxyProxy pour emprunter le tunnel SSH

On peut enfin ouvrir et exécuter le notebook depuis JupyterHub, hébergé sur le serveur EMR. En choisissant un kernel PySpark, une session Spark démarre à l'exécution de la première cellule (à partir de la partie 4.10).

Afin de procéder à une réduction de dimensionnalité, je fais une ACP. En temps normal je choisirais k en fonction du pourcentage de variance cumulée expliquée, mais les coûts montent vite sur AWS donc j'ai arbitrairement choisi k=50. Les résultats sont stockés directement sur S3.
