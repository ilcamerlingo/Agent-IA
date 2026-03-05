# TUTORIEL 3 : Intégration OpenClaw - LightClaw (Mécanisme de Fallback) Qwen3.5


## Architecture "Jill Munroe" - Économie de tokens Moonshot

**Objectif :** Configurer OpenClaw pour interroger d'abord Jill Munroe (LightClaw local via API Ollama), puis fallback sur Moonshot si nécessaire.  
**Durée estimée :** 45-60 minutes  
**Niveau :** Avancé (configuration de système)  
**Économie visée :** ~85-90% de réduction des coûts Moonshot  

---

## 📋 ARCHITECTURE COMPLÈTE

```text
Vous (Telegram @ilcamerlingo)
│
▼
┌──────────────────────────────────────────────────────────────┐
│ OpenClaw (Bosley)                                            │
│  ┌────────────────────────────────────────────────────────┐  │
│  │ 1. Reçoit votre question                               │  │
│  │ 2. Envoie à Jill Munroe (LightClaw sur Pi 5)           │  │
│  │    via API HTTP locale (http://192.168.1.45:8080)      │  │
│  │ 3. Attend réponse (timeout: 5 secondes)                │  │
│  │                                                        │  │
│  │ SI réponse reçue ET qualité OK (< 5s):                 │  │
│  │ → Affiche réponse Jill ($0)                            │  │
│  │ → Log: "Réponse locale - Économie: $0.005"             │  │
│  │                                                        │  │
│  │ SINON (timeout ou erreur):                             │  │
│  │ → Fallback Moonshot API ($0.001-0.01)                  │  │
│  │ → Log: "Fallback API - Coût: $X.XXX"                   │  │
│  └────────────────────────────────────────────────────────┘  │
└──────────────────────────────────────────────────────────────┘
│
├───────────────────┬───────────────────┬──────────────────────┐
▼                   ▼                   ▼
┌─────────────┐     ┌──────────────┐    ┌──────────────┐
│ Jill        │     │ Jill         │    │ Moonshot     │
│ (LightClaw) │     │ (Timeout)    │    │ (Fallback)   │
│ Local       │     │              │    │ Cloud        │
│ $0          │     │ ↓↓↓          │    │ $$$          │
└─────────────┘     └──────────────┘    └──────────────┘
85%                 5%                  10%
```

---

## ⚠️ AVANT DE COMMENCER

### Prérequis techniques
1. **Pi 5 opérationnel** avec Ollama installé et fonctionnel.
2. **Modèle chargé** : Assurez-vous que le modèle est présent :
   ```bash
   ollama pull qwen2.5:3b
   ```
3. **Dépendances Python** :
   ```bash
   sudo apt update
   sudo apt install -y python3-pip python3-requests jq
   ```
4. **Ollama accessible** : Le service Ollama doit tourner sur le port `11434`.

### Vérifications obligatoires
**Sur le Pi 5 (via SSH) :**
1. **Ollama tourne ?**
   ```bash
   systemctl status ollama
   ```
   Doit afficher `Active: active (running)`.
2. **Adresse IP du Pi connue ?**
   ```bash
   hostname -I
   ```
   Noter l'adresse (ex: `192.168.1.45`).

---

## ÉTAPE 1 : CRÉER UN SERVEUR API POUR LIGHTCLAW

LightClaw en mode Telegram ne peut pas être interrogé directement par OpenClaw de manière optimale. Nous allons créer une API HTTP légère qui parle directement à Ollama.

### 1.1 Se connecter au Pi 5 en SSH
**Dans votre terminal :**
```bash
ssh pi@192.168.1.45 # Remplacer par l'IP de votre Pi
```

### 1.2 Créer le script serveur API optimisé
**Créer le dossier si besoin :**
```bash
mkdir -p ~/.config/lightclaw
cd ~/.config/lightclaw
```

