# ansible-notes-training


Ansible est un outil de CM = Configuration Management

objectif de l'industrialisation <=> automatisation 
- déployer et installer les appli sans intervention humaine

Ansible s'est démarqué ces dernières années devant des outils existants depuis longtemps tels que Puppet, Chef


Faciliter la mise à jour de l'infra, sa maintenabilité, et l'intégrer dans les pipelines devops

en résumé Ansible peut servir à 

=> automatiser tout ce qui est fait d'habitude et ce de façon intelligente
=> mettre en place une/des procédure(s) pour permettre aux admins de masteriser une nouvelle machine et automatiser le déployer une nouvelle appli
=> automatiser la phase de production de l'artéfact client i.e. la release client



Culture devops  => déphasage entre dev et ops (mise en prod)
				=> on a des bugs qui font que les mises en prod font plusieurs mois
				=> l'agilité qui est déjà en place pour le dev est aussi mise en place pour l'ops 
				=> l'agilité dans l'ops donne naissance au devops (CI + CD)
				=> les délais de mise en prod sont énormement réduits et de l'ordre de quelques jours
				

Ansible permet de provisionner (sur l'infra on prem ou sur cloud)
 et configurer et déployer (l'infra et l'appli)
 
 Ansible n'a pas besoin d'agent, 
 On a un serveur de déploiement sur lequel Ansible est installé
 et il faut juste installer python sur le poste client
 
 Ansible est donc sécurisé puisqu'on ne peut pas corrompre l'agent (puisque pas d'agent), de plus pour se connecter aux machines distantes Ansible passe par OpenSSH
 Ansible est écrit en python => ça facilite la contribution des devs (puisque le python est très used)
				

*****  				
**************** Installation	**********
******* cette install est à refaire au bureau pour voir si c'est un souci de connexion internet
*****************************************************************************			
Pour installer Ansible sur Windows il faut avoir un noyau linux (ou un fake OS Linux)

* Cluster Ansible = serveur Ansible + client sur lequel on déploie les taches


installer pip 
=> ansible-galaxy install geerlingguy.pip

installer le rôle docker
=> ansible-galaxy install geerlingguy.docker

créer un répertoire (ici "docker") où l'on mettra le playbook de test
=> mkdir && cd docker 
créer le fichier d'inventaire "hosts"
=> vi hosts.ini



********* Commandes AD-HOC  ************

ansible-modules-core

Un module c'est du code python permettant de réaliser une action
Les dev créent des modules python

=> Je comprends que les commandes ad-hoc font référence à des commandes exécutées en une ligne,
elles permettent d'exécuter rapidemnt une action pour un test et constater le résultat

=> on peut obtenir le même résultat via un fichier yaml (mais il faut prendre le temps d'écrire le fichier)


Le module setup permet de découvrir la machine distante: on a beaucoup d'informations de la machine distante
on va pouvoir par exemple lancer des actions selon l'environnement de la machine distante, 
ex. selon son OS, si c'est un update qu'on veut faire, 
si c'est du CentOS on doit faire un yum, si c'est un debian il faut faire un apt, ...

on peut donc avoir le type de bios, la distribution, la taille du disque,

********** TP-2 : [correction] code utilisé ***************************************

=> après avoir créé les machines sur l'espace de labs, 
on applique la bonne pratique de passer sur un compte avec moins de privilèges que le root(dans notre cas, le compte admin)
=> sudo su admin -
=> cd => permet de passer dans le répertoire admin
=> pwd permet de s'en rendre compte (on obtient /home/admin/)



cat hosts
10.0.0.4 ansible_user=admin ansible_password=admin ansible_ssh_common_args='-o StrictHostKeyChecking=no'
10.0.0.5 ansible_user=admin ansible_password=admin ansible_ssh_common_args='-o StrictHostKeyChecking=no' 

ping command
ansible -i hosts all -m ping 

create file command
ansible -i hosts all -m copy -a "dest=/home/admin/toto.txt content='bonjour eazytraining'" 

setup command
ansible -i hosts all -m setup


NOTA: 
La commande => ansible_ssh_common_args='-o StrictHostKeyChecking=no'
permet de rassurer la machine distante en lui disant de faire confiance à la machine qui veut se connaitre
c'est équivalent à faire "yes" quand on tente une première connexion ssh avec paire de clés

C'est très efficace quand on a un grand nombre de machines à connecter
car ça évite de faire "yes" pour chaque machine individuellement

*************************************************************************************


-i renseigne le fichier d'inventaire
all indique qu'il s'agit de toutes les machines de l'inventaire
-m indique le module que je veux utiliser
-a indique les arguments (paramètres) de la commande
dest => indique le chemin du fichier à créer


***** fichier yaml *****

Dès qu'on a plusieurs serveurs et plusieurs commandes, on ne fait plus de ad-hoc 
on passe par un fichier YAML
=> Déclaration:
- une variable commence par une lettre ou un underscore, jamais par un chiffre
- après la variable on met un espace
- une liste peut se déclarer par "nom de liste" suivi de "deux points" puis on passe à la ligne
et on déclare les elts constitutifs de la liste, précédés de tirets de 6
- l'indentation n'est pas imposée mais dans plusieurs documents ce sera deux ou 4 espaces
- structures clé/valeur => permet de décrire un objet dans les détails 
- on peut définir l'inventaire en YAML depuis la version 2.4 de Ansible (Avant on avait la déclaration dans le fichier INI (i.e. le fichier hosts)
et dans cette notation il n'y a pas de notion d'indentation comme en YAML)


Attention: Chaque fois qu'on travaille avec Ansible, vérifier scrupuleusement le répertoire dans lequel on se trouve pour ne pas abimer tout

**** structure INI **********
On définit des groupes d'hôtes entre crochets

En cas de surcharge c'est la valeur de la variable qui est la plus proche de l'hote qui est considérée

[nom_catégorie]
nom_hote  nom_variable=valeur

ex. => [apache]
		rec-apache-1 apache_url=rec.wiki.localdomain
		
		et en YAML
		apache:
			hosts:
				rec-apache-1:
					apache_url: "rec.wiki.localdomain"

** Si on veut définir des variables pour un groupe d'hotes
[nom_groupe_d_hotes:vars]
nom_variable=valeur

ex. => [mysql:vars]
		mysql_user_password=MyPassWord!
		
		et en YAML
		
		mysql:
			vars:
				mysql_user_password: "MyPassWord!"


nota perso => en INI quand on fait var=value 
en yaml ça devient =>           var: "value"


* portée des variables: au fur et à mesure qu'on descend, on écrase les valeurs si elles sont définies dans la couche plus basse
c'est comme dans le code, on a des variables globales et elles sont écrasées par des variables locales quand il y en a

en d'autres termes, les host_vars surchargent les group_vars

=> (variables de) groupe dans le fichier host => 
								group_vars =>

=> (variables) machine au niveau du fichier host, ensuite dans host_vars

=> (variables) dans un fichier YAML

=> (variables) directement passées à Ansible 


*********** TP-5 ***********
Objectif => déployer un serveur web

become: true => permet à Ansible de faire une élevation de privilèges => ex. quand on a besoin d'être sudo pour faire une installation
on peut mettre déclarer le "become" de façon globale ou bien à l'intérieur d'une tache spécifique
si l'user a déjà beaucoup de droits (et a besoin d'un petit nombre de droits supplémentaires juste pour la tache courante)

# différence entre become et become_ask_pass

become_ask_pass permet de prompter pour entrer le mot de passe de façon à passer en super utilisateur

become tout seul donne le droit de passer en super utilisateur, mais à condition que l'utilisateur ait les privilèges
sinon il faut le prompter pour qu'il entre le mot de passe avant de devenir super-utilisateur et là on a besoin de become_ask_pass

become_ask_pass peut être défini dans le fichier de configuration ou encore dans le playbook 
(ou perso. dans le fichier des hôtes (du groupe) (par var: become_ask_pass: ...))


Un playbook est constitué de grands blocs (que Dirane appelle des 'plays') qui définissent le opérations à exécuter à son lancement.

Les "plays" contiennent des tasks 
Un play a aussi des pre_tasks qui décrivent les opérations à faire avant les tasks, ex. => install les EPEL release

Les tasks c'est des listes, donc elles commencent par des tirets avec le champ "name" qui nomme la tache, 
puis les autres "attributs" d'une tache tels que le module à installer;
quand c'est un module, on précise via le when, la condition d'exécution du module 

En dehors de tasks et pre_tasks, le play peut avoir des vars, une clause become, 
des hosts (qui sont soit une liste exhaustive d'hôtes, soit des groupes d'hôtes)

ansible-lint [nom_du_fichier_yaml] => pour la vérification de syntaxe

ansible-playbook -i [hosts.yml] [--ask-vault-pass] [-vvv][fichier_playbook_yml]


Le fichier ansible.cfg se trouve à plusieurs endroits

dans le rep personnel, 
/etc/ansible

mais aussi dans le répertoire de nos projets

le fichier ansible.cfg permet de définir quelques paramètres que l'on peut définir sous forme de variables dans notre inventaire
Il permet aussi de variabiliser certaines choses dans le playbook

*********
(dans le fichier de configuration ansible.cfg qui est par défaut au format INI)

[privilege_escalation]
become_ask_pass = true  
 => par défaut c'est à faux et en l'absence de mot de passe,
une erreur est renvoyée => obligation de définir le mot de passe en clair dans l'inventaire (ansible_sudo_pass)

Ce become_ask_pass est bien utile quand on a un user qui n'a pas beaucoup de droits, 
dans mon cas, vagrant a trop de droits du coup il n'a pas besoin d'élevation de privilèges,
et ce serait superflu de définir un become_ask_pass dans le cas de vagrant

 
 => à true on impose de demander le mot de passe quand il faut des privilèges élevés => ça permet d'enlever le password dans le fichier inventaire 

Dans le tp ça permet de taper le mot de passe "sudo" quand les privilèges hauts sont demandés 


=> option de débogage ansible (pour faire du débogage de base sur ansible)
ajouter des "-v", on peut mettre entre 1 et 5 "v"

dans le TP on en a mis 3 et c'est déjà assez verbeux

ansible-playbook -i [hosts.yml] -vvv [nom_du_fichier_playbook.yml]

*************************

templating loop condition

=> j'apprends que le format jinja est un format de templating


=> loop permet de signifier des boucles
=> when <=> permet de définir des conditions, on peut y adjoindre des "and"
quand on a plusieurs conditions 

=> il y a aussi les tags qui permettent de filtrer davantage dans les conditions 
ex. ne faire telle action que si le tag a telle valeur

*****
ansible_hostname est la variable qui contient le nom de l'hôte sur lequel les fichiers seront déposés
Elle est récupérée lors du "gathering facts" et on peut la voir quand on lance le module "setup"
***********
 Rque à propos du templating, le code ci-après 

 template:
        src: index.html.j2
        dest: /home/vagrant/index.html


permet de lancer le module "template" qui copie le fichier <index.html.j2>
dans le répertoire indiqué par "dest" => ici il s'agit plus d'un renommage de fichier

et les variables entre {} sont remplacées par celles de l'hôte sur lequel est copié le fichier de template

dans notre cas, la variable ansible_hostname prend la valeur du client (qui est dans l'idée, le serveur de prod 
sur lequel on se connecte pour faire des modifications)

NOTA => quand on travaille avec les containers, ne pas hésiter à restart, 
c'est ce qui a validé la modification dans mon cas => sudo docker restart $(sudo docker ps -q)
Puisqu'il n'y a qu'un container qui tourne ça marche => -q pour filtrer l'ID du container dans le résultat

***********
------

************
Questions:
1- différence playbook et rôle
2- comment on faisait avant Ansible? 
Qu'est ce que ça apporte réellement dans la pratique ?

***** génération de la paire de clé en indiquant (par "-t") le protocole used (ici rsa) *********
ssh-keygen -t rsa
=> valider sans rien mettre 
=> la passphrase (on ne l'a pas utilisée dans le tp) nous permet de protéger notre clé des intrusions, 
elle est demandée par exemple quand un programme cherche à hacker notre clé 
(mais en général on ne l'utilise pas dans les tp)

=> pour rappel la clé ssh se trouve dans le dossier (caché) .ssh du répertoire personnel du user 
=> et les clés sont dans les fichiers:
  id_rsa.pub (pour la clé publique) et id_rsa (pour la clé privée)
  
  tout le monde peut copier la clé id_rsa.pub mais il n'y a que le propriétaire (le user) qui a les droits
  sur sa clé privée
  
  par défaut ansible va chercher exactement id_rsa.pub dans le dossier .ssh 
  du coup si on modifie le nom et/ou l'emplacement, il faudra dire à ansible où chercher 
  

***** pour copier sa clé publique sur une machine client ************
on peut passer par ssh en utilisant la commande

ssh-copy-id user@IP    // ici "user" a les droits pour se connecter en SSH sur la machine distante

une fois la clé copiée, les prochaines connexions n'auront plus besoin de mention du login_user, 
on pourra simplement faire => ssh @IP 


autre NOTA: 
La clé publique est copiée sur la machine distante dans le répertoire "authorized_keys"

=> on peut forcer l'utilisation de la connexion par ssh 
=> pour ça il faut supprimer/commenter la déclaration du mot de passe dans le fichier "group_vars/prod.yml"


**** RETENIR SUR LES PAIRES DE CLES ET SSH ********
Sans les paires de clés, quand on fait une connexion ssh on renseigne le login et le password
Une fois la/les clé(s) publique(s) copiées sur la machine distante (par exemple via ssh-copy-id user@IP  => puis "yes", "yes"...)
les futures connexions ssh ne demanderont plus le mot de passe, et même le login
Il suffira d'avoir l'adresse IP, on fera: ssh @IP et la connexion se fera

=> l'objectif est d'avoir une sécurité à un niveau + élevé car avoir recours aux password maintient le risque
de se voir piquer son password

******  VAULT   *************
Vault est un utilitaire qui permet d'encypter des fichier avec ansible
ex. => on encrypte le fichier qui contient des mots de passe 

Quand on encrypte, on utilise une clé (Vault Key) et au moment de déchiffrer on ré-utilise la clé


la commande pour encrypter un fichier est:
=> ansible-vault encrypt [chemin_vers_fichier]
ex. => ansible-vault encrypt files/secrets/credentials.yml

(NOTA: de même pour décrypter il faut juste remplacer "encrypt" par "decrypt")

le pass vault que j'ai utilisé dans le tp c'est simplement "vault"

=> pour vérifier que le vault s'est bien passé on fait un "cat [fichier]" et on a bien, 
un fichier crypté (vaulté)

=> Ensuite on supprime le mot de passe écrit en clair (il était dans group_vars/prod.yml)

=> Puis on indique dans le playbook qu'il y a un nouveau fichier de variables à consulter:
le fichier vaulté qui contient les credentials (files/secrets/credentials.yml)
d'où la syntaxe:
 vars_files:
    - files/secrets/credentials.yml
	
	
=> puis on lance le playbook en indiquant un nouveau mot-clé: "ask-vault-pass" qui invite à 
entrer le mot de passe vault avant le lancement du playbook

Ce qui donne maintenant:

ansible-playbook -i [hosts.yml] [--ask-vault-pass] [-vvv][fichier_playbook_yml]

(-vvv => c'est le mode verbeux pour avoir des infos sur l'exécution courante => plus on a de v plus on a des détauks)

=> petite astuce pour permettre de retrouver l'endroit où a été définie la variable "ansible_password" (bien qu'encryptée)
de façon à la retrouver avec un "grep -r <ansible_password> <path_où_chercher>"   (par défaut <path_où_chercher> est implicite et c'est le répertoire courant)

dans le fichier vault on encrypte une variable intermédiaire (contenant le pass en clair) nommée par exemple: "vault_ansible_password"
et dans le fichier de vars (ici dans le fichier group_vars/prod.yml) on définit en clair une variable par une autre,
ca donne par exemple: ansible_password: "{{ vault_ansible_password }}"


=> En refaisait la commande=> grep -r "ansible_password" 
La recherche est fructueuse

=> NOTA: le fichier vaulté, même décrypté est en RAM de la machine distante, 
DOnc un attaquant ne peut y accéder

**********************************************************
installation des roles à partir du fichier requirements.yml

Se mettre dans le répertoire qui contient le fichier requirements 
avant de faire la commande d'extraction
=> ansible-galaxy install -r roles/requirements.yml

la commande ci-dessus permet d'extraire tous les roles du fichier requirements.yml
et donc d'avoir les noms de ses roles (si on ne les connait pas et qu'on veut y recourir)


les roles sont un moyen de rendre évolutif les playbook 
et les templatiser (les rendre dynamiques) pour qu'ils soient exploités par d'autres personnes
sans qu'il n y ait d'information sensible dans le playbook

La communauté propose des rôles qu'on peut trouver dans la galaxie ansible

Chaque role est lié à un repo github

pour interagir avec la galaxie ansible on utilise la commande => "ansible-galaxy"

la puissance du role c'est vraiment 

En entreprise on peut avoir des galaxies propres (à l'entreprise en effet) 
une galaxie c'est juste un gestionnaire d'archives, ça peut être un nexus, artifactory,

Le role peut être récupéré en local, sur un repo git, n'importe où,  


*************************************
docker-py permet d'installer la version python de docker 


******* lancement d'un playbook *********************

ansible-playbook -i [hosts.yml] [--ask-vault-pass] [-vvv][fichier_playbook_yml]


************* dernier TP *********
dans le dernier TP j'ai eu bcp de problèmes avec docker
j'ai dû installer docker sur le client
et docker-compose dans un répertoire particulier (/usr/local/bin/)

j'ai dû appliquer les 2 commandes suivantes:
sudo curl -SL https://github.com/docker/compose/releases/download/v2.28.1/docker-compose-linux-x86_64 -o /usr/local/bin/docker-compose

sudo chmod +x /usr/local/bin/docker-compose


***** Pour supprimer un role installé   **********

ansible-galaxy role remove [nom_du_role]

********* gathering-facts ******
permet de récupérer les informations sur l'hôte distant grâce au module setup notamment.
Grâce à lui on récupère par exemple la valeur de la variable ansible_hostname si elle a été utilisée dans nos playbook


**************************


































