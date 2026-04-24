# Image de base
FROM python:3.11-slim

# Maintainer
LABEL maintainer="CCNTechnologies <contact@ccntechnologies.cm>"

# Installation des dépendances système
RUN apt-get update -y && \
    apt-get install -y \
        gcc \
        python3-dev \
        libsasl2-dev \
        libldap2-dev \
        libssl-dev && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*

# Copie du code source de l'API à la racine
COPY student_age.py /student_age.py

# Copie et installation des dépendances Python
COPY requirements.txt /requirements.txt
RUN pip3 install -r /requirements.txt

# Création du dossier de données persistantes et déclaration en volume
RUN mkdir -p /data
VOLUME /data

# Exposition du port de l'API
EXPOSE 5000

# Commande de démarrage
CMD ["python3", "./student_age.py"]
