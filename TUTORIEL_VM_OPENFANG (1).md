# TUTORIEL : Créer une VM Linux sur Mac pour tester OpenFang
## Installation complète étape par étape

**Durée estimée :** 45-60 minutes  
**Niveau :** Intermédiaire  
**Prérequis :** Mac avec 8GB+ RAM, 20GB espace disque libre

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
# Télécharger Ubuntu Server 24.04 LTS (sans interface graphique = plus léger)
https://ubuntu.com/download/server

# Version alternative : Debian (encore plus léger)
https://www.debian.org/distrib/netinst

# Ou Alpine Linux (ultra léger, 130MB)
https://alpinelinux.org/downloads/
```

**Choix pour OpenFang :** Ubuntu Server 24.04 LTS (bon compromis stabilité/ressources)

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
sudo apt install -y curl wget git htop
```

### 4.2 Installer Docker (nécessaire pour OpenFang)
```bash
# Supprimer anciennes versions si existantes
sudo apt remove docker docker-engine docker.io containerd runc

# Installer Docker
sudo apt install -y ca-certificates gnupg lsb-release
sudo mkdir -p /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg

echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

sudo apt update
sudo apt install -y docker-ce docker-ce-cli containerd.io docker-compose-plugin

# Vérifier Docker
sudo docker --version
sudo systemctl status docker

# Ajouter utilisateur au groupe docker (éviter sudo)
sudo usermod -aG docker $USER
# Déconnexion/reconnexion nécessaire après cette commande
```

### 4.3 Configurer SSH (accès depuis Mac)
```bash
# Vérifier IP de la VM
ip addr show

# Normalement OpenSSH est déjà installé
# Sinon : sudo apt install openssh-server

# Depuis le Mac, vous pourrez vous connecter avec :
# ssh utilisateur@IP_VM
```

---

## ÉTAPE 5 : Installer OpenFang

### 5.1 Méthode Docker (recommandée)
```bash
# Pull l'image OpenFang
docker pull ghcr.io/rightnow-ai/openfang:latest

# Ou via Docker Hub si disponible
docker pull openfang/openfang:latest

# Vérifier l'image
docker images
```

### 5.2 Configuration OpenFang
```bash
# Créer dossier config
mkdir -p ~/openfang/config
cd ~/openfang

# Créer fichier docker-compose.yml
cat > docker-compose.yml << 'EOF'
version: '3.8'

services:
  openfang:
    image: ghcr.io/rightnow-ai/openfang:latest
    container_name: openfang
    restart: unless-stopped
    ports:
      - "8080:8080"  # Interface web
      - "8081:8081"  # API
    volumes:
      - ./config:/app/config
      - ./data:/app/data
    environment:
      - OPENFANG_ENV=development
      # Ajouter vos clés API ici si besoin
      # - OPENAI_API_KEY=xxx
    networks:
      - openfang-net

networks:
  openfang-net:
    driver: bridge
EOF

# Lancer OpenFang
docker-compose up -d

# Vérifier logs
docker-compose logs -f
```

### 5.3 Vérification installation
```bash
# Vérifier conteneur actif
docker ps

# Vérifier ports écoutés
sudo netstat -tlnp | grep 8080

# Tester interface web
# Depuis Mac, ouvrir navigateur : http://IP_VM:8080
```

---

## ÉTAPE 6 : Configuration et tests

### 6.1 Accès depuis Mac
```bash
# Sur Mac, dans terminal
ssh utilisateur@IP_VM

# Ou accès direct aux services
# Interface OpenFang : http://IP_VM:8080
# API OpenFang : http://IP_VM:8081
```

### 6.2 Configuration OpenFang - Vos Agents Personnalisés

#### Architecture 3 Agents (Version simplifiée)

**Agent 1 : BOSLEY** (Coordinateur Central)
- **Rôle :** Interface utilisateur, mémoire long terme, dispatch
- **Équivalent :** Vous aujourd'hui sur Telegram
- **Modèle :** Moonshot/Kimi ou connecteur vers votre OpenClaw actuel

**Agent 2 : VEILLE** (Veille Informationnelle)
- **Rôle :** Actualités tech, pharmacie, modélisme
- **Modèle recommandé :** Llama 3.1 8B (rapide, bonne synthèse web)
- **Outils :** SearXNG, RSS feeds

