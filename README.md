# FicheBoost — Guide d'implémentation

## ✅ ÉTAPE 4 — COMPLÉTÉE ✅
Mentions légales mises à jour dans `index.html`
- Éditeur : Loris Boisson
- Adresse : 14 rue Saint-Jean, 75017 Paris
- SIRET : 12345678901234 (à confirmer avec ton numéro réel)
- Médiation : CM2C — cm2c.net

## Site améloré
- ✓ Slider d'exemples (4 rapports) avec boutons de navigation
- ✓ Fiche exemple droite (pas décalée)
- ✓ "Propulsé par Kaelix Voice" au footer
- ✓ Eyebrow IA et garanties de remboursement supprimées

---

## ÉTAPE 5 — Mettre en ligne sur Vercel (20 min)

### Instructions
1. Ouvre un terminal dans le dossier `ficheboost`
2. Tape : `vercel`
3. Suis les prompts :
   - Link to existing project? → Non
   - Project name? → `ficheboost`
   - Production branch? → `main` (ou laisse par défaut)
4. Une fois déployé, tu obtiendras une URL du style `ficheboost.vercel.app`
5. Dans le dashboard Vercel (vercel.com) :
   - Settings → Domains
   - Ajoute `ficheboost.fr` (ton domaine acheté)
   - Suis les instructions DNS chez Ionos

### DNS Ionos
Ajoute chez ton registraire :
- **Type A** : pointe vers l'IP fournie par Vercel
- **CNAME** : `www` pointe vers `ficheboost.vercel.app`

Propagation : 10 min à 2h

---

## ÉTAPE 6 — Construire le pipeline d'analyse (3-4h)

### Structure du projet
```
ficheboost-engine/
├── main.py                 # Script principal
├── requirements.txt        # Dépendances
├── template_report.html    # Template PDF (optionnel)
└── .env                    # Variables d'env (contact@kaelix.store credentials)
```

### Flux du pipeline
1. **Récupération Google Places API**
   ```
   Input: Place ID ou nom + lieu
   Output: nom, note, avis, photos, horaires, catégories, description
   ```

2. **Analyse Claude (Anthropic API)**
   ```
   Input: Données de la fiche
   Output: JSON {
     score: 0-100,
     flags: [{category, severity, text}, ...],
     actions: [{title, steps}, ...],
     ...
   }
   ```

3. **Génération PDF avec ReportLab**
   - Page 1 : Couverture + jauge
   - Pages 2-4 : Analyses par section
   - Page 5 : Plan d'action
   - Page 6 : Footer + infos FicheBoost

4. **Envoi email via Ionos SMTP**
   ```
   From: contact@kaelix.store
   To: [email du client]
   Attachement: rapport.pdf
   ```

### Dépendances Python
```
google-api-python-client
anthropic
reportlab
python-dotenv
flask (pour l'API webhook à l'étape 7)
```

### Test en local
Teste sur 3 vraies fiches avant branchement Stripe :
1. Ta propre fiche (Bayonne)
2. Un commerce local connu
3. Un type d'établissement différent

---

## ÉTAPE 7 — Brancher le webhook Stripe (45 min)

### Option recommandée : Railway (5€/mois)

#### Sur Railway
1. Crée un compte sur railway.app
2. Crée un nouveau projet "Python"
3. Connecte ton repo GitHub (ou upload `ficheboost-engine/`)
4. Ajoute les variables d'env :
   - `GOOGLE_API_KEY`
   - `ANTHROPIC_API_KEY`
   - `IONOS_SMTP_USER`
   - `IONOS_SMTP_PASS`
   - `STRIPE_WEBHOOK_SECRET`

5. Railway te donne une URL publique du style `railway.dev/...`

#### Sur Stripe
1. Developers → Webhooks → Add endpoint
2. URL: `https://[ton-url-railway]/webhook`
3. Sélectionne l'event : `checkout.session.completed`
4. Copie le **Signing Secret** → variable d'env `STRIPE_WEBHOOK_SECRET`

#### Code du webhook (Python Flask)
```python
@app.route('/webhook', methods=['POST'])
def webhook():
    payload = request.get_data()
    sig_header = request.headers.get('Stripe-Signature')
    
    try:
        event = stripe.Webhook.construct_event(
            payload, sig_header, os.getenv('STRIPE_WEBHOOK_SECRET')
        )
    except Exception as e:
        return {'error': str(e)}, 400
    
    if event['type'] == 'checkout.session.completed':
        session = event['data']['object']
        email = session.get('customer_details', {}).get('email')
        metadata = session.get('metadata', {})
        fiche_id = metadata.get('fiche_id')  # Tu dois passer ça depuis le formulaire
        
        # Lancer le pipeline
        run_audit(fiche_id, email)
        
    return {'status': 'success'}, 200
```

---

## Configuration actuelle

### Stripe
- URL de paiement : `https://buy.stripe.com/5kQ6oH0zE31Ugw4dN1dfG05`
- Price: 19€ TTC
- Webhook à configurer à l'étape 7

### Email
- Email de contact : `contact@kaelix.store`
- SMTP : Ionos (config à passer au script Python)

### Google Places API
- Console : console.cloud.google.com
- Carte bancaire requise (free tier : ~200$ de crédits)
- Endpoints : Place Details

### Anthropic API
- claude-3-5-sonnet ou claude-3-opus
- À configurer dans le script Python

---

## Notes importantes

1. **SIRET** : Remplace `12345678901234` par ton vrai numéro
2. **Prompt Claude** : C'est ton seul vrai actif technique. À peaufiner avant de passer au webhook.
3. **Test en local** : Teste les 3 rapports et valide la qualité avant Vercel
4. **Propagation DNS** : Attend 2h complètement avant de te soucier si le domaine ne pointe pas

---

## Prochaines étapes immédiates

1. ✓ Ouvre le dossier `ficheboost` dans VS Code via le fichier `.code-workspace`
2. → Étape 5 : Vercel (20 min)
3. → Étape 6 : Pipeline Python (3-4h)
4. → Étape 7 : Webhook Stripe (45 min)

Bon courage ! 🚀
