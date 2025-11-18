# EdgeMap - POC Node-RED

> Proof of Concept pour la cartographie automatisée des infrastructures réseau industrielles CLAUGER

## 📋 Vue d'ensemble

EdgeMap est une application de cartographie réseau développée en Node-RED pour documenter et visualiser automatiquement l'infrastructure réseau des sites industriels CLAUGER. Ce POC valide l'approche full JavaScript avec Node-RED et MongoDB.

### Objectifs du POC

- Cartographie automatisée des équipements réseau (switches, routeurs, firewalls)
- Gestion de la hiérarchie organisationnelle (sites, bâtiments, zones)
- Documentation des équipements industriels et protocoles
- Interface de visualisation et d'export des données
- Validation de l'architecture Node-RED + MongoDB

## 🏗️ Architecture

```
┌─────────────────────────────────────────┐
│         Node-RED Application            │
│  ┌───────────────────────────────────┐  │
│  │     Flows & Dashboard UI          │  │
│  ├───────────────────────────────────┤  │
│  │   Business Logic (Functions)      │  │
│  ├───────────────────────────────────┤  │
│  │   MongoDB Integration Nodes       │  │
│  └───────────────────────────────────┘  │
└─────────────────┬───────────────────────┘
                  │
                  ▼
        ┌─────────────────────┐
        │   MongoDB Container │
        │                     │
        │  - Organizations    │
        │  - Sites            │
        │  - Buildings        │
        │  - NetworkEquipment │
        │  - IndustrialDevices│
        └─────────────────────┘
```

## 🚀 Démarrage rapide

### Prérequis

- Docker Desktop installé et démarré
- Git (pour cloner le projet)
- Port 1880 (Node-RED) et 27017 (MongoDB) disponibles

### Installation

1. **Cloner le projet**
```bash
git clone <repository-url>
cd edgemap-nodered-poc
```

2. **Créer le fichier docker-compose.yml**
```yaml
version: '3.8'

services:
  nodered:
    image: nodered/node-red:latest
    container_name: edgemap-nodered
    ports:
      - "1880:1880"
    volumes:
      - nodered-data:/data
    environment:
      - TZ=Europe/Paris
    depends_on:
      - mongodb
    restart: unless-stopped

  mongodb:
    image: mongo:latest
    container_name: edgemap-mongodb
    ports:
      - "27017:27017"
    volumes:
      - mongodb-data:/data/db
    environment:
      - MONGO_INITDB_DATABASE=edgemap
    restart: unless-stopped

volumes:
  nodered-data:
  mongodb-data:
```

3. **Démarrer les conteneurs**
```bash
docker-compose up -d
```

4. **Vérifier le démarrage**
```bash
docker-compose ps
```

5. **Accéder à Node-RED**
- URL: http://localhost:1880
- Attendre que l'interface soit chargée (~30 secondes au premier démarrage)

### Configuration initiale Node-RED

1. **Installer les nodes MongoDB**
   - Menu (☰) → Manage palette → Install
   - Rechercher et installer: `node-red-node-mongodb`

2. **Configurer la connexion MongoDB**
   - URI: `mongodb://mongodb:27017/edgemap`
   - Database: `edgemap`

## 📊 Modèle de données

### Collections MongoDB

#### Organizations
```javascript
{
  _id: ObjectId,
  name: String,           // "CLAUGER"
  code: String,           // "CLGR"
  description: String,
  createdAt: Date,
  updatedAt: Date
}
```

#### Sites
```javascript
{
  _id: ObjectId,
  organizationId: ObjectId,
  name: String,           // "Site Bordeaux"
  code: String,           // "BDX"
  address: {
    street: String,
    city: String,
    postalCode: String,
    country: String
  },
  coordinates: {
    latitude: Number,
    longitude: Number
  },
  contact: {
    name: String,
    email: String,
    phone: String
  },
  createdAt: Date,
  updatedAt: Date
}
```

#### Buildings
```javascript
{
  _id: ObjectId,
  siteId: ObjectId,
  name: String,           // "Atelier de production"
  code: String,           // "PROD-A"
  floor: String,          // "RDC", "R+1"
  description: String,
  createdAt: Date,
  updatedAt: Date
}
```

#### NetworkEquipment
```javascript
{
  _id: ObjectId,
  buildingId: ObjectId,
  type: String,           // "switch", "router", "firewall", "wifi-ap"
  manufacturer: String,   // "Cisco", "HPE", "Fortinet"
  model: String,
  hostname: String,
  ipAddress: String,
  macAddress: String,
  serialNumber: String,
  firmwareVersion: String,
  location: String,       // Description physique
  vlan: {
    management: Number,
    data: [Number],
    voice: Number,
    industrial: [Number]
  },
  ports: {
    total: Number,
    used: Number,
    speed: String       // "1Gbps", "10Gbps"
  },
  uplinks: [ObjectId],   // Références vers équipements parents
  status: String,         // "active", "inactive", "maintenance"
  notes: String,
  createdAt: Date,
  updatedAt: Date
}
```

#### IndustrialDevices
```javascript
{
  _id: ObjectId,
  buildingId: ObjectId,
  name: String,
  type: String,           // "plc", "hmi", "sensor", "actuator"
  manufacturer: String,
  model: String,
  ipAddress: String,
  protocol: {
    type: String,       // "modbus-tcp", "siemens-s7", "bacnet"
    port: Number,
    unitId: Number      // Pour Modbus
  },
  connectedTo: ObjectId,  // Référence vers NetworkEquipment
  function: String,       // Description fonctionnelle
  tags: [String],
  status: String,
  notes: String,
  createdAt: Date,
  updatedAt: Date
}
```

