# TUTORIEL : Créer une VM Linux sur Mac pour tester OpenFang
## Installation SIMPLIFIÉE sans Docker (version directe)

**Durée estimée :** 30-45 minutes  
**Niveau :** Intermédiaire  
**Prérequis :** Mac avec 8GB+ RAM, 20GB espace disque libre

**Note importante :** Contrairement à la version précédente avec Docker, cette installation est **DIRECTE** dans la VM (comme conseillé par Bebox). La VM est déjà isolée, pas besoin de Docker en plus.

---

## ÉTAPE 1 : Choisir et installer le logiciel de virtualisation

### Option A - VirtualBox (Gratuit, Intel et Apple Silicon M1/M2/M3)
```bash
# Télécharger VirtualBox
https://www.virtualbox.org/wiki/Downloads

# Pour Mac Apple Silicon (M1/M2/M3), télécharger la version ARM64
# Pour Mac Intel, télécharger la version AMD64
```

### Option B - UTM (Gratuit, meilleur pour Apple Silicon)
```bash
# Télécharger UTM (plus léger et rapide sur Mac M1/M2/M3)
https://mac.getutm.app/

# Ou via Homebrew
brew install utm
```

**Recommandation :** UTM pour Mac récents (M1/M2/M3), VirtualBox pour Mac Intel

---

## ÉTAPE 2 : Télécharger une image Linux légère

### Option recommandée : Ubuntu Server LTS (léger, stable)
```bash
# Télécharger Ubuntu Server 24.04.4 LTS (sans interface graphique = plus léger)
https://ubuntu.com/download/server

# Version alternative : Debian (encore plus léger)
https://www.debian.org/distrib/netinst

# Ou Alpine Linux (ultra léger, 130MB)
https://alpinelinux.org/downloads/
```

**Choix pour OpenFang :** Ubuntu Server 24.04.4 LTS (dernière version LTS, stabilité/ressources)

---

## ÉTAPE 3 : Créer la VM (Exemple avec VirtualBox)

### 3.1 Configuration initiale
1. **Ouvrir VirtualBox**
2. **Nouvelle VM** (bouton bleu "New")
3. **Paramètres :**
   - Nom : `OpenFang-Test`
   - Type : Linux
   - Version : Ubuntu (64-bit) ou Debian (64-bit)

### 3.2 Allocation ressources (Configuration Mac 32GB)
```
Mémoire RAM : 8192 MB (8GB) minimum
               10240 MB (10GB) recommandé
               12288 MB (12GB) optimal pour 2 agents + Ollama
               
Disque dur :   40 GB (minimum)
               50 GB (recommandé avec 2 modèles 7B-8B)
               
Processeurs :  4 cœurs minimum
                6 cœurs recommandé
                
Vidéo :        128 MB (suffisant sans interface graphique)
```

**Note :** Avec 12GB RAM alloués à la VM, vous pouvez faire tourner simultanément :
- VEILLE (Llama 3.1 8B) ~4.5GB
- PROJETS (Qwen 3.5) ~4-5GB  
- Système Linux ~2GB
- Marge ~1-1.5GB

**Pour test comparatif :**
- 8GB : Un seul agent actif à la fois (switch manuel)
- 10-12GB : Les deux agents actifs simultanément

### 3.3 Configuration réseau
```
Mode : NAT (accès Internet par défaut)
OU
Mode : Bridge (si besoin d'IP locale visible)
```

### 3.4 Démarrer l'installation
1. Sélectionner l'ISO Ubuntu Server téléchargé
2. Lancer la VM
3. Suivre l'installation Ubuntu (langue, timezone, nom d'utilisateur)

**Astuce :** Choisir "Install OpenSSH server" lors de l'installation pour accès remote

---

## ÉTAPE 4 : Configuration Linux post-installation

### 4.1 Mise à jour système
```bash
# Se connecter à la VM (login/mdp créés pendant install)

# Mettre à jour
sudo apt update
sudo apt upgrade -y

# Installer outils essentiels
sudo apt install -y curl wget git htop build-essential
```

### 4.2 Configurer SSH (accès depuis Mac)
```bash
# Vérifier IP de la VM
ip addr show

# Normalement OpenSSH est déjà installé
# Sinon : sudo apt install openssh-server

# Depuis le Mac, vous pourrez vous connecter avec :
# ssh utilisateur@IP_VM
```

---

## ÉTAPE 5 : Installation OLLAMA (pour modèles IA)