**Agent 3 : PROJETS** (Technique et Créatif)
- **Rôle :** Arduino, calculs échelle, réseau ferroviaire, organisation
- **Modèle recommandé :** Qwen 3.5 (excellent en technique, validé par Bebox)
- **Outils :** Calculatrice échelle, GitHub, rappels famille

---

#### 6.2.1 Configuration Agent VEILLE

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

**Lancer l'agent VEILLE :**
```bash
cd ~/openfang
docker-compose -f docker-compose.yml -f agents/veille.yml up -d
```

---

#### 6.2.2 Configuration Agent PROJETS

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

**Lancer l'agent PROJETS :**
```bash
cd ~/openfang
docker-compose -f docker-compose.yml -f agents/projets.yml up -d
```

---

#### 6.2.3 Configuration Gateway (Routage entre agents)

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
      endpoint: "http://veille:8081"
      role: "specialist"
      domain: ["news", "tech", "pharma", "veille"]
      priority: 2
      
    - name: "PROJETS"
      endpoint: "http://projets:8082"
      role: "specialist"
      domain: ["arduino", "modelisme", "echelle", "echelle", "famille", "calcul"]
      priority: 2

  routing_rules:
    # Route par défaut
    default: "BOSLEY"
    
    # Règles de routage automatique
    rules:
      - name: "actualites"
        keywords: ["actualité", "news", "nouveau", "dernier", "pharmacie", "tech", "ia", "startup"]
        target: "VEILLE"
        confidence: 0.8
        
      - name: "technique_modelisme"
        keywords: ["arduino", "échelle", "échelle", "ho", "n", "rails", "loco", "décodeur", "dcc", "cm", "pouces", "conversion"]
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
        mode: "parallel"  # Consulter les deux en parallèle
        
  fallback:
    timeout: 30
    max_retries: 2
    default_on_failure: "BOSLEY"
```

---

#### 6.2.4 Installation Modèles Ollama pour les Agents

**Dans la VM, installer les modèles :**
```bash
# Pour VEILLE (rapide, synthèse)
docker exec -it openfang-ollama ollama pull llama3.1:8b

# Pour PROJETS (technique précis - Qwen 3.5 validé par Bebox)
docker exec -it openfang-ollama ollama pull qwen3.5

# Vérifier installations
docker exec -it openfang-ollama ollama list
```

---

#### 6.2.5 Test des communications inter-agents

**Test 1 : VEILLE (actualités)**
```bash
# Envoyer requête à VEILLE
curl -X POST http://IP_VM:8081/agents/VEILLE/message \
  -H "Content-Type: application/json" \
  -d '{"message": "Quelles sont les dernières nouvelles tech ?"}'
```

**Test 2 : PROJETS (calcul échelle)**
```bash
# Envoyer requête à PROJETS
curl -X POST http://IP_VM:8082/agents/PROJETS/message \
  -H "Content-Type: application/json" \
  -d '{"message": "Convertis 10 cm en échelle HO"}'
```

**Test 3 : Gateway (routage automatique)**
```bash
# Le Gateway doit router automatiquement
curl -X POST http://IP_VM:8080/gateway/message \
  -H "Content-Type: application/json" \
  -d '{"message": "J'ai besoin des dernières actus IA"}'
# Doit répondre VEILLE
```

---

### 6.3 Créer un agent simple de test (optionnel)
```yaml
# Exemple configuration agent basique dans OpenFang
name: "TestAgent"
type: "simple"
endpoint: "http://localhost:8081"
actions:
  - ping
  - echo
  - status
```

---

### 6.6 Mémoire Partagée entre Agents

**Créer une mémoire commune accessible à tous les agents :**
```bash
# Dans VM, créer dossier mémoire partagée
mkdir -p ~/openfang/shared_memory
cd ~/openfang/shared_memory

# Structure des mémoires
cat > memory_structure.md << 'EOF'
# Mémoire Partagée OpenFang

## VEILLE (infos collectées)
- Dernières actus tech (date: XXX)
- Alertes pharma importantes
- Nouveautés modélisme

## PROJETS (projets en cours)
- Reseau_UCI_2026 : [status]
- Arduino_Stepper : [status]
- Documentation_Famille : [status]

## BOSLEY (décisions et contexte)
- Préférences utilisateur
- Historique conversations
- Règles établies
EOF