## 🔧 Fonctionnalités du POC

### Phase 1 - Gestion des données de base
- [ ] CRUD Organizations
- [ ] CRUD Sites avec géolocalisation
- [ ] CRUD Buildings avec hiérarchie
- [ ] Dashboard de vue d'ensemble

### Phase 2 - Équipements réseau
- [ ] Enregistrement des équipements réseau
- [ ] Gestion des connexions (uplinks)
- [ ] Vue topologie réseau
- [ ] Export des configurations

### Phase 3 - Équipements industriels
- [ ] Enregistrement des équipements industriels
- [ ] Mapping avec équipements réseau
- [ ] Documentation des protocoles
- [ ] Génération de rapports

### Phase 4 - Visualisation et export
- [ ] Dashboard interactif Node-RED
- [ ] Export Excel des inventaires
- [ ] Génération de diagrammes réseau
- [ ] API REST pour intégrations externes

## 📁 Structure des flows Node-RED

```
Flows suggérés:
├── 01_API_Organizations
├── 02_API_Sites
├── 03_API_Buildings
├── 04_API_NetworkEquipment
├── 05_API_IndustrialDevices
├── 06_Dashboard_Overview
├── 07_Dashboard_Network
├── 08_Dashboard_Industrial
├── 09_Reports_Export
└── 10_Utils_Database
```

## 🔌 Endpoints API suggérés

### Organizations
```
GET    /api/organizations
GET    /api/organizations/:id
POST   /api/organizations
PUT    /api/organizations/:id
DELETE /api/organizations/:id
```

### Sites
```
GET    /api/sites
GET    /api/sites/:id
GET    /api/organizations/:orgId/sites
POST   /api/sites
PUT    /api/sites/:id
DELETE /api/sites/:id
```

### Network Equipment
```
GET    /api/network-equipment
GET    /api/network-equipment/:id
GET    /api/buildings/:buildingId/network-equipment
POST   /api/network-equipment
PUT    /api/network-equipment/:id
DELETE /api/network-equipment/:id
GET    /api/network-equipment/:id/topology
```

## 🛠️ Commandes utiles

### Docker
```bash
# Démarrer les conteneurs
docker-compose up -d

# Arrêter les conteneurs
docker-compose down

# Voir les logs Node-RED
docker-compose logs -f nodered

# Voir les logs MongoDB
docker-compose logs -f mongodb

# Redémarrer Node-RED
docker-compose restart nodered

# Nettoyer tout (⚠️ perte de données)
docker-compose down -v
```

### MongoDB
```bash
# Se connecter au shell MongoDB
docker exec -it edgemap-mongodb mongosh edgemap

# Backup de la base
docker exec edgemap-mongodb mongodump -d edgemap -o /tmp/backup

# Restore de la base
docker exec edgemap-mongodb mongorestore -d edgemap /tmp/backup/edgemap
```

## 📝 Notes de développement

### Patterns Node-RED recommandés

1. **Validation des entrées**: Utiliser des function nodes pour valider avant insertion MongoDB
2. **Gestion des erreurs**: Toujours avoir un catch node par flow
3. **Logging**: Utiliser des debug nodes avec niveaux appropriés
4. **Modularité**: Créer des subflows pour les opérations répétitives

### Bonnes pratiques MongoDB

1. **Indexes**: Créer des index sur les champs recherchés fréquemment
```javascript
db.sites.createIndex({ code: 1 }, { unique: true })
db.networkEquipment.createIndex({ ipAddress: 1 })
```

2. **Validation**: Activer la validation de schéma MongoDB pour la cohérence

3. **Références**: Utiliser ObjectId pour les relations entre collections

## 🔒 Sécurité

### Pour la production

- [ ] Activer l'authentification MongoDB
- [ ] Configurer l'authentification Node-RED
- [ ] Mettre en place HTTPS
- [ ] Définir des variables d'environnement sécurisées
- [ ] Limiter l'exposition des ports
- [ ] Mettre en place des backups automatiques

## 📚 Ressources

- [Node-RED Documentation](https://nodered.org/docs/)
- [MongoDB Manual](https://docs.mongodb.com/manual/)
- [node-red-node-mongodb](https://flows.nodered.org/node/node-red-node-mongodb)
- [Docker Compose](https://docs.docker.com/compose/)

## 🐛 Dépannage

### Node-RED ne démarre pas
```bash
# Vérifier les logs
docker-compose logs nodered

# Supprimer le volume et recréer
docker-compose down -v
docker-compose up -d
```

### MongoDB connexion refusée
```bash
# Vérifier que MongoDB est démarré
docker-compose ps

# Vérifier les logs MongoDB
docker-compose logs mongodb

# Tester la connexion
docker exec -it edgemap-mongodb mongosh
```

### Port déjà utilisé
```bash
# Trouver le processus utilisant le port
lsof -i :1880  # ou :27017

# Modifier les ports dans docker-compose.yml
```

## 📄 License

Usage interne CLAUGER

## ✍️ Auteur

Johann - Cybersecurity Specialist @ CLAUGER

---

**Version**: 1.0.0-POC  
**Date**: Novembre 2025