### 5.1 Installer Ollama directement (PAS de Docker !)
```bash
# Installer Ollama (le moteur de LLM local)
curl -fsSL https://ollama.com/install.sh | sh

# Vérifier installation
ollama --version

# Démarrer le service Ollama
sudo systemctl start ollama
sudo systemctl enable ollama
```

### 5.2 Télécharger les modèles pour les agents
```bash
# Modèle pour VEILLE (synthèse web, rapide)
ollama pull llama3.1:8b

# Modèle pour PROJETS (technique, validé par Bebox)
ollama pull qwen3.5

# Vérifier les modèles installés
ollama list
```

**Tailles estimées :**
- llama3.1:8b ~4.5 GB
- qwen3.5 ~4-5 GB

**Temps de téléchargement :** 10-20 min selon connexion

---

## ÉTAPE 6 : Installation OpenFang (DIRECT, pas Docker)

### 6.1 Télécharger et installer OpenFang
```bash
# Créer dossier OpenFang
mkdir -p ~/openfang
cd ~/openfang

# Télécharger OpenFang (binaire ou depuis source)
# Méthode 1 : Binaire précompilé (si disponible)
wget https://github.com/RightNow-AI/openfang/releases/latest/download/openfang-linux-amd64
chmod +x openfang-linux-amd64
sudo mv openfang-linux-amd64 /usr/local/bin/openfang

# OU Méthode 2 : Depuis source (si binaire non dispo)
# git clone https://github.com/RightNow-AI/openfang.git
# cd openfang
# cargo build --release (nécessite Rust installé)
# sudo cp target/release/openfang /usr/local/bin/
```

### 6.2 Créer la structure des agents
```bash
# Créer les dossiers
mkdir -p ~/openfang/agents
mkdir -p ~/openfang/config
mkdir -p ~/openfang/shared_memory

cd ~/openfang
```

---

## ÉTAPE 7 : Configuration des Agents

### 7.1 Configuration Agent VEILLE
**Créer le fichier `~/openfang/agents/veille.yaml` :**
```yaml
name: "VEILLE"
model: "llama3.1:8b"
provider: "ollama"
system_prompt: |
  Tu es VEILLE, agent de veille informationnelle.
  Rôle : Surveiller actualités tech, pharmacie, modélisme.
  Style : Concis, factuel, sources citées.
  Format : Bullet points, 3-5 items max.
  
  Domaines de veille :
  - IA et nouvelles technologies
  - Actualités pharma / officine
  - Nouveautés modélisme ferroviaire
  - Outils et logiciels utiles

tools:
  - name: "web_search"
    endpoint: "http://192.168.68.60:8080"  # Votre SearXNG
    type: "searxng"
  
  - name: "rss_reader"
    feeds: 
      - "https://techcrunch.com/feed"
      - "https://www.lemonde.fr/sciences/rss_full.xml"
      - "https://www.model-railroader.com/rss"

schedule:
  - cron: "0 7 * * *"  # 7h00 quotidien
    action: "briefing_matin"
    description: "Envoyer résumé actus à BOSLEY"

memory:
  persist: true
  retention: "7d"  # Garder historique 7 jours
```

### 7.2 Configuration Agent PROJETS
**Créer le fichier `~/openfang/agents/projets.yaml` :**
```yaml
name: "PROJETS"
model: "qwen3.5"
provider: "ollama"
system_prompt: |
  Tu es PROJETS, agent technique et créatif.
  Domaines : Modélisme ferroviaire, Arduino, organisation familiale.
  Style : Précis, pédagogique, avec exemples concrets.
  
  Spécialités :
  1. Conversions d'échelle (HO 1/87, N 1/160, O 1/48)
  2. Plans réseau ferroviaire et cotes
  3. Code Arduino et tutoriels
  4. Rappels famille et organisation
  
  Réponses :
  - Toujours avec calculs précis
  - Proposer plusieurs options quand possible
  - Mémoriser projets en cours

tools:
  - name: "scale_calculator"
    type: "math"
    scales: [87, 160, 48, 22.5]  # HO, N, O, G
  
  - name: "github"
    token: "${GITHUB_TOKEN}"  # À configurer si besoin
    repos: ["Arduino-Projects"]
  
  - name: "family_calendar"
    type: "reminder"
    sources: ["TODO.md", "MEMORY.md"]
  
  - name: "arduino_snippets"
    path: "/app/data/arduino_snippets"

memory:
  persist: true
  projects_path: "/app/data/projects"
  active_projects:
    - "Reseau_UCI_2026"
    - "Arduino_Stepper"
    - "Documentation_Famille"
```

