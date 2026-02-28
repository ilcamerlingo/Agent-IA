# TUTORIEL 1 : Installation Ollama + LightClaw sur Raspberry Pi 5
## Via SSH avec Ghostty (MacBook)

**Objectif :** Installer Ollama (Qwen 2.5 3B) et LightClaw sur le Pi 5  
**Durée estimée :** 45-60 minutes  
**Niveau :** Intermédiaire (suivre les commandes exactement)  
**Connexion :** SSH depuis MacBook avec Ghostty

---

## 📋 PRÉREQUIS

### Sur le Raspberry Pi 5
- [ ] Raspberry Pi OS 64-bit installé (Bookworm recommandé)
- [ ] Connexion Internet active (WiFi ou Ethernet)
- [ ] Refroidissement actif installé (ventilateur ou dissipateur)
- [ ] Alimentation officielle 5V 5A (important !)
- [ ] SSH activé (voir annexe si besoin)

### Sur le MacBook
- [ ] Ghostty installé et fonctionnel
- [ ] Adresse IP du Pi 5 connue (voir méthode ci-dessous)
- [ ] Connexion SSH testée et fonctionnelle

---

## ÉTAPE 0 : OBTENIR L'ADRESSE IP DU PI 5

### Méthode A - Si vous avez un écran sur le Pi
1. Ouvrir un terminal sur le Pi
2. Taper : `hostname -I`
3. Noter l'adresse (ex: `192.168.1.45`)

### Méthode B - Sans écran (headless)
1. Sur votre Mac, ouvrir **Terminal** (ou Ghostty)
2. Taper : `arp -a | grep raspberry`
3. Chercher une ligne comme : `? (192.168.1.45) at b8:27:eb:...`
4. L'adresse IP est entre parenthèses (ex: `192.168.1.45`)

### Méthode C - Via routeur
1. Ouvrir l'interface admin de votre box/routeur
2. Chercher "Appareils connectés"
3. Trouver "raspberrypi" dans la liste
4. Noter l'adresse IP

**Notez l'adresse IP :** `192.168.1.___` (à compléter)

---

## ÉTAPE 1 : CONNEXION SSH AU PI 5

### 1.1 Ouvrir Ghostty sur MacBook

1. Lancer **Ghostty** (Cmd+Espace, taper "Ghostty")
2. Vous voyez une fenêtre terminal prête

### 1.2 Connexion SSH

**Dans Ghostty, taper :**

```bash
ssh pi@192.168.1.45
```

**Remplacer** `192.168.1.45` par l'adresse IP de votre Pi.

### 1.3 Authentification

**Première connexion (si jamais connecté) :**
```
The authenticity of host '192.168.1.45' can't be established.
Are you sure you want to continue connecting (yes/no)?
```

**Taper :** `yes` puis Entrée

**Entrer le mot de passe :**
```
pi@192.168.1.45's password:
```

**Mot de passe par défaut :** `raspberry` (ou votre mot de passe si changé)

**Note :** Quand vous tapez le mot de passe, rien ne s'affiche (c'est normal !)

### 1.4 Vérification de la connexion

**Si vous voyez :**
```
pi@raspberrypi:~ $
```

✅ **Vous êtes connecté au Pi 5 !**

---

## ÉTAPE 2 : MISE À JOUR DU SYSTÈME

**Important :** Toujours mettre à jour avant d'installer

### 2.1 Mettre à jour les paquets

**Dans le terminal SSH (Ghostty), taper :**

```bash
sudo apt update && sudo apt upgrade -y
```

**Cela prend 5-10 minutes.** Attendez la fin.

**Quand vous voyez :**
```
pic@raspberrypi:~ $
```

C'est terminé.

### 2.2 Installer les dépendances nécessaires

**Taper :**

```bash
sudo apt install -y curl wget git
```

**Attendre la fin.**

---

## ÉTAPE 3 : INSTALLATION D'OLLAMA

### 3.1 Télécharger et installer Ollama

**Taper cette commande (copier-coller) :**

```bash
curl -fsSL https://ollama.com/install.sh | sh
```

**Cela prend 2-3 minutes.**

**Si vous voyez :**
```
>>> The Ollama API is now available at 127.0.0.1:11434.
```

✅ **Ollama est installé !**

### 3.2 Vérifier l'installation

**Taper :**

```bash
ollama --version
```

**Vous devez voir :**
```
ollama version 0.x.x
```

### 3.3 Démarrer le service Ollama

**Taper :**

```bash
sudo systemctl start ollama
sudo systemctl enable ollama
```

**Vérifier que le service tourne :**

```bash
sudo systemctl status ollama
```

**Vous devez voir :** `Active: active (running)`

**Pour quitter l'affichage :** taper `q`

---

## ÉTAPE 4 : TÉLÉCHARGEMENT DU MODÈLE QWEN 2.5 3B

### 4.1 Télécharger le modèle

**Taper :**

```bash
ollama pull qwen2.5:3b
```

**⚠️ ATTENTION :** Cela prend **10-20 minutes** selon votre connexion.

**Taille du téléchargement :** ~1.9 GB

**Ne fermez pas le terminal !**

**Quand vous voyez :**
```
success
```

✅ **Le modèle est téléchargé !**

### 4.2 Tester le modèle

**Taper :**

```bash
ollama run qwen2.5:3b
```

**Vous voyez :**
```
>>> Send a message (/? for help)
```

**Tester avec une question simple :**
```
Bonjour, comment ça va ?
```

**Le modèle doit répondre en français.**

**Pour quitter :** taper `/bye` puis Entrée

---

## ÉTAPE 5 : INSTALLATION DE LIGHTCLAW

