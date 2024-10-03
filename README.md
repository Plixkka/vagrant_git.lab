Pré-requis
Vagrant et VirtualBox doivent être installés sur votre machine hôte.
Une connexion Internet pour télécharger les paquets nécessaires.
Étapes d'installation

1. Créer un répertoire de projet
Commencez par créer un répertoire où vous allez configurer votre projet Vagrant.

bash
Copier le code
```mkdir gitlab-vagrant
cd gitlab-vagrant
```
2. Initialiser Vagrant
Dans le répertoire créé, exécutez la commande suivante pour initialiser un fichier Vagrantfile.

bash
Copier le code
``` vagrant init ubuntu/bionic64
```
Cela génère un fichier Vagrantfile basique pour Ubuntu 18.04 LTS (Bionic Beaver).

3. Configurer le fichier Vagrantfile
Modifiez le fichier Vagrantfile pour installer GitLab CE et configurer la VM avec les paramètres nécessaires. Voici une configuration détaillée :

ruby
Copier le code

```Vagrant.configure("2") do |config|
  # Utiliser l'image de base Ubuntu 18.04
  config.vm.box = "ubuntu/bionic64"

  # Configurer une adresse IP statique
  config.vm.network "private_network", ip: "192.168.56.10"

  # Rediriger le port 80 de la VM vers le port 8080 de l'hôte
  config.vm.network "forwarded_port", guest: 80, host: 8080

  # Provisions pour installer GitLab CE
  config.vm.provision "shell", inline: <<-SHELL
    # Mettre à jour les paquets et installer les dépendances nécessaires
    apt-get update
    apt-get upgrade -y
    apt-get install -y curl openssh-server ca-certificates tzdata perl

    # Ajouter le dépôt GitLab
    curl https://packages.gitlab.com/install/repositories/gitlab/gitlab-ce/script.deb.sh | bash

    # Installer GitLab CE
    apt-get install -y gitlab-ce

    # Configurer GitLab avec une URL externe
    EXTERNAL_URL="http://192.168.56.10" gitlab-ctl reconfigure
  SHELL

  # Configuration des ressources de la VM
  config.vm.provider "virtualbox" do |vb|
    vb.memory = "4096"  # Allouer 4 Go de RAM
    vb.cpus = 2         # Allouer 2 CPUs
  end
end
```


Explication des configurations :

```config.vm.box : Utilise Ubuntu 18.04 LTS comme image de base.
config.vm.network : Définit une IP statique (192.168.56.10) pour la VM et redirige le port 80 de la VM vers le port 8080 de l'hôte.
config.vm.provision "shell" : Un provisionnement shell pour installer GitLab CE.
vb.memory et vb.cpus : Alloue 4 Go de RAM et 2 processeurs à la VM pour assurer des performances suffisantes.
```
4. Démarrer et provisionner la VM
Une fois le fichier Vagrantfile configuré, démarrez la VM et exécutez les provisions pour installer GitLab.

bash
Copier le code
```vagrant up
```
Cela va :

Télécharger et démarrer la VM Ubuntu.
Exécuter les commandes définies dans la section config.vm.provision pour installer GitLab.
5. Accéder à GitLab
Une fois l’installation terminée, vous pouvez accéder à GitLab à l’adresse suivante sur votre navigateur :

http
Copier le code
```http://192.168.56.10
```
Si vous avez configuré la redirection de port, vous pouvez aussi utiliser :

http
Copier le code
```http://localhost:8080
```
Note : Lors de la première connexion, vous devrez définir un mot de passe pour l'utilisateur root. Celui-ci sera l'administrateur de GitLab.

6. (Optionnel) Configurer GitLab Container Registry
Si vous souhaitez activer et configurer le GitLab Container Registry pour gérer des images Docker, voici comment faire :

Ouvrez le fichier de configuration de GitLab sur la VM :

bash
Copier le code
```sudo nano /etc/gitlab/gitlab.rb
```
Ajoutez ou modifiez les lignes suivantes pour activer le Container Registry sur le port 5050 :

ruby
Copier le code
```registry_external_url 'http://192.168.56.10:5050'
```
Reconfigurez GitLab pour appliquer les modifications :

bash
Copier le code
```sudo gitlab-ctl reconfigure
```
Après la reconfiguration, le GitLab Container Registry sera accessible à l'adresse http://192.168.56.10:5050.

7. Gérer GitLab avec gitlab-ctl
Voici quelques commandes utiles pour gérer l’installation GitLab :

Vérifier le statut des services :

bash
Copier le code
```sudo gitlab-ctl status
```
Redémarrer GitLab :

bash
Copier le code
```sudo gitlab-ctl restart
```
Appliquer les modifications après une modification dans gitlab.rb :

bash
Copier le code
```sudo gitlab-ctl reconfigure
```
8. (Optionnel) Gestion des ports et pare-feu
Si vous rencontrez des problèmes d’accès depuis l’hôte, assurez-vous que les ports nécessaires sont ouverts et que le pare-feu ne bloque pas les connexions.

Vérifiez que le port 8080 est bien redirigé sur votre hôte :

bash
Copier le code
```sudo netstat -tuln | grep 8080
```
Désactivez temporairement le pare-feu pour tester :

bash
Copier le code
```sudo ufw disable
```
9. Arrêter et détruire la VM
Si vous souhaitez arrêter ou détruire la VM :

Pour arrêter la VM (la mettre en pause sans détruire les données) :

bash
Copier le code
```vagrant halt
```
Pour détruire la VM et effacer toutes les données :

bash
Copier le code
```vagrant destroy
```
Conclusion
Vous avez maintenant une installation GitLab CE fonctionnelle sur une machine virtuelle avec Vagrant. Cette configuration peut être modifiée en fonction de vos besoins, et l’utilisation d'une IP statique permet de facilement accéder à GitLab depuis votre machine hôte.