### 7.3 Configuration Gateway (Routage)
**Créer le fichier `~/openfang/config/gateway.yaml` :**
```yaml
gateway:
  name: "OpenFang-Gateway"
  version: "1.0"
  
  agents:
    - name: "BOSLEY"
      endpoint: "telegram://@ilcamerlingo"  # Ou webhook vers votre OpenClaw
      role: "coordinator"
      priority: 1
      
    - name: "VEILLE"
      endpoint: "http://localhost:8081"
      role: "specialist"
      domain: ["news", "tech", "pharma", "veille"]
      priority: 2
      
    - name: "PROJETS"
      endpoint: "http://localhost:8082"
      role: "specialist"
      domain: ["arduino", "modelisme", "echelle", "famille", "calcul"]
      priority: 2

  routing_rules:
    default: "BOSLEY"
    
    rules:
      - name: "actualites"
        keywords: ["actualité", "news", "nouveau", "dernier", "pharmacie", "tech", "ia", "startup"]
        target: "VEILLE"
        confidence: 0.8
        
      - name: "technique_modelisme"
        keywords: ["arduino", "échelle", "ho", "n", "rails", "loco", "décodeur", "dcc", "cm", "pouces", "conversion"]
        target: "PROJETS"
        confidence: 0.9
        
      - name: "calculs"
        keywords: ["calcul", "dimension", "plan", "schéma", "cotes", "mm", "cm", "mètre"]
        target: "PROJETS"
        confidence: 0.8
        
      - name: "famille"
        keywords: ["rappel", "anniversaire", "sophie", "quentin", "amélie", "paul", "clarice", "famille"]
        target: "PROJETS"
        confidence: 0.9
        
      - name: "multi_agents"
        keywords: ["compare", "analyse", "synthese", "complexe", "urgent"]
        target: ["VEILLE", "PROJETS"]
        mode: "parallel"
        
  fallback:
    timeout: 30
    max_retries: 2
    default_on_failure: "BOSLEY"
```

---

## ÉTAPE 8 : Lancer OpenFang

### 8.1 Démarrer les services
```bash
cd ~/openfang

# Démarrer Ollama (déjà fait normalement)
sudo systemctl start ollama

# Démarrer OpenFang avec les agents
openfang start --config ./config/gateway.yaml

# OU si nécessaire de spécifier les agents
openfang start \
  --agent veille:./agents/veille.yaml \
  --agent projets:./agents/projets.yaml \
  --gateway ./config/gateway.yaml
```

### 8.2 Vérifier que tout fonctionne
```bash
# Vérifier Ollama
curl http://localhost:11434/api/tags

# Vérifier OpenFang
curl http://localhost:8080/health

# Voir les processus actifs
ps aux | grep openfang
ps aux | grep ollama
```

### 8.3 Accès depuis Mac
```bash
# Dans le Mac, ouvrir navigateur :
# http://IP_VM:8080

# Ou en ligne de commande :
curl http://IP_VM:8080/agents
```

---

## ÉTAPE 9 : Tests des agents

### 9.1 Test VEILLE
```bash
# Envoyer requête à VEILLE
curl -X POST http://IP_VM:8081/agents/VEILLE/message \
  -H "Content-Type: application/json" \
  -d '{"message": "Quelles sont les dernières nouvelles tech ?"}'
```

### 9.2 Test PROJETS
```bash
# Envoyer requête à PROJETS
curl -X POST http://IP_VM:8082/agents/PROJETS/message \
  -H "Content-Type: application/json" \
  -d '{"message": "Convertis 10 cm en échelle HO"}'
```

### 9.3 Test Gateway (routage auto)
```bash
# Le Gateway doit router automatiquement
curl -X POST http://IP_VM:8080/gateway/message \
  -H "Content-Type: application/json" \
  -d '{"message": "J'ai besoin des dernières actus IA"}'
```

---

## ÉTAPE 10 : Optimisation VM (optionnel)

### 10.1 Partage de fichiers Mac ↔ VM
```bash
# Dans VirtualBox : Devices > Shared Folders
# Ajouter un dossier Mac partagé
# Monter dans Linux :
sudo mkdir -p /mnt/macshare
sudo mount -t vboxsf nom_du_partage /mnt/macshare
```