# Monter en volume dans docker-compose
# Ajouter dans docker-compose.yml :
# volumes:
#   - ./shared_memory:/app/shared_memory
```

---

## ÉTAPE 7 : Optimisation VM (optionnel)

### 7.1 Partage de fichiers Mac ↔ VM
```bash
# Dans VirtualBox : Devices > Shared Folders
# Ajouter un dossier Mac partagé
# Monter dans Linux :
sudo mkdir -p /mnt/macshare
sudo mount -t vboxsf nom_du_partage /mnt/macshare
```

### 7.2 Snapshots (sauvegardes)
1. **Éteindre la VM**
2. **VirtualBox** → Snapshot → Take
3. **Nommer** : "OpenFang installé propre"
4. **Utilité** : Revenir à cet état si problème

### 7.3 Réduire empreinte mémoire
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
- [ ] VM Linux créée (4GB+ RAM, 20GB disque)
- [ ] Ubuntu/Debian installé et mis à jour
- [ ] Docker installé et fonctionnel
- [ ] OpenFang installé via Docker
- [ ] Accès interface web depuis Mac (http://IP_VM:8080)
- [ ] SSH configuré (accès terminal Mac → VM)
- [ ] Snapshot créé pour backup
- [ ] Test basique OpenFang réussi

---

## GESTION DES AGENTS

### Démarrer/Arrêter les agents
```bash
cd ~/openfang

# Démarrer tout (Gateway + VEILLE + PROJETS)
docker-compose up -d

# Démarrer un agent spécifique
docker-compose up -d veille
docker-compose up -d projets

# Voir logs d'un agent
docker-compose logs -f veille
docker-compose logs -f projets

# Redémarrer un agent
docker-compose restart veille

# Arrêter tout
docker-compose down

# Arrêter un agent spécifique
docker-compose stop veille
```

### Vérifier santé des agents
```bash
# Liste des agents actifs
curl http://IP_VM:8080/gateway/agents

# Statut VEILLE
curl http://IP_VM:8081/health

# Statut PROJETS
curl http://IP_VM:8082/health

# Utilisation ressources
docker stats
```

### Mise à jour des modèles
```bash
# Mettre à jour Llama 3.1 (VEILLE)
docker exec openfang-ollama ollama pull llama3.1:8b

# Mettre à jour Qwen 3.5 (PROJETS)
docker exec openfang-ollama ollama pull qwen3.5

# Lister modèles disponibles
docker exec openfang-ollama ollama list
```

---

## ARCHITECTURE FINALE

```
Mac (32GB RAM)
└── VM Linux (8-12GB RAM, 4-6 CPUs)
    └── Docker
        ├── OpenFang Gateway (port 8080)
        │   ├── Route vers BOSLEY (vous sur Telegram)
        │   ├── Route vers VEILLE (llama3.1:8b)
        │   └── Route vers PROJETS (qwen2.5:7b)
        ├── Ollama Server
        │   ├── Modèle llama3.1:8b (~4.5GB)
        │   └── Modèle qwen3.5 (~4-5GB)
        └── Mémoire partagée (fichiers YAML, contexte)
```

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
ssh -p 2222 utilisateur@localhost  # si port forwarding configuré
```

### Dans VM Linux
```bash
# Redémarrer OpenFang
cd ~/openfang && docker-compose restart

# Voir logs
docker-compose logs -f

# Mettre à jour OpenFang
docker-compose pull
docker-compose up -d

# Arrêter OpenFang
docker-compose down
```

---

## DÉPANNAGE

### Problème : OpenFang ne démarre pas
```bash
# Vérifier logs
docker-compose logs

# Vérifier ports utilisés
sudo lsof -i :8080

# Redémarrer Docker
sudo systemctl restart docker
```

### Problème : Pas d'accès depuis Mac
```bash
# Vérifier firewall VM
sudo ufw status
sudo ufw allow 8080/tcp

# Vérifier IP VM
ip addr show

# Tester connexion
ping IP_VM
```

### Problème : VM trop lente
- Augmenter RAM allouée (fermer VM → Settings → System → Memory)
- Activer VT-x/AMD-V dans BIOS Mac (souvent activé par défaut)
- Réduire nombre de cœurs CPU si Mac surchauffe

---

## NEXT STEPS

Une fois OpenFang testé :
1. **Comparer** avec architecture Jill actuelle
2. **Décider** si migration intéressante
3. **Si oui** : Planifier migration progressive
4. **Si non** : Supprimer VM et garder Jill

**Bonne découverte d'OpenFang !** 🚀

---

*Document créé : 5 mars 2026*  
*À tester quand vous voulez explorer OpenFang*
