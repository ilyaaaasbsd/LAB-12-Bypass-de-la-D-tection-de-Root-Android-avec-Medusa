# LAB-12-Bypass-de-la-D-tection-de-Root-Android-avec-Medusa
 Rapport d’audit – Bypass de détection Root Android avec Medusa
 Informations générales
Projet : Bypass de détection Root Android
Application cible : Root Check App
Package : com.example.rootcheck
Version : 1.0
Date : 25/3/2026
Auditeur : ilyas
 Résumé exécutif

Dans ce laboratoire, nous avons étudié les mécanismes de détection de root dans une application Android et mis en œuvre un contournement à l’aide de l’outil Medusa (basé sur Frida).

Les tests ont démontré que :

L’application détecte correctement un environnement rooté en conditions normales.
Après injection des hooks via Medusa, les mécanismes de détection sont neutralisés.
L’application fonctionne comme si l’appareil n’était pas rooté.

Ce type de vulnérabilité montre que les mécanismes de protection côté client peuvent être contournés et ne doivent pas être considérés comme fiables pour des décisions critiques de sécurité.

 Objectifs du lab
Comprendre les techniques de détection de root Android
Utiliser Medusa pour bypass ces détections
Valider le contournement via instrumentation dynamique
Diagnostiquer les erreurs d’injection
 Environnement de test
 Machine hôte
OS : Windows / Linux / macOS
Python : 3.10+
Outils installés :
frida
frida-tools
adb
medusa
 Appareil Android
Version : Android 10+
Mode développeur activé
Débogage USB activé
Connexion USB active
 Mise en place
1. Vérification des outils
frida --version
python -c "import frida; print(frida.__version__)"
adb devices

 Résultat : outils correctement installés et appareil détecté.

2. Lancement de frida-server
adb push frida-server /data/local/tmp/
adb shell chmod 755 /data/local/tmp/frida-server
adb shell "/data/local/tmp/frida-server -l 0.0.0.0"

 frida-server lancé avec succès.

3. Vérification de la connexion
frida-ps -Uai

 Liste des applications affichée.

 Analyse de la détection Root
Techniques identifiées :
 Java
android.os.Build.TAGS → test-keys
java.io.File.exists() → vérification de /system/xbin/su
Runtime.exec("su")
Librairie RootBeer
 Natif (C/C++)
open / access / stat
Lecture de /proc/mounts
 Exploitation – Bypass avec Medusa
Commande utilisée
medusa --usb --spawn com.example.rootcheck --module root-bypass
Alternative (attach)
medusa --usb --attach "com.example.rootcheck" --module root-bypass
 Résultats
 Avant bypass
Message affiché : "Root detected"
Fonctionnalités bloquées
 Après bypass
Message affiché : "Not rooted"
Application fonctionnelle
 Logs observés
[+] Build.TAGS -> release-keys
[+] File.exists bypass for /system/xbin/su
[+] Blocked Runtime.exec: su
[+] RootBeer.isRooted -> false

 Les hooks sont correctement exécutés.

 Plan B – Frida pur

En cas d’échec de Medusa, utilisation de scripts Frida.

Commande :
frida -U -f com.example.rootcheck -l bypass_root.js --no-pause

 Résultat identique : bypass réussi.

 Analyse des risques
ID	Vulnérabilité	Impact	Sévérité
V1	Détection root contournable	Accès non autorisé	Élevée
V2	Vérification côté client uniquement	Manipulation possible	Critique
V3	Absence de protection anti-instrumentation	Hooking facile	Élevée
 Recommandations
 1. Implémenter des vérifications côté serveur
Ne pas se baser uniquement sur le client
 2. Ajouter des protections anti-Frida
Détection de ports (27042/27043)
Détection de chaînes "frida"
3. Obfuscation du code
ProGuard / R8
 4. Renforcer les checks natifs
Vérifications multi-couches
 5. Implémenter SafetyNet / Play Integrity API
Vérification d’intégrité du device
 Mapping OWASP MASVS
ID	Vulnérabilité	Référence MASVS
V1	Root bypass	MSTG-RESILIENCE-1
V2	Protection insuffisante	MSTG-PLATFORM-1
V3	Absence anti-tampering	MSTG-RESILIENCE-3
 Preuves
/preuves/
  /logs/
    - medusa_logs.txt
    - frida_logs.txt
  /captures/
    - before_root_detected.png
    - after_bypass.png
 Conclusion

Ce laboratoire a démontré que :

Les mécanismes de détection de root peuvent être contournés facilement
Les protections côté client sont insuffisantes seules
L’instrumentation dynamique (Medusa / Frida) est une technique puissante

 Il est essentiel d’adopter une approche de sécurité en profondeur (Defense in Depth).

 Checklist
 Frida installé et fonctionnel
 frida-server opérationnel
 Medusa configuré
 Bypass réalisé
 Logs collectés
 Rapport documenté
📎 Références
https://frida.re/
https://github.com/frida/frida
https://owasp.org/www-project-mobile-security-testing-guide/
https://developer.android.com
