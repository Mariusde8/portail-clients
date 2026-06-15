# 📋 Guide de Déploiement — Portail Clients Prépayé

## 📁 Contenu du dossier `portail_clients/`

| Fichier | Description |
|---|---|
| `index.html` | Page web du portail (login + tableau de bord) |
| `clients_db.json` | Base de données clients (253 133 entrées) |
| `team_users.json` | Liste des agents commerciaux (1 189 entrées) |
| `qrcode_portail.png` | QR code à imprimer et distribuer |

---

## 🚀 Étapes de déploiement

### Option A — GitHub Pages (GRATUIT, recommandé)

1. Créez un compte sur https://github.com
2. Cliquez **"New repository"** → nommez-le `portail-clients`
3. Uploadez **tous les fichiers** du dossier `portail_clients/`
4. Allez dans **Settings → Pages → Source** → choisissez `main` branch
5. Votre URL sera : `https://VOTRE_NOM.github.io/portail-clients/`

### Option B — Hébergeur web (OVH, Hostinger, etc.)

1. Connectez-vous à votre hébergeur via FTP
2. Uploadez **tous les fichiers** dans le dossier `public_html/portail/`
3. Votre URL sera : `https://votre-domaine.com/portail/`

---

## 🔗 Mise à jour du QR Code

Après déploiement, regenerez le QR code avec la vraie URL :

```python
import qrcode
from PIL import Image, ImageDraw, ImageFont

url = 'https://VOTRE_URL_REELLE/'  # ← Remplacez ici

qr = qrcode.QRCode(error_correction=qrcode.constants.ERROR_CORRECT_H, box_size=10, border=4)
qr.add_data(url)
qr.make(fit=True)
img = qr.make_image(fill_color='#0d4f8c', back_color='white').convert('RGB')
W, H = img.size
canvas = Image.new('RGB', (W, H + 80), 'white')
canvas.paste(img, (0, 0, W, H))
draw = ImageDraw.Draw(canvas)
draw.rectangle([(0, H), (W, H+80)], fill='#0d4f8c')
canvas.save('qrcode_final.png')
print('QR code généré !')
```

---

## 🔐 Logique d'authentification

| Situation | Résultat |
|---|---|
| Compteur **introuvable** | ❌ Accès refusé |
| Compteur OK + Téléphone **correct** | ✅ Accès accordé |
| Compteur OK + Téléphone **absent en BD** | 📞 Invitation à enregistrer → accès accordé |
| Compteur OK + Téléphone **incorrect/manquant** | 📞 Invitation à mettre à jour → accès accordé |

---

## 🔄 Mise à jour de la base de données

Pour mettre à jour `clients_db.json` avec un nouveau fichier Excel :

```python
import pandas as pd, json

df = pd.read_excel('Base_de_données_clients_prépayé.xlsx', dtype=str)
df['N°Compteur'] = df['N°Compteur'].fillna('').str.strip()
df['Téléphone'] = df['Téléphone'].fillna('').str.strip()

data = {row['N°Compteur']: row['Téléphone'] 
        for _, row in df.iterrows() if row['N°Compteur']}

with open('portail_clients/clients_db.json', 'w', encoding='utf-8') as f:
    json.dump(data, f, ensure_ascii=False)
print(f'{len(data)} entrées exportées')
```

---

## ⚠️ Note importante

La mise à jour du numéro de téléphone par le client est **en mémoire locale** (pendant la session).
Pour **persister ces mises à jour** dans la vraie base de données, il faudra ajouter un backend (ex: Firebase, Supabase) ou prévoir une procédure manuelle de collecte.
