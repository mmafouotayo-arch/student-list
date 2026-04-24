# Student List – Mini Projet Docker

Application simple de liste d'étudiants avec une API Flask et un frontend PHP/Apache, conteneurisée avec Docker.

---

## Architecture

```
[Navigateur] → [PHP/Apache :8080] → [API Flask :5000] → [student_age.json]
```

Deux services Docker communiquent via un réseau bridge interne (`student-network`) :
- **api** : API REST Flask avec authentification basique
- **website** : Interface web PHP/Apache

---

## Structure du projet

```
.
├── Dockerfile            # Image de l'API Flask
├── docker-compose.yml    # Orchestration des deux services
├── student_age.py        # Code source de l'API
├── student_age.json      # Données des étudiants
├── requirements.txt      # Dépendances Python
└── website/
    └── index.php         # Interface web PHP
```

---

## Dockerfile – Explication

```dockerfile
FROM python:3.11-slim
LABEL maintainer="CCNTechnologies <contact@ccntechnologies.cm>"

RUN apt-get update -y && apt-get install -y python3-dev libsasl2-dev libldap2-dev libssl-dev

COPY student_age.py /student_age.py
COPY requirements.txt /requirements.txt
RUN pip3 install -r /requirements.txt

RUN mkdir -p /data
VOLUME /data

EXPOSE 5000
CMD ["python3", "./student_age.py"]
```

- Image légère `python:3.11-slim`
- Installation des dépendances système et Python
- Le dossier `/data` est déclaré en volume pour la persistance du JSON
- Port 5000 exposé

---

## docker-compose.yml – Explication

```yaml
services:
  api:
    image: student-list-api:latest
    build: .
    volumes:
      - ./student_age.json:/data/student_age.json
    ports:
      - "5000:5000"
    networks:
      - student-network

  website:
    image: php:apache
    environment:
      - USERNAME=toto
      - PASSWORD=python
    volumes:
      - ./website:/var/www/html
    ports:
      - "8080:80"
    depends_on:
      - api
    networks:
      - student-network

networks:
  student-network:
    driver: bridge
```

- Le service `website` dépend du service `api` (démarrage ordonné)
- Les credentials sont injectés via variables d'environnement
- Les deux services partagent le réseau `student-network`

---

## Déploiement

### 1. Préparer le dossier website

```bash
mkdir -p website
cp index.php website/
```

### 2. Construire et démarrer

```bash
docker-compose up -d --build
```

### 3. Tester l'API directement

```bash
curl -u toto:python -X GET http://localhost:5000/pozos/api/v1.0/get_student_ages
```

Réponse attendue :
```json
{
  "student_ages": {
    "alice": "12",
    "bob": "13"
  }
}
```

### 4. Accéder au site web

Ouvrir http://localhost:8080 puis cliquer sur **"List Student"**

---

## Registre Docker privé (optionnel)

```bash
# Lancer un registre privé local
docker run -d -p 5001:5000 --name registry registry:2

# Lancer l'interface web du registre
docker run -d -p 8888:80 \
  -e REGISTRY_URL=http://localhost:5001 \
  joxit/docker-registry-ui:latest

# Tagger et pousser l'image
docker tag student-list-api:latest localhost:5001/student-list-api:latest
docker push localhost:5001/student-list-api:latest
```

---

## Bonnes pratiques appliquées

- Image de base légère (`slim`)
- Nettoyage du cache apt après installation
- Variables d'environnement pour les secrets (pas de credentials codés en dur)
- Volume pour la persistance des données
- Réseau dédié au projet
- `depends_on` pour l'ordre de démarrage des services
