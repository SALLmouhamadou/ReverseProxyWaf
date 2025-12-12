# WAF ModSecurity CRS - Juice Shop

Configuration d'un Web Application Firewall (WAF) avec OWASP ModSecurity Core Rule Set (CRS) devant l'application vulnérable Juice Shop.

## Architecture

```
┌─────────────┐     ┌─────────────────┐     ┌─────────────┐
│  Navigateur │────▶│  WAF ModSecurity│────▶│  Juice Shop │
│             │     │  (Port 8000/8443)│     │  (Port 3000)│
└─────────────┘     └─────────────────┘     └─────────────┘
```

## Prérequis

- Docker Desktop installé
- Juice Shop en cours d'exécution sur le port 3000 :
  ```bash
  docker run -d -p 3000:3000 --name juice-shop bkimminich/juice-shop
  ```

## Démarrage rapide

1. **Générer les certificats SSL** (optionnel, pour HTTPS) :
   ```bash
   docker run --rm -v "${PWD}:/certs" alpine/openssl req -x509 -nodes \
       -days 365 -newkey rsa:2048 \
       -keyout /certs/server.key -out /certs/server.crt \
       -subj "/C=FR/ST=France/L=Paris/O=Formation/OU=WAF/CN=localhost"
   ```

2. **Démarrer le WAF** :
   ```bash
   docker compose up -d
   ```

3. **Accéder à l'application** :
   - HTTP : http://localhost:8000
   - HTTPS : https://localhost:8443

## URLs

| Service | URL | Description |
|---------|-----|-------------|
| WAF HTTP | http://localhost:8000 | Accès via WAF |
| WAF HTTPS | https://localhost:8443 | Accès sécurisé via WAF |
| Juice Shop Direct | http://localhost:3000 | Sans protection WAF |

## Configuration ModSecurity

| Variable | Valeur | Description |
|----------|--------|-------------|
| `MODSEC_RULE_ENGINE` | `DetectionOnly` | Mode observation (pas de blocage) |
| `PARANOIA` | `1` | Niveau de paranoïa (1-4) |
| `ANOMALY_INBOUND` | `5` | Seuil de blocage des requêtes |
| `ANOMALY_OUTBOUND` | `999` | Seuil pour les réponses |

### Modes du Rule Engine

- **Off** : ModSecurity désactivé
- **DetectionOnly** : Détection et journalisation uniquement
- **On** : Blocage actif des attaques

## Tester le WAF

**Attaque XSS (sera détectée/bloquée selon le mode)** :
```
http://localhost:8000/?search=<script>alert('XSS')</script>
```

**Voir les logs** :
```bash
docker logs waf-modsecurity 2>&1 | findstr "ModSecurity"
```

## Documentation

- [Procedure_modsecurity_rule-engines.pdf](./Procedure_modsecurity_rule-engines.pdf) - Procédure d'activation progressive
- [PROCÉDURE DE MISE EN PLACE DES CERTIFICATS SSL.pdf](./PROCÉDURE%20DE%20MISE%20EN%20PLACE%20DES%20CERTIFICATS%20SSL%20POUR%20WAF%20MODSECURITY.pdf) - Configuration SSL
- [Attaques XSS et Injection SQL.pdf](./Attaques%20XSS%20(stockée%20%26%20reflétée)%20et%20Injection%20SQL.pdf) - Tests d'attaques

## Licence

Projet de formation - Usage éducatif uniquement.
