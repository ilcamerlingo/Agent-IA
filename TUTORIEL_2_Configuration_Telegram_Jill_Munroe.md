# TUTORIEL 2 : Configuration Telegram "Jill Munroe" avec LightClaw
## Via SSH avec Ghostty (MacBook)

**Objectif :** Créer le bot Telegram "Jill Munroe" et le connecter à LightClaw  
**Durée estimée :** 30-40 minutes  
**Niveau :** Intermédiaire  
**Prérequis :** Tutoriel 1 terminé (Ollama + LightClaw installés)

---

## 📋 PRÉREQUIS

### Déjà fait (Tutoriel 1)
- [ ] Ollama installé et fonctionnel
- [ ] LightClaw installé
- [ ] Modèle Qwen 2.5 3B téléchargé
- [ ] Connexion SSH active (Ghostty)

### Pour ce tutoriel
- [ ] Smartphone avec Telegram installé
- [ ] Compte Telegram actif
- [ ] Connexion Internet stable

---

## ÉTAPE 1 : CRÉER LE BOT TELEGRAM "JILL MUNROE"

### 1.1 Ouvrir Telegram sur votre smartphone

1. Lancer l'app **Telegram** sur iPhone/Android
2. Vous devez être connecté à votre compte

### 1.2 Trouver BotFather

1. Dans la barre de recherche Telegram, taper : `@BotFather`
2. Cliquer sur **BotFather** (vérifier la coche bleue ✅)
3. Cliquer **START** ou envoyer `/start`

### 1.3 Créer un nouveau bot

1. Envoyer la commande : `/newbot`
2. BotFather demande : **"Alright, a new bot. How are we going to call it?"**
3. **Répondre :** `Jill Munroe`

**Attention :** Le nom d'affichage peut contenir des espaces.

### 1.4 Choisir le username

BotFather demande : **"Good. Now let's choose a username for your bot."**

**Répondre :** `jillmunroe_bot`

**RÈGLES :**
- Doit se terminer par `_bot` ou `bot`
- Pas d'espaces
- Pas de majuscules
- Doit être unique

**Si déjà pris, essayer :**
- `jillmunroe38_bot`
- `jill_munroe_bot`
- `jillmunroeai_bot`

### 1.5 Récupérer le token

**Si tout est OK, BotFather répond :**
```
Done! Congratulations on your new bot. You will find it at t.me/jillmunroe_bot

Use this token to access the HTTP API:
123456789:ABCDEFghijklmnopQRSTuvwxyz12345
```

**COPIER LE TOKEN** (la longue chaîne de caractères après "Use this token...")

**Notez-le précieusement !** Format : `123456789:ABC...xyz`

**Exemple de token :**
```
123456789:AAHdGfZP0a4jRk9Xc8vB7nM6lK5jH4gF3eD2cB1a
```

**⚠️ IMPORTANT :** Ce token est SECRET. Ne le partagez avec personne.

---

## ÉTAPE 2 : CONFIGURER LIGHTCLAW AVEC LE TOKEN

### 2.1 Retourner sur Ghostty (SSH sur Pi)

**Vous devez être connecté en SSH au Pi 5.**

**Vérifier :** Le prompt doit afficher `pi@raspberrypi:~ $`

### 2.2 Éditer le fichier de configuration

**Taper :**

```bash
cd ~/.config/lightclaw
```

**Ouvrir le fichier avec nano :**

```bash
nano config.yaml
```

**Vous voyez l'éditeur nano avec votre config précédente.**

### 2.3 Modifier la section interface

**Utiliser les flèches pour naviguer** jusqu'à la section :

```yaml
interface:
  type: telegram
  # Token sera ajouté plus tard
```

**Remplacer par :**

```yaml
interface:
  type: telegram
  telegram:
    token: "VOTRE_TOKEN_ICI"
    allowed_users: []
    # Laisser allowed_users vide pour autoriser tout le monde
    # Ou mettre votre ID Telegram pour restreindre
```

**Remplacer** `VOTRE_TOKEN_ICI` par le token copié à l'étape 1.5

**Exemple complet :**

```yaml
# Configuration LightClaw pour Raspberry Pi 5
# Jill Munroe - Assistant IA locale

# Modèle LLM
llm:
  provider: ollama
  model: qwen2.5:3b
  host: http://localhost:11434
  timeout: 10  # secondes

# Interface Telegram
interface:
  type: telegram
  telegram:
    token: "123456789:AAHdGfZP0a4jRk9Xc8vB7nM6lK5jH4gF3eD2cB1a"
    allowed_users: []

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
```

### 2.4 Sauvegarder et quitter nano

**Pour sauvegarder :**
1. Appuyer sur **Ctrl+O** (lettre O)
2. Confirmer le nom : **Entrée**
3. Message : `[ Wrote 25 lines ]`

**Pour quitter :**
1. Appuyer sur **Ctrl+X**
2. Vous êtes revenu au terminal

### 2.5 Vérifier la configuration

**Taper :**

```bash
cat config.yaml
```

**Vérifier que :**
- Le token est bien présent
- Il n'y a pas d'erreurs de syntaxe
- Le fichier est complet

---

## ÉTAPE 3 : DÉMARRER LIGHTCLAW POUR LA PREMIÈRE FOIS

### 3.1 Lancer le service LightClaw

**Taper :**

