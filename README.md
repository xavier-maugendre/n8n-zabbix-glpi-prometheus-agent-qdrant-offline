
# 🎛️ n8n-zabbix-glpi-aiops — Création & clôture automatique de tickets GLPI avec diagnostic IA

![n8n](https://img.shields.io/badge/n8n-workflow-blue?logo=n8n&logoColor=white)
![Zabbix](https://img.shields.io/badge/Zabbix-7.x-red?logo=zabbix&logoColor=white)
![GLPI](https://img.shields.io/badge/GLPI-10.x-orange?logo=glpi&logoColor=white)
![Prometheus](https://img.shields.io/badge/Prometheus-metrics-orange?logo=prometheus&logoColor=white)
![Loki](https://img.shields.io/badge/Loki-logs-yellow?logo=grafana&logoColor=white)
![Qdrant](https://img.shields.io/badge/Qdrant-vector%20store-red?logo=qdrant&logoColor=white)
![Ollama](https://img.shields.io/badge/Ollama-embeddings-black?logo=ollama&logoColor=white)
![Gemma](https://img.shields.io/badge/Gemma-4-4285F4?logo=google&logoColor=white)
![100% Local](https://img.shields.io/badge/100%25-local%20%2F%20offline-brightgreen)
![License](https://img.shields.io/badge/license-MIT-green)

> 🇬🇧 English speakers: 100% on-prem n8n workflow that turns Zabbix alerts into AI-diagnosed GLPI tickets using a local Gemma model — no data sent to OpenAI, Gemini, Claude or any external API. Auto-closes tickets on resolution and feeds resolved cases into a Qdrant vector store for RAG.

---
<img width="1536" height="1024" alt="Architecture" src="https://github.com/user-attachments/assets/1bb6740f-c106-4dee-8290-465081ee120d" />
---
## 🔒 Pourquoi "offline" ?

**Aucune donnée ne quitte ton infrastructure.** Le diagnostic IA, les embeddings et le stockage vectoriel tournent **intégralement en local** :

- **LLM** : Gemma 4 servi en local (via une API compatible OpenAI — LM Studio, Ollama, vLLM…)
- **Embeddings** : `nomic-embed-text` via Ollama, en local
- **Vector store** : Qdrant self-hosted

Le node n8n s'appelle `OpenAI Chat Model` mais c'est juste le **type de node LangChain** utilisé : il pointe en réalité vers **ton serveur local** qui expose une API au format OpenAI. Aucun appel sortant vers `api.openai.com`, `api.anthropic.com`, `generativelanguage.googleapis.com` ou autre service tiers.

Idéal pour les environnements souverains, OIV, OSE, ou simplement pour ne pas fuiter ses logs et ses incidents vers un cloud public.

---

## ✨ Fonctionnalités

- **100% local / offline** — LLM, embeddings et vector store tournent **on-prem**, aucune donnée envoyée à un service tiers (pas d'OpenAI, Gemini, Claude…)
- **Webhook unique pour PROBLEM et OK** — un seul point d'entrée Zabbix gère création et clôture via une branche conditionnelle
- **Création automatique de ticket GLPI** — ouverture, association à l'asset (Computer) via le hostname, urgence et contenu pré-remplis
- **Diagnostic IA croisé multi-sources** — l'agent corrèle l'alerte Zabbix, les métriques `item.get` Zabbix, les métriques Prometheus/cAdvisor/process_exporter et les logs Loki des 5 dernières minutes
- **Top conteneurs & top processus identifiés** — détection automatique du conteneur ou du process fautif via `topk` Prometheus pour le serveur Docker
- **Acquittement Zabbix avec n° de ticket** — l'event Zabbix est acquitté avec l'ID GLPI directement renvoyé en commentaire
- **Clôture automatique sur OK** — recherche du ticket par `event_id`, ajout d'une `ITILSolution` puis passage en statut Clos (6)
- **RAG sur historique GLPI** — les tickets résolus éligibles (durée > 2 min, contenu > 100 caractères, technicien identifié) sont vectorisés via Ollama et indexés dans Qdrant
- **Historique consulté à chaque incident** — l'agent IA interroge systématiquement la base Qdrant pour proposer en priorité une solution déjà appliquée
- **Diagnostic structuré pour humain pressé** — sortie en 3 blocs : *Constat & Responsable* / *Analyse Logs* / *Hypothèse & Action*
- **Anti-pollution de la base vectorielle** — un node de validation rejette les tickets clôturés en moins de 2 min ou sans contenu utile

---

## 🧱 Stack

| Composant | Détail |
|-----------|--------|
| Orchestrateur | n8n (workflow JSON importable) |
| Supervision | Zabbix 7.x (`api_jsonrpc.php`, `event.acknowledge`, `item.get`) |
| ITSM | GLPI 10.x (REST API : `/initSession`, `/search/Computer`, `/Ticket`, `/ITILSolution`) |
| Métriques système | Prometheus + node_exporter + cAdvisor + process_exporter |
| Logs | Loki (`/loki/api/v1/query_range`) |
| LLM | **Gemma 4** servi en local (via API OpenAI-compatible : LM Studio, Ollama, vLLM…) |
| Embeddings | `nomic-embed-text` via Ollama (local, sur `:1234/v1/embeddings`) |
| Vector store | Qdrant (collection `glpi_incidents`) |
| Auth Webhook | Header Auth (token partagé Zabbix → n8n) |

---

## 🖥️ Utilisation locale

Le projet est un workflow n8n. Il s'importe dans une instance n8n existante.

```bash
# Cloner le repo
git clone https://github.com/xavier-maugendre/n8n-zabbix-glpi-aiops.git
cd n8n-zabbix-glpi-aiops

# Ouvrir le JSON pour inspection
open Création_de_ticket_offline.json        # macOS
xdg-open Création_de_ticket_offline.json    # Linux
start Création_de_ticket_offline.json       # Windows
```

Import dans n8n :

1. n8n → **Workflows** → **Import from File** → sélectionner `Création_de_ticket_offline.json`
2. Recréer les credentials manquants (voir section ⚙️)
3. Activer le workflow et copier l'URL du Webhook
4. Côté Zabbix : créer un **Media type Webhook** qui POST en JSON sur cette URL avec le token en header

---

## ⚙️ Paramètres / Configuration

### Credentials n8n à créer

| Credential | Type | Utilisé par |
|------------|------|-------------|
| `Header - Zabbix Token` | HTTP Header Auth | Node `Webhook` (auth entrante depuis Zabbix) |
| `Zabbix account` | Zabbix API | Nodes `Zabbix Acquittement` et `Zabbix Get Metrics` |
| `OpenAI account` | OpenAI API key | Node `OpenAI Chat Model` — ⚠️ pointer vers **ton endpoint local** (LM Studio, vLLM…), pas vers `api.openai.com` |
| `Ollama account` | Ollama API | Node `Embeddings Ollama` (utilisé par le tool Qdrant Vector Store) |
| `QdrantApi account` | Qdrant API | Node `Qdrant Vector Store` |

> ⚠️ Le workflow contient en dur des `App-Token` et `user_token` GLPI. **À remplacer** par des credentials n8n typés `HTTP Header Auth` avant tout usage en prod.

### Payload Webhook attendu (depuis Zabbix)

```json
{
  "status": "PROBLEM",
  "event_id": "12345",
  "host": "<hostname>",
  "host_id": "<zabbix_host_id>",
  "ip": "<host_ip>",
  "alert": "High CPU utilization",
  "severity": "High",
  "zabbix_url": "http://<ZABBIX_HOST>/tr_events.php?eventid=12345"
}
```

Les champs critiques : `status` (PROBLEM/OK), `event_id` (pivot pour la clôture), `host` (clé pour Loki, Prometheus, GLPI Computer), `host_id` (clé pour Zabbix `item.get`).

### Endpoints cibles à configurer

Avant import, remplacer dans chaque node HTTP les URLs par celles de ton infra. Variables suggérées :

| Variable | Service | Exemple d'URL |
|----------|---------|---------------|
| `${GLPI_HOST}` | GLPI REST API | `http://${GLPI_HOST}:8088/apirest.php` |
| `${LOKI_HOST}` | Loki | `http://${LOKI_HOST}:3100` |
| `${PROMETHEUS_HOST}` | Prometheus | `http://${PROMETHEUS_HOST}:9090` |
| `${ZABBIX_HOST}` | Zabbix API | `http://${ZABBIX_HOST}/api_jsonrpc.php` |
| `${QDRANT_HOST}` | Qdrant | `http://${QDRANT_HOST}:6333` |
| `${OLLAMA_HOST}` | Ollama (endpoint compatible OpenAI) | `http://${OLLAMA_HOST}:1234/v1/embeddings` |

> 💡 Côté n8n, ces valeurs peuvent être centralisées via des **variables d'environnement** (`{{ $env.GLPI_HOST }}`) plutôt qu'écrites en dur dans chaque node.

### Règles de validation avant indexation Qdrant

| Règle | Seuil | Action |
|-------|-------|--------|
| Technicien assigné | `users_id_lastupdater != 0` | Sinon rejet |
| Durée de résolution | `≥ 2 min` | Sinon rejet (faux positif) |
| Contenu nettoyé HTML | `≥ 100 caractères` | Sinon rejet |
| Collection cible | `glpi_incidents` | Insert via `PUT /collections/{name}/points` |

---

## 📊 Logique du workflow

### Branche PROBLEM (création)

```
Webhook
  └─> PROBLEM OR RESOLVED (IF status == "PROBLEM")
        └─> Get Token GLPI
              └─> Recherche ID (Computer GLPI via hostname)
                    └─> Loki Get Log (logs des 5 dernières minutes)
                          └─> Tranformation Log (formatage texte pour LLM)
                                └─> Zabbix Get Metrics (item.get sur host_id)
                                      └─> Prometheus Get Metrics (CPU/RAM/disk/net + topk conteneurs/process)
                                            └─> Transformation Prometheus
                                                  └─> AI Agent (+ Qdrant Vector Store en tool)
                                                        └─> Création de ticket GLPI
                                                              └─> Zabbix Acquittement (event.acknowledge action=6)
```

### Branche OK / RESOLVED (clôture + RAG)

```
Webhook
  └─> PROBLEM OR RESOLVED (IF status != "PROBLEM")
        └─> Get Token GLPI1
              └─> Recherche ID Ticket (par [ID:event_id] dans le titre)
                    └─> If (ticket trouvé)
                          └─> SOLUTION de ticket (POST ITILSolution)
                                └─> GET Ticket complet
                                      └─> Code Validation (filtres anti-pollution)
                                            └─> Ollama Embeddings
                                                  └─> Format Qdrant
                                                        └─> Qdrant insert hybride (PUT points)
                                                              └─> CLOSE de ticket (PUT status=6)
```

### Requête Prometheus utilisée

Une seule requête `or` qui agrège tout via `label_replace` et expose une métrique `metric` lisible côté n8n :

```
cpu_percent, ram_percent, swap_percent, disk_percent (mountpoint=/)
load1, load5, procs_running, tcp_connections
net_rx_bytes_per_sec, net_tx_bytes_per_sec (device=eth0)
top_container_cpu_percent (topk 3, cAdvisor)
top_container_ram_mb     (topk 3, cAdvisor)
top_process_cpu_percent  (topk 5, process_exporter)
top_process_ram_bytes    (topk 5, process_exporter)
```

---

## 🤖 Prompt système de l'agent IA

L'agent reçoit en entrée structurée :

1. Déclencheur (alerte Zabbix : nom, sévérité, host, IP)
2. Métriques globales Zabbix (`item.get` brut)
3. Métriques Prometheus formatées (hôte + top conteneurs + top processus)
4. Logs Loki des 5 dernières minutes
5. Tool de recherche Qdrant sur l'historique GLPI

Règles imposées au modèle :

- Réponse **en français**, ton technique et concis, pas de formules de politesse
- **Pas d'hallucination** : si Loki est vide, le dire explicitement
- Priorité aux **chiffres** (métriques) sur les logs
- **Règle d'architecture** : seul le serveur `Docker` a des conteneurs → métriques `top_container_*` n'ont de sens que pour lui
- Sortie structurée en 3 sections : `**Constat & Responsable**` / `**Analyse Logs**` / `**Hypothèse & Action**`

---

## 📁 Structure

```
n8n-zabbix-glpi-aiops/
├── Création_de_ticket_offline.json   # Workflow n8n exporté
└── README.md
```

---

## 🔗 Projets liés

- [ansible-zabbix-deploy](https://github.com/xavier-maugendre/ansible-zabbix-deploy) — Déploiement automatisé Zabbix 7.x via Ansible (AlmaLinux 9)
- [zabbix-server-tuner](https://github.com/xavier-maugendre/zabbix-server-tuner) — Simulateur de configuration `zabbix_server.conf`