### 10.2 Snapshots (sauvegardes)
1. **Éteindre la VM**
2. **VirtualBox** → Snapshot → Take
3. **Nommer** : "OpenFang installé propre"
4. **Utilité** : Revenir à cet état si problème

### 10.3 Réduire empreinte mémoire
```bash
# Désactiver services inutiles
sudo systemctl disable snapd
sudo apt remove --purge -y snapd

# Nettoyer
sudo apt autoremove -y
sudo apt clean
```

---

## CHECKLIST FINALE

- [ ] VirtualBox/UTM installé sur Mac
- [ ] VM Linux créée (8-12GB RAM, 40-50GB disque)
- [ ] Ubuntu Server 24.04.4 LTS installé et mis à jour
- [ ] **Ollama installé DIRECTEMENT** (pas Docker)
- [ ] Modèles llama3.1:8b et qwen3.5 téléchargés
- [ ] **OpenFang installé DIRECTEMENT** (pas Docker)
- [ ] Agents VEILLE et PROJETS configurés
- [ ] Gateway configuré avec routage
- [ ] Accès interface web depuis Mac (http://IP_VM:8080)
- [ ] SSH configuré (accès terminal Mac → VM)
- [ ] Snapshot créé pour backup
- [ ] Tests des 3 agents réussis

---

## COMMANDES UTILES

### Sur Mac (hôte)
```bash
# Démarrer VM headless (sans fenêtre)
VBoxManage startvm "OpenFang-Test" --type headless

# Arrêter VM proprement
VBoxManage controlvm "OpenFang-Test" acpipowerbutton

# Voir IP VM
VBoxManage guestproperty get "OpenFang-Test" "/VirtualBox/GuestInfo/Net/0/V4/IP"

# SSH rapide
ssh utilisateur@IP_VM
```

### Dans VM Linux
```bash
# Redémarrer OpenFang
sudo systemctl restart openfang

# Voir logs
journalctl -u openfang -f

# Voir logs Ollama
journalctl -u ollama -f

# Arrêter OpenFang
sudo systemctl stop openfang

# Arrêter Ollama
sudo systemctl stop ollama

# Mettre à jour modèles
ollama pull llama3.1:8b
ollama pull qwen3.5
```

---

## ARCHITECTURE FINALE (SIMPLIFIÉE)

```
Mac (32GB RAM)
└── VM Ubuntu 24.04.4 (8-12GB RAM, 4-6 CPUs)  ← ISOLÉE
    ├── Ollama (natif, PAS Docker)
    │   ├── Modèle llama3.1:8b (VEILLE)
    │   └── Modèle qwen3.5 (PROJETS)
    ├── OpenFang (natif, PAS Docker)
    │   ├── Gateway (routage)
    │   ├── Agent VEILLE
    │   └── Agent PROJETS
    └── Mémoire partagée (fichiers YAML)
```

**Avantages simplification :**
- ✅ Moins de couches (pas de Docker overhead)
- ✅ Plus rapide à démarrer
- ✅ Moins de RAM utilisée
- ✅ Même isolation (VM déjà isolée)

---

## DÉPANNAGE

### Problème : Ollama ne démarre pas
```bash
# Vérifier statut
sudo systemctl status ollama

# Voir logs
sudo journalctl -u ollama -n 50

# Redémarrer
sudo systemctl restart ollama
```

### Problème : OpenFang ne démarre pas
```bash
# Vérifier si exécutable
which openfang

# Voir logs
journalctl -u openfang -n 50

# Tester config
openfang validate --config ./config/gateway.yaml
```

### Problème : Pas d'accès depuis Mac
```bash
# Vérifier firewall VM
sudo ufw status
sudo ufw allow 8080/tcp
sudo ufw allow 8081/tcp
sudo ufw allow 8082/tcp

# Vérifier IP VM
ip addr show

# Tester connexion
ping IP_VM
```

---

## NEXT STEPS

Une fois OpenFang testé :
1. **Comparer** avec architecture Jill actuelle (Pi 5)
2. **Décider** si migration intéressante ou non
3. **Si oui** : Planifier bascule progressive
4. **Si non** : Supprimer VM, garder Jill

**Rappel important :**
- **Jill (Pi 5)** fonctionne TOUJOURS indépendamment
- **OpenFang (VM)** = tests uniquement
- Les deux peuvent coexister pour comparaison

**Bonne découverte d'OpenFang !** 🚀

---

*Document créé : 5 mars 2026*  
*Version : Installation directe sans Docker (simplifiée selon conseil Bebox)*