### 5.1 Installer LightClaw

**Taper cette commande unique :**

```bash
curl -fsSL https://lightclaw.dev/install.sh | bash
```

**Cela prend 1-2 minutes.**

**Quand vous voyez :**
```
LightClaw installed successfully!
```

✅ **LightClaw est installé !**

### 5.2 Vérifier l'installation

**Taper :**

```bash
lightclaw --version
```

**Vous devez voir :**
```
lightclaw x.x.x
```

### 5.3 Créer le dossier de configuration

**Taper :**

```bash
mkdir -p ~/.config/lightclaw
cd ~/.config/lightclaw
```

---

## ÉTAPE 6 : CONFIGURATION INITIALE DE LIGHTCLAW

### 6.1 Configurer LightClaw avec Ollama

**Créer le fichier de configuration :**

```bash
cat > config.yaml << 'EOF'
# Configuration LightClaw pour Raspberry Pi 5
# Jill Munroe - Assistant IA locale

# Modèle LLM
llm:
  provider: ollama
  model: qwen2.5:3b
  host: http://localhost:11434
  timeout: 10  # secondes

# Interface (sera configurée dans le tutoriel 2)
interface:
  type: telegram
  # Token sera ajouté plus tard

# Mémoire
memory:
  enabled: true
  storage: sqlite
  path: ~/.config/lightclaw/memory.db

# Outils disponibles
tools:
  - file
  - shell
  - web
  
# Logging
logging:
  level: info
  path: ~/.config/lightclaw/lightclaw.log
EOF
```

**Vérifier le fichier :**

```bash
cat config.yaml
```

**Vous devez voir votre configuration.**

---

## ÉTAPE 7 : TEST DE LIGHTCLAW AVEC OLLAMA

### 7.1 Lancer LightClaw en mode test

**Taper :**

```bash
lightclaw run --config ~/.config/lightclaw/config.yaml
```

**Si vous voyez des erreurs, vérifier que :**
- Ollama tourne (`sudo systemctl status ollama`)
- Le modèle est bien téléchargé (`ollama list`)

**Pour l'instant, arrêter avec :** Ctrl+C

---

## ÉTAPE 8 : CRÉER UN SERVICE SYSTÈME POUR LIGHTCLAW

### 8.1 Créer le fichier de service

**Taper :**

```bash
sudo tee /etc/systemd/system/lightclaw.service << 'EOF'
[Unit]
Description=LightClaw AI Assistant
After=network.target ollama.service

[Service]
Type=simple
User=pi
WorkingDirectory=/home/pi
ExecStart=/usr/local/bin/lightclaw run --config /home/pi/.config/lightclaw/config.yaml
Restart=always
RestartSec=10

[Install]
WantedBy=multi-user.target
EOF
```

### 8.2 Activer le service

**Taper :**

```bash
sudo systemctl daemon-reload
sudo systemctl enable lightclaw
```

### 8.3 Vérifier le service (ne pas démarrer maintenant)

**Vérifier que le service est bien créé :**

```bash
sudo systemctl status lightclaw
```

**Vous devez voir :** `Loaded: loaded` (mais pas encore active)

**Pour quitter :** taper `q`

---

## ✅ VÉRIFICATION FINALE

### Liste de contrôle

- [ ] SSH fonctionne (Ghostty connecté au Pi)
- [ ] `ollama --version` affiche une version
- [ ] `ollama list` montre `qwen2.5:3b`
- [ ] `lightclaw --version` affiche une version
- [ ] Fichier `~/.config/lightclaw/config.yaml` existe
- [ ] Service `lightclaw` créé (`sudo systemctl status lightclaw`)

### Commandes utiles à retenir

| Action | Commande |
|--------|----------|
| Vérifier Ollama | `sudo systemctl status ollama` |
| Vérifier LightClaw | `sudo systemctl status lightclaw` |
| Lister modèles | `ollama list` |
| Tester modèle | `ollama run qwen2.5:3b` |

---

## 🎯 PROCHAINES ÉTAPES

**Vous avez maintenant :**
- ✅ Ollama installé avec Qwen 2.5 3B
- ✅ LightClaw installé et configuré

**Prochain tutoriel :** Configuration Telegram "Jill Munroe"

**Ne débranchez pas le Pi 5 !**

---

## ANNEXE : ACTIVER SSH SUR PI 5 (si pas déjà fait)

### Si vous avez un écran

1. Menu → Preferences → Raspberry Pi Configuration
2. Onglet "Interfaces"
3. Activer "SSH"
4. Redémarrer

### Si vous n'avez pas d'écran (headless)

1. Éteindre le Pi
2. Retirer la carte SD
3. Insérer dans un lecteur sur Mac
4. Créer un fichier vide nommé `ssh` (sans extension) à la racine
5. Réinsérer la carte SD dans le Pi
6. Démarrer le Pi

---

## DÉPANNAGE

### "Connection refused" quand SSH
- Vérifier que le Pi est allumé (LED verte)
- Vérifier que SSH est activé (voir annexe)
- Vérifier l'adresse IP (`arp -a` sur Mac)

### "Permission denied" quand SSH
- Vérifier le mot de passe (par défaut: `raspberry`)
- Vérifier le username (`pi` par défaut)

### Ollama ne démarre pas
```bash
sudo systemctl restart ollama
sudo systemctl status ollama
```

### Plus d'espace disque
```bash
df -h
```
Si le disque est plein à plus de 90%, supprimer des fichiers ou agrandir la partition.

---

**Tutoriel 1 terminé ! Passer au Tutoriel 2 : Configuration Telegram**

*Date de création : 28 février 2026*  
*Par : Bosley pour Il Camerlingo*