**Créer le fichier `jill_api.py` :**
```bash
cat > jill_api.py << 'EOF'
#!/usr/bin/env python3
"""
API Bridge pour Jill Munroe (LightClaw)
Permet à OpenClaw d'interroger Ollama via HTTP sans surcharge CLI
"""
import requests
import json
from http.server import HTTPServer, BaseHTTPRequestHandler
from urllib.parse import urlparse

HOST = "0.0.0.0"
PORT = 8080
OLLAMA_URL = "http://localhost:11434/api/generate"
MODEL_NAME = "qwen2.5:3b"

class JillHandler(BaseHTTPRequestHandler):
    def log_message(self, format, *args):
        # Logging minimal pour ne pas saturer
        pass

    def send_json(self, status_code, data):
        self.send_response(status_code)
        self.send_header('Content-type', 'application/json')
        self.send_header('Access-Control-Allow-Origin', '*')
        self.end_headers()
        self.wfile.write(json.dumps(data).encode())

    def do_GET(self):
        """Endpoint de test (Healthcheck)"""
        self.send_json(200, {
            "status": "ok",
            "service": "Jill Munroe API",
            "version": "1.1-Optimized"
        })

    def do_POST(self):
        """Endpoint pour poser une question à Ollama"""
        try:
            content_length = int(self.headers.get('Content-Length', 0))
            post_data = self.rfile.read(content_length)
            data = json.loads(post_data.decode('utf-8'))
            question = data.get('question', '')

            if not question:
                self.send_json(400, {"status": "error", "error": "Missing 'question' field"})
                return

            # Appel direct à l'API Ollama (Plus rapide que CLI)
            response_ollama = requests.post(
                OLLAMA_URL,
                json={
                    "model": MODEL_NAME,
                    "prompt": question,
                    "stream": False
                },
                timeout=8  # Timeout strict
            )
            response_ollama.raise_for_status()
            result = response_ollama.json()
            answer = result.get('response', '')

            self.send_json(200, {
                "status": "success",
                "question": question,
                "answer": answer.strip(),
                "source": "local",
                "cost": 0.0
            })

        except requests.exceptions.Timeout:
            self.send_json(504, {
                "status": "timeout",
                "error": "Ollama took too long (> 8s)"
            })
        except Exception as e:
            self.send_json(500, {
                "status": "error",
                "error": str(e)
            })

if __name__ == "__main__":
    server = HTTPServer((HOST, PORT), JillHandler)
    print(f"Jill API Server running on http://{HOST}:{PORT}")
    try:
        server.serve_forever()
    except KeyboardInterrupt:
        print("\nShutting down...")
        server.shutdown()
EOF
```

### 1.3 Rendre le script exécutable
```bash
chmod +x jill_api.py
```

---

## ÉTAPE 2 : CRÉER UN SERVICE SYSTÈME (SYSTEMD)

Pour que l'API redémarre automatiquement avec le Pi.

### 2.1 Créer le fichier de service
```bash
sudo tee /etc/systemd/system/jill-api.service << 'EOF'
[Unit]
Description=Jill Munroe API Bridge (Ollama HTTP)
After=network.target ollama.service
Wants=ollama.service

[Service]
Type=simple
User=pi
WorkingDirectory=/home/pi/.config/lightclaw
ExecStart=/usr/bin/python3 /home/pi/.config/lightclaw/jill_api.py
Restart=always
RestartSec=5
Environment="PYTHONUNBUFFERED=1"

[Install]
WantedBy=multi-user.target
EOF
```

### 2.2 Activer et démarrer le service
```bash
sudo systemctl daemon-reload
sudo systemctl enable jill-api
sudo systemctl start jill-api
```

### 2.3 Vérifier l'état
```bash
sudo systemctl status jill-api
```
**Doit afficher :** `Active: active (running)`

### 2.4 Tester l'API
**Depuis un autre terminal (ou votre Mac) :**
```bash
curl -X POST http://192.168.1.45:8080 \
  -H "Content-Type: application/json" \
  -d '{"question": "Bonjour"}'
```

**Réponse attendue (JSON propre) :**
```json
{
  "status": "success",
  "answer": "Bonjour ! Comment puis-je vous aider ?",
  "source": "local"
}
```

---

## ÉTAPE 3 : CONFIGURER OPENCLAW (SCRIPT BRIDGE)

Ce script sera appelé par OpenClaw. Il doit être robuste et gérer le JSON correctement.

### 3.1 Installer l'outil JSON `jq`
Indispensable pour parser la réponse sans erreur.
```bash
sudo apt install -y jq
```

### 3.2 Créer le script `ask_jill.sh`
**Toujours dans `~/.config/lightclaw` :**
```bash
cat > ask_jill.sh << 'EOF'
#!/bin/bash
# Script bridge pour OpenClaw -> Jill Munroe
# Retourne la réponse en stdout et le statut en code de sortie

QUESTION="$1"
JILL_API="http://192.168.1.45:8080" # ⚠️ REMPLACER PAR L'IP DU PI
TIMEOUT=5

if [ -z "$QUESTION" ]; then
  echo "Erreur: Aucune question fournie" >&2
  exit 1
fi

# Appel API avec timeout strict
RESPONSE=$(curl -s -X POST "$JILL_API" \
  -H "Content-Type: application/json" \
  -d "{\"question\": \"$QUESTION\"}" \
  --max-time $TIMEOUT 2>/dev/null)

# Vérification du code de retour curl
if [ $? -ne 0 ] || [ -z "$RESPONSE" ]; then
  echo "Jill unavailable (curl error)" >&2
  exit 1
fi

# Parsing JSON robuste avec jq
STATUS=$(echo "$RESPONSE" | jq -r '.status')
ANSWER=$(echo "$RESPONSE" | jq -r '.answer')

if [ "$STATUS" == "success" ] && [ "$ANSWER" != "null" ]; then
  echo "$ANSWER"
  echo "[SOURCE: Jill Munroe (local) - Économie: $0.00]" >&2
  exit 0
else
  echo "Jill unavailable (status: $STATUS)" >&2
  exit 1
fi
EOF
```

