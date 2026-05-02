#  Lab : Analyse Statique et Dynamique Android avec MobSF

Ce dépôt documente un laboratoire pratique de tests d'intrusion mobile (Android). L'objectif de ce lab est de déployer un environnement d'analyse, d'intercepter le trafic et d'identifier des vulnérabilités critiques au sein d'une application Android délibérément vulnérable en utilisant **MobSF (Mobile Security Framework)**.

##  Cible
*   **Application :** [DIVA (Damn Insecure and Vulnerable App)](https://github.com/payatu/diva-android)
*   **Vecteurs d'attaque :** Stockage non sécurisé, fuite d'informations, problèmes de logique.

##  Architecture et Prérequis

Pour reproduire cet environnement d'analyse, les outils suivants ont été utilisés :
*   **Conteneur d'analyse :** Docker (pour isoler et exécuter MobSF). *(Note : L'environnement Mobexler peut également être utilisé).*
*   **Émulateur Android :** Android Studio (AVD) avec une image système **API 30 (Android 11) SANS Google Play** pour faciliter l'accès Root.
*   **Outils CLI :** `adb` (Android Debug Bridge).

---

##  Étape 1 : Déploiement de l'environnement

### 1. Préparation de l'Émulateur
L'émulateur doit être lancé avec les droits d'écriture sur le système de fichiers pour permettre à MobSF d'y injecter ses certificats d'interception et ses agents (Frida).

```bash
# Lancement de l'AVD (ex: MobSF_DIVA_API_30)
emulator -avd MobSF_DIVA_API_30 -writable-system -no-snapshot &
```` 
# Vérification de l'identifiant de l'appareil
adb devices
# Résultat attendu : emulator-5554 device

## 2. Démarrage de MobSF via Docker
Afin de réaliser l'Analyse Dynamique, le conteneur MobSF doit être lié à l'émulateur tournant sur la machine hôte via la variable d'environnement MOBSF_ANALYZER_IDENTIFIER.
(Commande adaptée pour les terminaux Windows, exécutée sur une seule ligne) :
````Bash
docker run -it --rm -p 8000:8000 -e MOBSF_ANALYZER_IDENTIFIER=emulator-5554 opensecurity/mobile-security-framework-mobsf:latest
````
L'interface de MobSF est ensuite accessible via : http://localhost:8000.

#  Étape 2 : Méthodologie d'Audit
## 1. Analyse Statique
L'APK diva.apk a été soumis à l'interface de MobSF. L'outil a procédé au reverse engineering de l'application (décompilation) pour analyser :
Le manifeste Android (AndroidManifest.xml).
Les permissions requises.
La présence potentielle de secrets codés en dur (Hardcoded secrets).
## 2. Analyse Dynamique (Runtime)
L'analyse dynamique a été initiée directement depuis MobSF. L'outil automatise :
L'installation de l'APK sur l'émulateur.
Le lancement du serveur Frida pour le hooking.
La configuration du proxy HTTPS global pour l'interception réseau.

#  Résultats de l'Audit : Preuves de Concept (PoC)
Au cours de l'exploration dynamique, plusieurs vulnérabilités ont été identifiées. Voici deux exemples concrets extraits de ce lab :
## Vulnérabilité : Insecure Logging (Fuite de données dans les journaux)
Description : L'application écrit des données sensibles en clair dans les logs système (Logcat) de l'appareil Android.
Reproduction : Dans le challenge "1. Insecure Logging" de DIVA, la saisie d'un numéro de carte de crédit déclenche une erreur.
Observation MobSF : En surveillant le Logcat Stream depuis le tableau de bord MobSF, le faux numéro de carte de crédit tapé dans l'application est apparu en texte clair dans les logs interceptés.
Impact : Toute application tierce malveillante disposant de la permission de lire les journaux système pourrait exfiltrer ces données critiques.

