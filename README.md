# Résumé Expert TP4 : Processus & Services

[cite_start]Voici un résumé expert de votre TP4[cite: 1], axé sur les concepts clés que vous devez maîtriser pour votre QCM.

## 🎯 1. Concepts Fondamentaux : Processus, Démons et Services

Ce qu'il faut retenir :

* **Application :** Le programme que vous lancez (ex: Firefox, un script shell).
* **Processus :** Une instance d'une application en cours d'exécution. Chaque processus a un identifiant unique (PID).
* [cite_start]**Démon (Daemon) :** Un processus qui tourne en arrière-plan, sans interface utilisateur, pour effectuer des tâches système (ex: serveur web, service d'impression)[cite: 6].
* **Service :** Un terme (principalement `systemd`) pour gérer un démon ou une tâche. Le service *contrôle* le processus du démon.

---
Un service = un programme qui tourne en arrière-plan et que systemd gère pour toi.
## ⚙️ 2. `systemd` : Le Cœur de la Gestion

[cite_start]`systemd` est le **premier processus** démarré par le noyau Linux (son PID est 1)[cite: 28]. [cite_start]Son rôle principal est d'initialiser le reste du système et de **gérer les services**[cite: 8].

### Commandes `systemctl` à connaître

* [cite_start]`systemctl status <service>` : Affiche l'état détaillé d'un service[cite: 14, 42, 72].
* [cite_start]`systemctl start <service>` : Démarre un service[cite: 70].
* [cite_start]`systemctl stop <service>` : Arrête un service[cite: 41].
* [cite_start]`systemctl enable <service>` : Active le service pour qu'il se lance au démarrage [cite: 69] [cite_start](en créant un lien symbolique dans un répertoire `.wants` [cite: 19, 22]).
* `systemctl disable <service>` : Désactive le service au démarrage.
* [cite_start]`systemctl get-default` : Affiche la "cible" (target) par défaut[cite: 15, 102], l'équivalent `systemd` du *runlevel*.

### Fichiers de configuration des services (`.service`)

[cite_start]Vous avez créé deux services (`mondemon` et `mondemon2`)[cite: 58, 80], ce qui met en lumière des directives cruciales pour un QCM :

* **Emplacements :**
    * [cite_start]`/lib/systemd/system` : Fichiers fournis par les paquets logiciels[cite: 20].
    * [cite_start]`/etc/systemd/system` : Fichiers créés par l'administrateur (vous !) ou modifiés[cite: 18]. [cite_start]C'est ici que vous avez mis vos fichiers `.service`[cite: 58, 80].
* **Directives Clés :**
    * [cite_start]`[Unit]`: Contient la `Description` du service[cite: 60, 82].
    * [cite_start]`[Service]`: Définit le comportement du service[cite: 61, 83].
        * `Type=oneshot` : Le service exécute un script une seule fois et termine. [cite_start]Il est considéré comme "actif" s'il est combiné avec `RemainAfterExit=yes` (comme dans `mondemon` v1)[cite: 62, 63].
        * `Type=simple` : Le processus principal (`ExecStart`) est le service lui-même. [cite_start]S'il se termine, `systemd` considère que le service s'est arrêté (comme dans `mondemon` v2)[cite: 84, 85].
        * [cite_start]`Restart=always` : Si le service s'arrête (comme le script `mondemon run` qui s'exécute et se termine) [cite: 53][cite_start], `systemd` le redémarrera automatiquement (c'est ce qui remplissait votre log)[cite: 88, 95].
    * `[Install]`:
        * [cite_start]`WantedBy=multi-user.target` : Indique à `systemctl enable` d'associer ce service au démarrage multi-utilisateur normal[cite: 65, 90].

### Analyse du démarrage (Bonus)

* [cite_start]`systemd-analyze time` : Montre le temps total du démarrage[cite: 129].
* [cite_start]`systemd-analyze blame` : Liste les services, du plus lent au plus rapide, au démarrage[cite: 130].

---

## 📊 3. Gérer et Analyser les Processus

### [cite_start]Le système de fichiers `/proc` [cite: 103]

C'est un système de fichiers **virtuel** qui contient des informations sur le système et les processus en temps réel.

* [cite_start]**Répertoires numérotés** (`/proc/1`, `/proc/1234`...) : Correspondant au **PID** de chaque processus actif[cite: 104].
* [cite_start]`/proc/[PID]/status` : Donne l'état détaillé d'un processus (ex: Sleeping, Running, Zombie)[cite: 105, 106].
* **Fichiers importants :**
    * [cite_start]`/proc/cpuinfo` : Infos sur le processeur[cite: 110].
    * [cite_start]`/proc/modules` : Modules du noyau chargés[cite: 110].
    * [cite_start]`/proc/sys` : Permet de lire/modifier les paramètres du noyau à la volée[cite: 107].
* **Danger :** `kcore`. N'affichez **jamais** ce fichier (`cat /proc/kcore`). Il représente l'intégralité de la **mémoire vive (RAM)** de votre machine. [cite_start]Tenter de l'afficher va saturer votre terminal et peut geler votre système[cite: 111].

### Commandes de processus

* [cite_start]`ps` : Affiche un "snapshot" des processus[cite: 26].
    * [cite_start]`ps -e` : Montre **tous** les processus[cite: 27].
    * [cite_start]`ps -aux` : Format BSD, montre tous les processus (même ceux sans tty) avec le nom d'utilisateur[cite: 27].
    * [cite_start]`ps -jfx` : Affiche l'arborescence des processus (qui a lancé quoi)[cite: 27].
* [cite_start]`top` / `htop` : Affiche les processus en **temps réel**, triés par défaut par usage CPU[cite: 113, 117].
* [cite_start]`pstree` : Visualise l'arborescence des processus[cite: 133].
* [cite_start]`pgrep` / `pidof` : Trouve le PID d'un processus par son nom[cite: 133].

### Signaux et Contrôle de Tâches

* [cite_start]`sleep 9999 &` : Lance la commande `sleep` en **arrière-plan** (job)[cite: 119].
* [cite_start]`kill <PID>` : Envoie un signal à un processus[cite: 37].
    * [cite_start]**SIGSTOP** (`kill -STOP <PID>`) : Met le processus en **pause** (il est "stoppé")[cite: 121].
    * [cite_start]**SIGCONT** (`kill -CONT <PID>`) : **Reprend** un processus stoppé[cite: 122].
    * **SIGTERM** (défaut, `kill <PID>`) : Demande poliment au processus de s'arrêter.
    * **SIGKILL** (`kill -9 <PID>`) : Tue le processus de force (à éviter sauf si nécessaire).
* `nice` : Modifie la **priorité** d'un processus. Une valeur *positive* (`+10`) le rend *moins* prioritaire ("plus gentil"). [cite_start]Une valeur *négative* (`-10`) le rend *plus* prioritaire (nécessite `sudo`)[cite: 147, 148].

---

## 🖥️ 4. TTY et Services Associés

* [cite_start]Une **TTY** est une console (un terminal)[cite: 29].
* [cite_start]Vous avez ouvert `tty2` (avec Alt+F2)[cite: 30, 32].
* [cite_start]Le service qui gère la connexion sur `tty2` est `getty@tty2.service`[cite: 39].
* [cite_start]Lorsque vous avez tué les processus de `tty2` avec `kill` [cite: 37][cite_start], `systemd` (via ce service) les a probablement relancés (comportement normal d'un service de type `getty`)[cite: 38].
* [cite_start]En revanche, en arrêtant le *service* (`systemctl stop getty@tty2`) [cite: 41][cite_start], vous avez empêché toute nouvelle connexion sur `tty2`[cite: 44].

---

## 📎 5. Concepts Bonus (MIME et Matériel)

* **MIME Types :** Le système utilise les types MIME pour savoir quelle application ouvrir pour quel fichier.
    * [cite_start]`xdg-mime query filetype <fichier>` : Donne le type MIME du fichier (ex: `text/plain`)[cite: 164, 168].
    * [cite_start]`xdg-mime query default <type>` : Donne l'application par défaut pour ce type MIME (ex: `gedit.desktop`)[cite: 170].
    * [cite_start]`/usr/share/applications` : Contient les fichiers `.desktop` (les "lanceurs" d'applications)[cite: 169].
* **Analyse Matériel :**
    * [cite_start]`lspci` : Liste les périphériques PCI[cite: 175].
    * [cite_start]`lsusb` : Liste les périphériques USB[cite: 175].
    * [cite_start]`lscpu` : Affiche les informations du processeur[cite: 175].
    * [cite_start]`lsmod` : Liste les modules du noyau actuellement chargés[cite: 175].

Bonnes révisions pour votre QCM !
