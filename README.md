# Rapport technique : Mise en place d'un environnement d'instrumentation dynamique avec Frida

Ce document détaille les procédures techniques suivies pour remplir les objectifs de configuration et de validation d'un environnement d'analyse dynamique utilisant Frida sur un hôte Windows 11 ciblant un émulateur Android[cite: 1].

---

## 1. Installation et vérification de la partie cliente (Windows)

L'objectif initial était de configurer l'hôte de contrôle pour exécuter les outils de ligne de commande Frida[cite: 1].

### 1.1 Configuration de l'environnement Python
L'installation a été réalisée sous **Python 3.14**[cite: 1]. Une difficulté majeure a été rencontrée lors de l'appel initial de la commande `pip`, celle-ci n'étant pas reconnue par le système[cite: 1].
*   **Action corrective :** Ajout manuel du répertoire `Scripts` de Python (situé dans `C:\Users\fajri\AppData\Local\Python\pythoncore-3.14-64\Scripts`) aux variables d'environnement (PATH) de Windows[cite: 1].
*   **Vérification :** La commande `frida --version` a été exécutée avec succès pour confirmer la disponibilité des outils[cite: 1].

### 1.2 Installation des outils Frida
Le déploiement des paquets nécessaires a été effectué via le gestionnaire de paquets Python :
```powershell
pip install frida-tools frida
```
La version installée et validée est **Frida 17.9.1**[cite: 1].

---

## 2. Déploiement et exécution de frida-server (Android)

L'agent Frida doit être déployé sur l'appareil cible pour permettre la communication avec l'hôte[cite: 1].

### 2.1 Identification de l'architecture
L'architecture CPU de l'émulateur Android (ID: emulator-5554) a été identifiée comme **x86_64**[cite: 1]. Cette étape est critique pour garantir la compatibilité du binaire[cite: 1].

### 2.2 Transfert et configuration du serveur
Le binaire `frida-server-17.9.1-android-x86_64` a été transféré sur l'appareil dans le répertoire `/data/local/tmp/`, seul répertoire système permettant généralement l'exécution de fichiers binaires par l'utilisateur via ADB[cite: 1].
```powershell
adb push frida-server-17.9.1-android-x86_64 /data/local/tmp/
adb shell chmod 755 /data/local/tmp/frida-server-17.9.1-android-x86_64
```

### 2.3 Exécution et gestion des privilèges
Une erreur de type `Permission denied` liée à la politique SELinux du noyau a été diagnostiquée lors du premier lancement[cite: 1].
*   **Résolution :** Élévation des privilèges du démon ADB via `adb root` et lancement du serveur avec l'option d'écoute réseau `-l 0.0.0.0`[cite: 1].

---

## 3. Établissement de la connexion et injection minimale

Une fois le serveur actif, la validation de la chaîne d'instrumentation a été réalisée par l'injection de scripts JavaScript dans le processus cible[cite: 1].

### 3.1 Vérification de la connectivité
La commande suivante a permis de confirmer que l'hôte Windows pouvait énumérer les applications installées sur l'émulateur :
```powershell
frida-ps -Uai
```

### 3.2 Injection du script de validation (hello.js)
Un script minimal visant à valider l'accès au runtime Java a été injecté dans l'application cible `owasp.mstg.uncrackable1`[cite: 1].
*   **Script :** `Java.perform(function () { console.log("[+] Frida Java.perform OK"); });`[cite: 1].
*   **Résultat :** L'affichage du message `[+] Frida Java.perform OK` dans le terminal a confirmé la capacité de Frida à interagir avec la machine virtuelle Android (ART)[cite: 1].

---

## 4. Diagnostics et résolution des problèmes courants

Le laboratoire a permis de documenter et de résoudre plusieurs obstacles techniques classiques lors d'une installation Frida[cite: 1].

### 4.1 Erreurs de chemin (PATH)
L'absence des exécutables dans le PATH système a été identifiée comme la cause principale de l'échec des commandes `pip` et `frida`[cite: 1]. Une inspection du répertoire `Scripts` a permis de confirmer la présence des fichiers et de corriger la configuration système[cite: 1].

### 4.2 Conflits de signatures (Overloads)
Lors de l'analyse avancée d'une application obfuscée (Uncrackable1), une erreur `specified argument types do not match` a été rencontrée[cite: 1].
*   **Analyse :** La méthode ciblée possédait plusieurs surcharges (overloads) avec des signatures différentes[cite: 1].
*   **Diagnostic :** Frida nécessite une spécification explicite des types d'arguments lors d'une ambiguïté[cite: 1].
*   **Correction :** Utilisation de `.overload('[B', '[B')` pour cibler précisément la méthode utilisant des tableaux d'octets, permettant ainsi l'extraction finale de la clé secrète[cite: 1].

---

## Conclusion
Tous les objectifs du laboratoire ont été atteints[cite: 1]. L'environnement est désormais fonctionnel pour des analyses de sécurité complexes, incluant l'interception de fonctions natives et la manipulation de classes Java en temps réel[cite: 1].