### 3.3 Rendre exécutable et tester
```bash
chmod +x ask_jill.sh
./ask_jill.sh "Quelle est la capitale de la France ?"
```
**Doit afficher la réponse suivie du message de succès.**

---

## ÉTAPE 4 : DOCUMENTATION POUR OPENCLAW (MEMORY)

Créez un fichier `JILL_INTEGRATION.md` dans votre dossier projet pour que l'IA s'en souvienne.

```markdown
# Intégration Jill Munroe - Configuration OpenClaw

## Architecture
- **Local :** Pi 5 (Ollama + Qwen2.5:3b) via API HTTP port 8080.
- **Fallback :** Moonshot API si timeout > 5s ou erreur.

## Procédure
1. Toujours tenter le script local : `~/.config/lightclaw/ask_jill.sh "QUESTION"`
2. Si `exit code 0` → Utiliser la réponse.
3. Si `exit code 1` → Basculer sur Moonshot.

## Sécurité
- API ouverte sur le réseau local uniquement.
- Ne pas exposer le port 8080 sur Internet sans protection.
```

---

## ÉTAPE 5 : TEST COMPLÈT & MAINTENANCE

### 5.1 Test de charge rapide
Vérifiez que le Pi ne chauffe pas trop lors de plusieurs requêtes :
```bash
vcgencmd measure_temp
```

### 5.2 Logs en temps réel
Pour déboguer si OpenClaw ne reçoit pas la réponse :
```bash
sudo journalctl -u jill-api -f
```

### 5.3 Redémarrage automatique (Sécurité)
Pour éviter les fuites de mémoire sur le long terme, un redémarrage hebdomadaire est conseillé.

**Éditer la cron :**
```bash
sudo crontab -e
```

**Ajouter (Redémarrage le dimanche à 3h du matin) :**
```bash
0 3 * * 0 /sbin/reboot
```

---

## ✅ CHECKLIST FINALE

- [ ] **Dépendances** : `python3-requests` et `jq` installés.
- [ ] **Modèle** : `qwen2.5:3b` téléchargé (`ollama pull`).
- [ ] **Service** : `jill-api` est `active (running)`.
- [ ] **Test API** : Le `curl` POST renvoie un JSON valide.
- [ ] **Test Script** : `ask_jill.sh` renvoie la réponse et code 0.
- [ ] **IP** : L'adresse IP du Pi est fixe (DHCP static ou IP statique) pour ne pas changer au redémarrage.

---

## 💰 CALCUL DES ÉCONOMIES (Mise à jour)

| Source | Questions | Coût/unité | Coût Total |
| :--- | :--- | :--- | :--- |
| **Avant (Moonshot 100%)** | 100 | $0.012 | **$1.20** |
| **Après (Jill 85%)** | 85 | $0.00 | $0.00 |
| **Après (Fallback 15%)** | 15 | $0.012 | $0.18 |
| **Total Après** | 100 | - | **$0.18** |

**Économie : 85%**  
**ROI Matériel :** Rentabilisé en ~1 mois d'utilisation intensive.

---

## 🎯 RÉSULTAT FINAL

**Vous avez maintenant :**
- ✅ **Jill Munroe** optimisée (API HTTP Ollama)
- ✅ **Zéro latence CLI** (Plus besoin d'ouvrir un processus à chaque fois)
- ✅ **Parsing JSON robuste** (Grâce à `jq`)
- ✅ **Fallback sécurisé**

**Coût total infrastructure :**
- Pi 5 : Déjà acquis
- Électricité : ~2-3€/mois
- **Économie tokens : ~92€/mois**

---

## 📞 EN CAS DE PROBLÈME

1. **Erreur 500 dans l'API** : Vérifiez qu'Ollama tourne (`systemctl status ollama`).
2. **Erreur de connexion** : Vérifiez que l'IP du Pi n'a pas changé.
3. **Timeout trop court** : Augmentez `timeout=8` dans `jill_api.py` si le Pi est lent.

**🎉 FÉLICITATIONS ! Votre architecture est maintenant robuste et optimisée.**

*Date de mise à jour du tutoriel : Février 2024*  
*Par : Bosley (Corrigé & Optimisé)*
