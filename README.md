# 🛡️ Lab : Analyse Statique Automatisée avec MobSF (Docker Edition)

Ce laboratoire documente l'audit de sécurité statique d'une application Android (APK) en utilisant **MobSF (Mobile Security Framework)**. L'objectif est d'automatiser la détection de vulnérabilités, de configurer un environnement de scan via Docker et de corréler les résultats avec le standard **OWASP MASVS**.

> **Environnement :** Kali Linux + Docker (Podman). 
> **Périmètre :** Analyse défensive sur APK pédagogique dans un cadre strictement légal.

---

## 🛠️ Phase 1 : Préparation du Workspace

Avant de lancer l'analyse, nous préparons les fichiers et calculons l'empreinte de l'APK pour garantir l'intégrité de l'audit.

```bash
# Création du dossier d'audit
mkdir -p ~/mobsf_analysis/$(date +%Y-%m-%d)
cd ~/mobsf_analysis/$(date +%Y-%m-%d)

# Calcul du hash SHA-256 de l'APK
sha256sum app-debug.apk > apk_hash.txt
cat apk_hash.txt
```
<img width="945" height="216" alt="image" src="https://github.com/user-attachments/assets/dc786c43-8d80-42fd-b2ff-c46bfeedd335" />


---

## 🐳 Phase 2 : Déploiement de MobSF avec Docker

Plutôt qu'une installation locale complexe, nous utilisons Docker pour isoler l'outil. Sous Kali Linux (utilisant Podman), il est nécessaire de spécifier le registre complet (`docker.io`).

### Étape 1 : Lancement du conteneur
```bash
# Commande de déploiement (Port 8000)
sudo docker run -it --rm -p 8000:8000 docker.io/opensecurity/mobile-security-framework-mobsf:latest
```
*Action : Attendre le message "Server is running on http://0.0.0.0:8000".*

<img width="1209" height="509" alt="image" src="https://github.com/user-attachments/assets/a34bad1c-fb0a-432e-8618-5682f78542b8" />


<img width="1187" height="414" alt="image" src="https://github.com/user-attachments/assets/9f727cd0-ac2d-4cd7-90c2-9096fb832657" />



### Étape 2 : Accès à l'interface
Ouvrir le navigateur à l'adresse suivante : `http://127.0.0.1:8000`.

<img width="1457" height="717" alt="image" src="https://github.com/user-attachments/assets/6a7ae0f1-a871-4e57-875d-3003475a62f6" />


---

## 🔍 Phase 3 : Analyse Statique et Triage

Nous téléversons l'APK (`Static Analysis`) et explorons le rapport généré par MobSF.

### Étape 3 : Analyse du Manifeste (Surface d'Attaque)
MobSF décompile l'APK et analyse le fichier `AndroidManifest.xml`.

<img width="1305" height="629" alt="image" src="https://github.com/user-attachments/assets/676e0c08-086b-49c7-a00e-af08e646199a" />


*Points d'attention :*
- **Composants exportés :** `android:exported="true"`.
- **Sauvegarde :** `android:allowBackup="true"`.
- **Mode Debug :** `android:debuggable="true"`.

<img width="1918" height="523" alt="image" src="https://github.com/user-attachments/assets/a7b7ec32-ee88-4dfe-8a5e-d96b001fbb0e" />


### Étape 4 : Sécurité Réseau et Secrets
Nous vérifions si l'application expose des données lors des communications ou dans son code source.

*Observations :*
- **Trafic en clair :** Présence de `android:usesCleartextTraffic="true"` (Risque d'interception MitM).
- **Secrets hardcodés :** Recherche de clés API ou tokens dans l'onglet "Hardcoded Secrets".

<img width="1906" height="488" alt="image" src="https://github.com/user-attachments/assets/d367335c-17ec-4ffd-bfc7-6aa90f91f554" />


---

## 📊 Phase 4 : Rapport d'Audit (Synthèse)

### Informations Générales
- **Analyste :** Youssef CHARAF
- **Cible :** `app-debug.apk`
- **Score de Sécurité :** [Indiquer le score MobSF, ex: 45/100]

### 🔴 Top 3 Vulnérabilités (OWASP MASVS)

| Vulnérabilité | Sévérité | MASVS ID | Impact |
| :--- | :--- | :--- | :--- |
| **Secrets en clair** | Élevée | MSTG-STORAGE-14 | Vol de clés API et accès aux services tiers. |
| **Trafic HTTP** | Moyenne | MSTG-NETWORK-1 | Interception des données (MitM). |
| **Composant Exporté** | Moyenne | MSTG-PLATFORM-2 | Accès non autorisé à des fonctionnalités internes. |

### 💡 Recommandations
1. **Désactiver le trafic en clair** en forçant le HTTPS via une `Network Security Configuration`.
2. **Obfusquer les secrets** et ne jamais stocker de clés API sensibles en clair dans les ressources XML.
3. **Restreindre les composants** : mettre `android:exported="false"` par défaut.

**![Capture 6 : Export du rapport final PDF](assets/capture_6_report.png)**
```

***

### 🎓 Pour aller plus loin : Évaluer la criticité
Dans un audit professionnel, identifier la faille ne suffit pas ; il faut évaluer son risque (Impact x Probabilité). Pour t'aider dans ton rapport, voici un outil interactif pour calculer la sévérité des failles que tu as trouvées avec MobSF.

```json?chameleon
{"component":"LlmGeneratedComponent","props":{"height":"600px","prompt":"Crée un simulateur de triage de vulnérabilités MobSF. L'objectif est d'aider l'étudiant à classer les failles trouvées. \n\nStructure : \n1. Une liste déroulante pour choisir la vulnérabilité détectée (ex: allowBackup=true, Hardcoded API Key, usesCleartextTraffic=true, Exported Activity).\n2. Des curseurs pour évaluer l'impact (0-10) et la facilité d'exploitation (0-10).\n3. Affichage dynamique d'un score de risque global (Impact * Facilité) avec une étiquette de sévérité (Faible, Moyen, Élevé, Critique).\n4. Affichage automatique de la recommandation de remédiation et de la règle OWASP MASVS correspondante pour chaque type de vulnérabilité.\n\nComportement : \n- Initialiser avec 'Hardcoded API Key' comme exemple.\n- Utiliser des calculs mathématiques pour la sévérité.\n- Ne pas utiliser de couleurs nommées spécifiques dans le prompt.\n- Langue : Français.","id":"im_0999bc79b93d5bfa"}}
```