```bash
sudo systemctl start lightclaw
```

### 3.2 Vérifier que le service tourne

**Taper :**

```bash
sudo systemctl status lightclaw
```

**Vous devez voir :**
```
● lightclaw.service - LightClaw AI Assistant
     Loaded: loaded (/etc/systemd/system/lightclaw.service; enabled)
     Active: active (running) since ...
```

**Si vous voyez `Active: active (running)` :** ✅ C'est bon !

**Pour quitter l'affichage :** taper `q`

### 3.3 Vérifier les logs

**Taper :**

```bash
sudo journalctl -u lightclaw -f
```

**Vous voyez les logs en temps réel.**

**Chercher des messages comme :**
```
Connected to Ollama
Telegram bot started
Waiting for messages...
```

**Pour quitter les logs :** Ctrl+C

---

## ÉTAPE 4 : TESTER JILL MUNROE SUR TELEGRAM

### 4.1 Trouver le bot sur Telegram

1. Ouvrir **Telegram** sur smartphone
2. Dans la barre de recherche, taper : `jillmunroe_bot`
3. Cliquer sur le bot (vérifier que c'est le vôtre)
4. Cliquer **START**

### 4.2 Envoyer un premier message

**Envoyer :**
```
Bonjour Jill, tu fonctionnes ?
```

**Attendre 3-5 secondes.**

### 4.3 Vérifier la réponse

**Si Jill répond (en français) :**
```
Bonjour ! Oui, je fonctionne parfaitement. Comment puis-je vous aider ?
```

🎉 **SUCCESS ! Jill Munroe est opérationnelle !**

**Si pas de réponse après 10 secondes :**
→ Voir section "Dépannage"

---

## ÉTAPE 5 : CONFIGURER LES PERMISSIONS (OPTIONNEL)

### 5.1 Trouver votre ID Telegram (pour sécuriser)

**Pour restreindre l'accès à vous seul :**

1. Sur Telegram, chercher le bot : `@userinfobot`
2. Envoyer `/start`
3. Le bot répond avec votre ID : `Your user ID: 123456789`

**Noter cet ID** (ex: `123456789`)

### 5.2 Modifier la config pour restreindre

**Dans Ghostty (SSH) :**

```bash
nano ~/.config/lightclaw/config.yaml
```

**Modifier la section allowed_users :**

```yaml
interface:
  type: telegram
  telegram:
    token: "VOTRE_TOKEN"
    allowed_users:
      - 123456789  # Remplacer par VOTRE ID
```

**Sauvegarder :** Ctrl+O, Entrée, Ctrl+X

### 5.3 Redémarrer LightClaw

```bash
sudo systemctl restart lightclaw
```

**Maintenant, seul vous pouvez parler à Jill !**

---

## ✅ VÉRIFICATION FINALE

### Tests à effectuer

- [ ] Bot "Jill Munroe" visible sur Telegram
- [ ] Envoyer message → Réponse en < 5 secondes
- [ ] Réponse en français correct
- [ ] Service `lightclaw` actif (`sudo systemctl status lightclaw`)
- [ ] Logs sans erreur (`sudo journalctl -u lightclaw`)

### Commandes utiles

| Action | Commande |
|--------|----------|
| Voir si LightClaw tourne | `sudo systemctl status lightclaw` |
| Redémarrer LightClaw | `sudo systemctl restart lightclaw` |
| Arrêter LightClaw | `sudo systemctl stop lightclaw` |
| Voir les logs | `sudo journalctl -u lightclaw -f` |

---

## 🎯 RÉSULTAT

**Vous avez maintenant :**
- ✅ Bot Telegram "Jill Munroe" créé
- ✅ Connecté à LightClaw sur le Pi 5
- ✅ Fonctionne avec Qwen 2.5 3B (local, gratuit)

**Prochain tutoriel :** Intégration avec OpenClaw (mécanisme de fallback)

---

## DÉPANNAGE

### "Bot not responding" sur Telegram

**Vérifier que LightClaw tourne :**
```bash
sudo systemctl status lightclaw
```

**Si inactif :**
```bash
sudo systemctl start lightclaw
```

**Voir les erreurs :**
```bash
sudo journalctl -u lightclaw -n 50
```

### "Token invalid" dans les logs

- Vérifier que le token est correct dans `config.yaml`
- Vérifier qu'il n'y a pas d'espaces autour du token
- Regénérer un token via BotFather si besoin (`/revoke`)

### "Cannot connect to Ollama"

**Vérifier Ollama :**
```bash
sudo systemctl status ollama
```

**Si arrêté :**
```bash
sudo systemctl start ollama
```

**Tester Ollama manuellement :**
```bash
ollama run qwen2.5:3b
```

### Réponses très lentes (> 10 secondes)

- Normal sur Pi 5 (CPU, pas GPU)
- Vérifier la température : `vcgencmd measure_temp`
- Si > 70°C, améliorer le refroidissement

### Erreur "Permission denied" dans les logs

**Vérifier les permissions :**
```bash
ls -la ~/.config/lightclaw/
```

**Si besoin :**
```bash
sudo chown -R pi:pi ~/.config/lightclaw/
```

---

**Tutoriel 2 terminé ! Passer au Tutoriel 3 : Intégration OpenClaw**

*Date de création : 28 février 2026*  
*Par : Bosley pour Il Camerlingo*
