{
  "name": "Création de ticket offline",
  "nodes": [
    {
      "parameters": {
        "httpMethod": "POST",
        "path": "REPLACE_WITH_WEBHOOK_UUID",
        "authentication": "headerAuth",
        "options": {}
      },
      "type": "n8n-nodes-base.webhook",
      "typeVersion": 2.1,
      "position": [
        -720,
        -32
      ],
      "id": "32ff0196-6241-427f-8498-164186bda87f",
      "name": "Webhook",
      "webhookId": "REPLACE_WITH_WEBHOOK_UUID",
      "credentials": {
        "httpHeaderAuth": {
          "id": "REPLACE_CRED_ID_HTTPHEADERAUTH",
          "name": "Header - Zabbix Token"
        }
      }
    },
    {
      "parameters": {
        "url": "http://GLPI_HOST:8088/apirest.php/initSession",
        "sendHeaders": true,
        "headerParameters": {
          "parameters": [
            {
              "name": "App-Token",
              "value": "REPLACE_WITH_GLPI_APP_TOKEN"
            },
            {
              "name": "Authorization",
              "value": "user_token REPLACE_WITH_GLPI_USER_TOKEN"
            }
          ]
        },
        "options": {}
      },
      "id": "010976d2-1059-4eae-9c13-1aa465392c87",
      "name": "Get Token GLPI",
      "type": "n8n-nodes-base.httpRequest",
      "typeVersion": 3,
      "position": [
        -272,
        -304
      ]
    },
    {
      "parameters": {
        "url": "http://GLPI_HOST:8088/apirest.php/search/Computer/",
        "sendQuery": true,
        "queryParameters": {
          "parameters": [
            {
              "name": "criteria[0][field]",
              "value": "1"
            },
            {
              "name": "criteria[0][searchtype]",
              "value": "contains"
            },
            {
              "name": "criteria[0][value]",
              "value": "={{ $('Webhook').item.json.body.host }}"
            },
            {
              "name": "forcedisplay[0]",
              "value": "2"
            }
          ]
        },
        "sendHeaders": true,
        "headerParameters": {
          "parameters": [
            {
              "name": "App-Token",
              "value": "REPLACE_WITH_GLPI_APP_TOKEN"
            },
            {
              "name": "Authorization",
              "value": "user_token REPLACE_WITH_GLPI_USER_TOKEN"
            },
            {
              "name": "Session-Token",
              "value": "={{ $json.session_token }}"
            }
          ]
        },
        "options": {}
      },
      "id": "3ed097d5-2cdf-438e-adca-85b41693348b",
      "name": "Recherche ID",
      "type": "n8n-nodes-base.httpRequest",
      "typeVersion": 3,
      "position": [
        -48,
        -304
      ]
    },
    {
      "parameters": {
        "method": "POST",
        "url": "http://GLPI_HOST:8088/apirest.php/Ticket/",
        "sendHeaders": true,
        "headerParameters": {
          "parameters": [
            {
              "name": "App-Token",
              "value": "REPLACE_WITH_GLPI_APP_TOKEN"
            },
            {
              "name": "Authorization",
              "value": "user_token REPLACE_WITH_GLPI_USER_TOKEN"
            },
            {
              "name": "Session-Token",
              "value": "={{ $('Get Token GLPI').item.json.session_token }}"
            }
          ]
        },
        "sendBody": true,
        "specifyBody": "json",
        "jsonBody": "={\n  \"input\": [\n    {\n      \"name\": {{ JSON.stringify(\"[ID:\" + $('Webhook').item.json.body.event_id + \"] \" + $('Webhook').item.json.body.alert) }},\n      \"content\": {{ JSON.stringify(\"⚠️ Alerte Zabbix ⚠️\\n\\nServeur : \" + $('Webhook').item.json.body.host + \"\\nIP : \" + $('Webhook').item.json.body.ip + \"\\nSévérité : \" + $('Webhook').item.json.body.severity + \"\\n\\n--- 🤖 DIAGNOSTIC IA ---\\n\" + $('AI Agent').item.json.output + \"\\n\\nLien Zabbix : \" + $('Webhook').item.json.body.zabbix_url) }},\n      \"urgency\": \"3\",\n      \"items_id\": {\n        \"Computer\": [\n          \"{{ $('Recherche ID').item.json.data[0]['2'] }}\"\n        ]\n      }\n    }\n  ]\n}",
        "options": {}
      },
      "id": "811b32f9-4cc7-406f-a479-9ca50fd26e34",
      "name": "Création de ticket",
      "type": "n8n-nodes-base.httpRequest",
      "typeVersion": 3,
      "position": [
        1824,
        -304
      ]
    },
    {
      "parameters": {
        "curlImport": "",
        "httpVariantWarning": "",
        "method": "POST",
        "url": "http://ZABBIX_HOST/api_jsonrpc.php",
        "": "",
        "authentication": "predefinedCredentialType",
        "nodeCredentialType": "zabbixApi",
        "provideSslCertificates": false,
        "sendQuery": false,
        "sendHeaders": false,
        "sendBody": true,
        "contentType": "json",
        "specifyBody": "json",
        "jsonBody": "={\n   \"jsonrpc\": \"2.0\",\n   \"method\": \"event.acknowledge\",\n   \"params\": {\n       \"eventids\": \"{{ $('Webhook').item.json.body.event_id }}\",\n       \"action\": 6,\n       \"message\": \"Ticket GLPI créé : #{{ $json.id }}\"\n   },\n   \"id\": 1\n}",
        "options": {},
        "infoMessage": ""
      },
      "type": "n8n-nodes-base.httpRequest",
      "typeVersion": 4.3,
      "position": [
        2048,
        -304
      ],
      "id": "e171e2f8-5370-4e9e-b37d-f6dd4adbd64a",
      "name": "Zabbix Acquittement",
      "extendsCredential": "zabbixApi",
      "credentials": {
        "zabbixApi": {
          "id": "REPLACE_CRED_ID_ZABBIXAPI",
          "name": "Zabbix account"
        }
      }
    },
    {
      "parameters": {
        "url": "http://LOKI_HOST:3100/loki/api/v1/query_range",
        "sendQuery": true,
        "queryParameters": {
          "parameters": [
            {
              "name": "query",
              "value": "={host=\"{{ $node[\"Webhook\"].json[\"body\"][\"host\"] }}\"}"
            },
            {
              "name": "limit",
              "value": "50"
            },
            {
              "name": "start",
              "value": "={{ (Math.floor(Date.now() / 1000) - 300) * 1000000000 }}"
            }
          ]
        },
        "options": {}
      },
      "type": "n8n-nodes-base.httpRequest",
      "typeVersion": 4.3,
      "position": [
        176,
        -304
      ],
      "id": "6abd6007-0f71-4af8-8124-6037df8a7686",
      "name": "Loki Get Log"
    },
    {
      "parameters": {
        "jsCode": "const result = $input.first().json.data.result;\nlet textePourIA = \"Logs trouvés dans Loki :\\n\";\n\nresult.forEach(stream => {\n  stream.values.forEach(val => {\n    textePourIA += `- ${val[1]}\\n`;\n  });\n});\n\nreturn { logs_formates: textePourIA };"
      },
      "type": "n8n-nodes-base.code",
      "typeVersion": 2,
      "position": [
        400,
        -304
      ],
      "id": "6a4f2809-9acb-4fa9-96e9-b2bd74876073",
      "name": "Tranformation Log"
    },
    {
      "parameters": {
        "promptType": "define",
        "text": "==Tu es un Expert SRE (Site Reliability Engineer) responsable de la supervision d'une infrastructure critique.\nTon rôle est de diagnostiquer la cause racine d'un incident en croisant plusieurs sources d'information : L'Alerte, les Métriques (Zabbix & Prometheus) et les Logs.\n\n---\n🔴 1. LE DÉCLENCHEUR (L'Alerte Zabbix)\n- Nom de l'alerte : {{ $('Webhook').item.json.body.alert }}\n- Sévérité : {{ $('Webhook').item.json.body.severity }}\n- Serveur concerné : {{ $('Webhook').item.json.body.host }} ({{ $('Webhook').item.json.body.ip }})\n\n---\n📊 2. LES SIGNES VITAUX (Métriques Globales Zabbix)\nVoici un JSON brut contenant les capteurs globaux du serveur.\n{{ JSON.stringify($('Zabbix Get Metrics').item.json.result) }}\n\n---\n🐳 3. MÉTRIQUES SYSTÈME & CONTENEURS (Prometheus / cAdvisor)\nVoici un instantané précis de l'état du serveur et le Top 3 des conteneurs les plus gourmands en CPU et RAM au moment de l'incident.\n{{ $('Transformation Prometheus').item.json.prometheus_formate ? $('Transformation Prometheus').item.json.prometheus_formate : \"⚠️ Aucune métrique Prometheus disponible.\" }}\n\n---\n📜 4. LES PREUVES (Logs Système - Dernières 5 min)\nAnalyse ces logs pour trouver des erreurs, des arrêts de service ou des \"Kernel Panics\".\n{{ $('Tranformation Log').item.json.logs_formates ? $('Tranformation Log').item.json.logs_formates : \"⚠️ Aucun log pertinent retourné par Loki.\" }}\n\n---\n🔍 5. CONSULTATION DE L'HISTORIQUE (Tool disponible)\nTu as accès à un outil de recherche dans l'historique des incidents GLPI résolus par les techniciens.\n\nQUAND l'utiliser :\n- Systématiquement à chaque analyse d'incident\n- Formule ta recherche avec le type d'alerte + le serveur concerné\n  Exemple : \"High CPU utilization Ansible\" ou \"disk full Docker\"\n\nCE QUE TU DOIS FAIRE avec les résultats :\n- Si similarité élevée → propose en priorité la solution qui a déjà fonctionné\n- Si même serveur + même alerte → signale explicitement le pattern récurrent\n- Si aucun résultat → indique \"Premier incident de ce type\"\n\nINTÈGRE les incidents similaires dans ta section Hypothèse & Action.\n\n---\n🧠 TA DÉMARCHE D'ANALYSE (Raisonnement dynamique) :\n\n1. **Qualification de l'Alerte** : De quoi parle l'alerte ? (Disque plein, Service coupé, CPU, RAM ?).\n2. **Corrélation Zabbix & Prometheus** :\n   - Si Alerte Disque : Vérifie `disk_percent` dans Prometheus ou `vfs.fs.size` dans Zabbix.\n   - Si Alerte Charge/CPU/RAM : Identifie la cause. ⚠️ ATTENTION : Seul le serveur \"Docker\" possède des conteneurs. Si l'alerte concerne ce serveur, cherche le conteneur fautif via `top_container_cpu_percent` ou `top_container_ram_mb`. Si l'alerte concerne les serveurs \"Ansible\" ou \"Zabbix\", concentre-toi uniquement sur les processus ou métriques système globales car il n'y a pas de conteneurs.\n   - Si Alerte Service : Le service tourne-t-il ? (Zabbix `proc.num`).\n3. **Corrélation Logs** :\n   - Trouves-tu une erreur explicite correspondant au problème à l'heure de l'alerte ?\n   - Si les métriques sont rouges mais les logs vides, précise qu'il s'agit d'une \"saturation silencieuse\" (ex: fuite mémoire, boucle infinie).\n\n👉 **SORTIE ATTENDUE (Pour Ticket GLPI)** :\nRédige un diagnostic technique, synthétique et professionnel en 3 parties :\n- **Constat & Responsable** : Ce que confirment les métriques, en nommant explicitement le conteneur fautif s'il y en a un (ex: \"Je confirme une saturation CPU à 95%, causée principalement par le conteneur [mysql-db] à 75%\").\n- **Analyse Logs** : Ce que disent (ou ne disent pas) les logs.\n- **Hypothèse & Action** : La cause probable et la commande/action recommandée pour réparer (ex: redémarrer le conteneur ciblé, purger les logs).",
        "options": {
          "systemMessage": "Tu es un Expert SRE (Site Reliability Engineer) Senior et un Assistant de Support Niveau 2.\nTa mission est d'analyser automatiquement les incidents remontés par Zabbix pour pré-mâcher le travail des administrateurs humains dans GLPI.\nTu consultes systématiquement l'historique des incidents avant de formuler ton diagnostic.\n\nTES RÈGLES D'OR (A respecter impérativement) :\n\n1. **Ton/Style** : Sois extrêmement concis, technique et professionnel. Pas de formules de politesse (\"Bonjour\", \"J'espère que vous allez bien\"). Va droit au but.\n\n2. **Langue** : Réponds TOUJOURS en Français.\n\n3. **Groundedness (Pas d'hallucination)** :\n   - Ne jamais inventer de logs ou de métriques. Si les logs fournis sont vides ou marqués \"Aucun log pertinent\", tu DOIS le dire explicitement (ex: \"Les logs ne montrent aucune trace de l'incident\").\n   - Base ton diagnostic sur les MÉTRIQUES (les chiffres) en priorité. Les chiffres ne mentent pas.\n\n4. **Interprétation des Données (Zabbix & Prometheus)** :\n   - Tu vas recevoir un JSON brut de Zabbix ET une liste claire de métriques Prometheus/cAdvisor. Croise ces deux sources.\n   - Tu dois trouver l'aiguille dans la botte de foin (le disque plein, le service arrêté, la RAM saturée). Ignore les métriques qui vont bien et concentre-toi sur l'anormal.\n   - ⚠️ **RÈGLE ARCHITECTURE** : Seul le serveur nommé \"Docker\" héberge des conteneurs. Si l'alerte concerne ce serveur, regarde impérativement les métriques \"top_container\" de Prometheus pour identifier le conteneur fautif. Pour les autres serveurs (Ansible, Zabbix...), analyse uniquement les métriques système.\n\n5. **Format de Sortie** :\n   - Utilise le Markdown pour la mise en forme (Gras ** **, Listes -).\n   - Structure ta réponse en 3 parties claires (Constat, Analyse Logs, Action) pour qu'elle soit lisible en un coup d'œil par un humain pressé."
        }
      },
      "type": "@n8n/n8n-nodes-langchain.agent",
      "typeVersion": 3.1,
      "position": [
        1264,
        -304
      ],
      "id": "d9502b72-d0b4-4623-921a-bb77f6c91c40",
      "name": "AI Agent"
    },
    {
      "parameters": {
        "conditions": {
          "options": {
            "caseSensitive": true,
            "leftValue": "",
            "typeValidation": "strict",
            "version": 3
          },
          "conditions": [
            {
              "id": "14a17986-af52-42f5-b54d-4f2623538317",
              "leftValue": "={{ $json.body.status }}",
              "rightValue": "PROBLEM",
              "operator": {
                "type": "string",
                "operation": "equals"
              }
            }
          ],
          "combinator": "or"
        },
        "options": {}
      },
      "type": "n8n-nodes-base.if",
      "typeVersion": 2.3,
      "position": [
        -496,
        -32
      ],
      "id": "666d448b-ceec-4763-9c75-e1a5f6fb52f5",
      "name": "PROBLEM OR RESOLVED"
    },
    {
      "parameters": {
        "url": "http://GLPI_HOST:8088/apirest.php/initSession",
        "sendHeaders": true,
        "headerParameters": {
          "parameters": [
            {
              "name": "App-Token",
              "value": "REPLACE_WITH_GLPI_APP_TOKEN"
            },
            {
              "name": "Authorization",
              "value": "user_token REPLACE_WITH_GLPI_USER_TOKEN"
            }
          ]
        },
        "options": {}
      },
      "id": "ce5198ab-8b28-45d4-8568-5b560170cc1b",
      "name": "Get Token GLPI1",
      "type": "n8n-nodes-base.httpRequest",
      "typeVersion": 3,
      "position": [
        -144,
        384
      ]
    },
    {
      "parameters": {
        "method": "PUT",
        "url": "=http://GLPI_HOST:8088/apirest.php/Ticket/{{ $('Recherche ID Ticket').item.json.data[0]['2'] }}",
        "sendHeaders": true,
        "headerParameters": {
          "parameters": [
            {
              "name": "App-Token",
              "value": "REPLACE_WITH_GLPI_APP_TOKEN"
            },
            {
              "name": "Authorization",
              "value": "user_token REPLACE_WITH_GLPI_USER_TOKEN"
            },
            {
              "name": "Session-Token",
              "value": "={{ $('Get Token GLPI1').item.json.session_token }}"
            }
          ]
        },
        "sendBody": true,
        "specifyBody": "json",
        "jsonBody": "={\n  \"input\": {\n    \"status\": 6\n  }\n}",
        "options": {}
      },
      "id": "08c74afa-b2bc-4fb6-a074-7fa2ef926262",
      "name": "CLOSE de ticket",
      "type": "n8n-nodes-base.httpRequest",
      "typeVersion": 3,
      "position": [
        1920,
        384
      ]
    },
    {
      "parameters": {
        "url": "http://GLPI_HOST:8088/apirest.php/search/Ticket/",
        "sendQuery": true,
        "queryParameters": {
          "parameters": [
            {
              "name": "criteria[0][field]",
              "value": "1"
            },
            {
              "name": "criteria[0][searchtype]",
              "value": "contains"
            },
            {
              "name": "criteria[0][value]",
              "value": "=[ID:{{ $('Webhook').item.json.body.event_id }}]"
            },
            {
              "name": "criteria[1][link]",
              "value": "AND"
            },
            {
              "name": "criteria[1][field]",
              "value": "12"
            },
            {
              "name": "criteria[1][searchtype]",
              "value": "equals"
            },
            {
              "name": "criteria[1][value]",
              "value": "1"
            }
          ]
        },
        "sendHeaders": true,
        "headerParameters": {
          "parameters": [
            {
              "name": "App-Token",
              "value": "REPLACE_WITH_GLPI_APP_TOKEN"
            },
            {
              "name": "Authorization",
              "value": "user_token REPLACE_WITH_GLPI_USER_TOKEN"
            },
            {
              "name": "Session-Token",
              "value": "={{ $json.session_token }}"
            }
          ]
        },
        "options": {}
      },
      "id": "60f25438-0063-49bc-b385-4a37dc80afd7",
      "name": "Recherche ID Ticket",
      "type": "n8n-nodes-base.httpRequest",
      "typeVersion": 3,
      "position": [
        80,
        384
      ]
    },
    {
      "parameters": {
        "method": "POST",
        "url": "=http://GLPI_HOST:8088/apirest.php/ITILSolution/",
        "sendHeaders": true,
        "headerParameters": {
          "parameters": [
            {
              "name": "App-Token",
              "value": "REPLACE_WITH_GLPI_APP_TOKEN"
            },
            {
              "name": "Authorization",
              "value": "user_token REPLACE_WITH_GLPI_USER_TOKEN"
            },
            {
              "name": "Session-Token",
              "value": "={{ $('Get Token GLPI1').item.json.session_token }}"
            }
          ]
        },
        "sendBody": true,
        "specifyBody": "json",
        "jsonBody": "={\n  \"input\": {\n    \"items_id\": \"{{ $('Recherche ID Ticket').item.json.data[0]['2'] }}\",\n    \"itemtype\": \"Ticket\",\n    \"content\": \"✅ RÉSOLU : Alerte terminée. Clôture automatique.\",\n    \"solutiontypes_id\": 1\n  }\n}",
        "options": {}
      },
      "id": "4ac804d1-d20d-46ed-919e-36250c0c4757",
      "name": "SOLUTION de ticket",
      "type": "n8n-nodes-base.httpRequest",
      "typeVersion": 3,
      "position": [
        528,
        384
      ]
    },
    {
      "parameters": {
        "conditions": {
          "options": {
            "caseSensitive": true,
            "leftValue": "",
            "typeValidation": "strict",
            "version": 3
          },
          "conditions": [
            {
              "id": "88532857-d9ce-49e5-a2af-a5fbd41cc72f",
              "leftValue": "={{ $('Recherche ID Ticket').item.json.totalcount }}",
              "rightValue": 0,
              "operator": {
                "type": "number",
                "operation": "gt"
              }
            }
          ],
          "combinator": "and"
        },
        "options": {}
      },
      "type": "n8n-nodes-base.if",
      "typeVersion": 2.3,
      "position": [
        304,
        384
      ],
      "id": "d59ae785-bcd2-4f5c-bbd0-99010721c76b",
      "name": "If"
    },
    {
      "parameters": {
        "curlImport": "",
        "httpVariantWarning": "",
        "method": "POST",
        "url": "http://ZABBIX_HOST/api_jsonrpc.php",
        "": "",
        "authentication": "predefinedCredentialType",
        "nodeCredentialType": "zabbixApi",
        "provideSslCertificates": false,
        "sendQuery": false,
        "sendHeaders": false,
        "sendBody": true,
        "contentType": "json",
        "specifyBody": "json",
        "jsonBody": "={\n    \"jsonrpc\": \"2.0\",\n    \"method\": \"item.get\",\n    \"params\": {\n        \"output\": [\"name\", \"key_\", \"lastvalue\", \"units\"],\n        \"hostids\": \"{{ $('Webhook').item.json.body.host_id }}\",\n        \"filter\": {\n            \"status\": \"0\" \n        },\n        \"sortfield\": \"name\"\n   \n   },\n   \"id\": 1\n}",
        "options": {},
        "infoMessage": ""
      },
      "type": "n8n-nodes-base.httpRequest",
      "typeVersion": 4.3,
      "position": [
        592,
        -304
      ],
      "id": "0e692e4e-2115-44f1-ba62-07f5dffc0ad8",
      "name": "Zabbix Get Metrics",
      "extendsCredential": "zabbixApi",
      "credentials": {
        "zabbixApi": {
          "id": "REPLACE_CRED_ID_ZABBIXAPI",
          "name": "Zabbix account"
        }
      }
    },
    {
      "parameters": {
        "content": "## GLPI\nClôture Automatique du ticket & index Qdrant\n",
        "height": 336,
        "width": 2352
      },
      "type": "n8n-nodes-base.stickyNote",
      "position": [
        -208,
        288
      ],
      "typeVersion": 1,
      "id": "86178589-3cdc-47f0-9028-f9287b94c26b",
      "name": "Sticky Note"
    },
    {
      "parameters": {
        "content": "##  GLPI\nRécupération Token",
        "height": 336,
        "width": 448,
        "color": 3
      },
      "type": "n8n-nodes-base.stickyNote",
      "position": [
        -336,
        -416
      ],
      "typeVersion": 1,
      "id": "501c9ca9-7555-4da4-a6c2-6d0fdf228df6",
      "name": "Sticky Note1"
    },
    {
      "parameters": {
        "content": "## Loki & Zabbix\nRécupération des logs , métrique et Transformation",
        "height": 336,
        "width": 608,
        "color": 5
      },
      "type": "n8n-nodes-base.stickyNote",
      "position": [
        128,
        -416
      ],
      "typeVersion": 1,
      "id": "efa89442-d0bd-4715-97d6-3a6e5d882ab6",
      "name": "Sticky Note2"
    },
    {
      "parameters": {
        "content": "## Analyse IA\nCorrélation log, métrique Zabbix et recherche Vector Base",
        "height": 624,
        "width": 544,
        "color": 2
      },
      "type": "n8n-nodes-base.stickyNote",
      "position": [
        1168,
        -416
      ],
      "typeVersion": 1,
      "id": "b7404d9a-8d5a-4b34-809d-4845083f5b58",
      "name": "Sticky Note3"
    },
    {
      "parameters": {
        "content": "## GLPI\nCréation Automatique du Ticket & acquittement de l'évènement sur zabbix + Commentaire Numéro de ticket",
        "height": 336,
        "width": 512,
        "color": 6
      },
      "type": "n8n-nodes-base.stickyNote",
      "position": [
        1728,
        -416
      ],
      "typeVersion": 1,
      "id": "f01c7a7a-f470-476a-afac-2069313b79a7",
      "name": "Sticky Note4"
    },
    {
      "parameters": {
        "url": "http://PROMETHEUS_HOST:9090/api/v1/query",
        "sendQuery": true,
        "queryParameters": {
          "parameters": [
            {
              "name": "query",
              "value": "=label_replace(100 - (avg by (hostname) (rate(node_cpu_seconds_total{mode=\"idle\",hostname=\"{{ $('Webhook').item.json.body.host }}\"}[5m])) * 100), \"metric\", \"cpu_percent\", \"\", \"\")\nor\nlabel_replace((1 - node_memory_MemAvailable_bytes{hostname=\"{{ $('Webhook').item.json.body.host }}\"} / node_memory_MemTotal_bytes{hostname=\"{{ $('Webhook').item.json.body.host }}\"}) * 100, \"metric\", \"ram_percent\", \"\", \"\")\nor\nlabel_replace((1 - node_memory_SwapFree_bytes{hostname=\"{{ $('Webhook').item.json.body.host }}\"} / node_memory_SwapTotal_bytes{hostname=\"{{ $('Webhook').item.json.body.host }}\"}) * 100, \"metric\", \"swap_percent\", \"\", \"\")\nor\nlabel_replace(100 - (node_filesystem_avail_bytes{hostname=\"{{ $('Webhook').item.json.body.host }}\",mountpoint=\"/\"} / node_filesystem_size_bytes{hostname=\"{{ $('Webhook').item.json.body.host }}\",mountpoint=\"/\"} * 100), \"metric\", \"disk_percent\", \"\", \"\")\nor\nlabel_replace(node_load1{hostname=\"{{ $('Webhook').item.json.body.host }}\"}, \"metric\", \"load1\", \"\", \"\")\nor\nlabel_replace(node_load5{hostname=\"{{ $('Webhook').item.json.body.host }}\"}, \"metric\", \"load5\", \"\", \"\")\nor\nlabel_replace(node_procs_running{hostname=\"{{ $('Webhook').item.json.body.host }}\"}, \"metric\", \"procs_running\", \"\", \"\")\nor\nlabel_replace(node_netstat_Tcp_CurrEstab{hostname=\"{{ $('Webhook').item.json.body.host }}\"}, \"metric\", \"tcp_connections\", \"\", \"\")\nor\nlabel_replace(rate(node_network_receive_bytes_total{hostname=\"{{ $('Webhook').item.json.body.host }}\",device=\"eth0\"}[5m]), \"metric\", \"net_rx_bytes_per_sec\", \"\", \"\")\nor\nlabel_replace(rate(node_network_transmit_bytes_total{hostname=\"{{ $('Webhook').item.json.body.host }}\",device=\"eth0\"}[5m]), \"metric\", \"net_tx_bytes_per_sec\", \"\", \"\")\nor\nlabel_replace(topk(3, sum by (name) (rate(container_cpu_usage_seconds_total{name!=\"\", hostname=\"{{ $('Webhook').item.json.body.host }}\"}[5m])) * 100), \"metric\", \"top_container_cpu_percent\", \"\", \"\")\nor\nlabel_replace(topk(3, sum by (name) (container_memory_working_set_bytes{name!=\"\", hostname=\"{{ $('Webhook').item.json.body.host }}\"}) / 1024 / 1024), \"metric\", \"top_container_ram_mb\", \"\", \"\")\nor\nlabel_replace(topk(5, rate(namedprocess_namegroup_cpu_seconds_total{hostname=\"{{ $('Webhook').item.json.body.host }}\", mode=\"user\"}[2m])) * 100, \"metric\", \"top_process_cpu_percent\", \"\", \"\")\nor\nlabel_replace(topk(5, namedprocess_namegroup_memory_bytes{hostname=\"{{ $('Webhook').item.json.body.host }}\", mode=\"resident\"}), \"metric\", \"top_process_ram_bytes\", \"\", \"\")"
            }
          ]
        },
        "options": {}
      },
      "type": "n8n-nodes-base.httpRequest",
      "typeVersion": 4.3,
      "position": [
        816,
        -304
      ],
      "id": "16eed6f1-3601-492b-9e0b-2174b0880f00",
      "name": "Prometheus Get Metrics"
    },
    {
      "parameters": {
        "jsCode": "const results = $input.first().json.data.result;\nlet promMetrics = \"📈 Métriques Système (Hôte + Top Conteneurs + Top Processus) :\\n\";\n\nif (!results || results.length === 0) {\n  return { prometheus_formate: \"⚠️ Aucune métrique système Prometheus disponible.\" };\n}\n\nresults.forEach(item => {\n  const metricName = item.metric.metric || item.metric.__name__ || \"Métrique Inconnue\";\n  \n  // Nom du conteneur (cAdvisor)\n  const containerLabel = item.metric.name || item.metric.container || item.metric.container_name || item.metric.id;\n  const containerName = containerLabel ? ` [Conteneur: ${containerLabel}]` : \"\";\n\n  // Nom du processus (process_exporter)\n  const processLabel = item.metric.groupname;\n  const processName = processLabel ? ` [Process: ${processLabel}]` : \"\";\n\n  const metricValue = parseFloat(item.value[1]);\n\n  let unit = \"\";\n  let displayValue = metricValue.toFixed(2);\n\n  if (metricName.includes(\"percent\")) {\n    unit = \"%\";\n  } else if (metricName.includes(\"bytes_per_sec\")) {\n    unit = \" Octets/s\";\n  } else if (metricName.includes(\"ram_mb\")) {\n    unit = \" MB\";\n  } else if (metricName === \"top_process_ram_bytes\") {\n    // Convertir bytes → MB pour la lisibilité\n    displayValue = (metricValue / 1024 / 1024).toFixed(1);\n    unit = \" MB\";\n  }\n\n  promMetrics += `- ${metricName}${containerName}${processName} : ${displayValue}${unit}\\n`;\n});\n\nreturn { prometheus_formate: promMetrics };"
      },
      "type": "n8n-nodes-base.code",
      "typeVersion": 2,
      "position": [
        1008,
        -304
      ],
      "id": "6554de2a-1c9c-48bb-a2ed-fdd6ad2fe32e",
      "name": "Transformation Prometheus"
    },
    {
      "parameters": {
        "content": "## Prometheus\nRécupération des métriques et transformation",
        "height": 336,
        "width": 400,
        "color": 4
      },
      "type": "n8n-nodes-base.stickyNote",
      "position": [
        752,
        -416
      ],
      "typeVersion": 1,
      "id": "b60e9a82-9bb2-4e05-9bd6-c616bb5e3adb",
      "name": "Sticky Note5"
    },
    {
      "parameters": {
        "model": {
          "__rl": true,
          "value": "google/gemma-4-26b-a4b",
          "mode": "list",
          "cachedResultName": "google/gemma-4-26b-a4b"
        },
        "responsesApiEnabled": false,
        "options": {}
      },
      "type": "@n8n/n8n-nodes-langchain.lmChatOpenAi",
      "typeVersion": 1.3,
      "position": [
        1264,
        -112
      ],
      "id": "8d28f06d-1a2d-4c1e-9946-a872075d7e52",
      "name": "OpenAI Chat Model",
      "credentials": {
        "openAiApi": {
          "id": "REPLACE_CRED_ID_OPENAIAPI",
          "name": "OpenAi account"
        }
      }
    },
    {
      "parameters": {
        "url": "=http://GLPI_HOST:8088/apirest.php/Ticket/{{ $('Recherche ID Ticket').item.json.data[0]['2'] }}",
        "sendHeaders": true,
        "headerParameters": {
          "parameters": [
            {
              "name": "App-Token",
              "value": "REPLACE_WITH_GLPI_APP_TOKEN"
            },
            {
              "name": "Authorization",
              "value": "user_token REPLACE_WITH_GLPI_USER_TOKEN"
            },
            {
              "name": "Session-Token",
              "value": "={{ $('Get Token GLPI1').item.json.session_token }}"
            }
          ]
        },
        "options": {}
      },
      "id": "a9a96dd8-1736-4f7a-8847-f7ea65fda159",
      "name": "GET Ticket complet",
      "type": "n8n-nodes-base.httpRequest",
      "typeVersion": 3,
      "position": [
        752,
        384
      ]
    },
    {
      "parameters": {
        "method": "POST",
        "url": "http://OLLAMA_HOST:1234/v1/embeddings",
        "sendBody": true,
        "contentType": "raw",
        "rawContentType": "application/json",
        "body": "={{ JSON.stringify({ model: \"nomic-embed-text\", input: $('Code Validation').item.json.text }) }}",
        "options": {
          "response": {
            "response": {}
          }
        }
      },
      "type": "n8n-nodes-base.httpRequest",
      "typeVersion": 4.4,
      "position": [
        1232,
        384
      ],
      "id": "f82cb249-1009-491f-b6a4-f7210784003f",
      "name": "Ollama Embeddings"
    },
    {
      "parameters": {
        "jsCode": "const embedding = $json.data[0].embedding;\nconst data = $('Code Validation').item.json;\n\nreturn {\n  json: {\n    points: [{\n      id: data.ticket_id,\n      vector: embedding,  // tableau direct, plus de { dense: ... }\n      payload: {\n        content: data.text,\n        titre: data.titre,\n        serveur: data.serveur,\n        solution: data.text,\n        date: data.date,\n        duree_minutes: data.duree_minutes,\n        ticket_id: data.ticket_id\n      }\n    }]\n  }\n};"
      },
      "type": "n8n-nodes-base.code",
      "typeVersion": 2,
      "position": [
        1456,
        384
      ],
      "id": "f36b6b6c-77d5-4e2e-a7ef-daded1b77555",
      "name": "Format Qdrant"
    },
    {
      "parameters": {
        "jsCode": "// Gestion tableau ou objet direct\nconst raw = $input.first().json;\nconst ticket = Array.isArray(raw) ? raw[0] : raw;\n\nconst dureeMinutes = Math.round((ticket.solve_delay_stat || 0) / 60);\nconst contenu = ticket.content?.replace(/<[^>]*>/g, '').trim() || \"\";\nconst updater = ticket.users_id_lastupdater || 0;\n\nif (updater === 0) {\n  return [{ json: { eligible: false, raison: \"Aucun utilisateur impliqué\" } }];\n}\n\nif (dureeMinutes < 2) {\n  return [{ json: { eligible: false, raison: `Trop rapide: ${dureeMinutes} min` } }];\n}\n\nif (contenu.length < 100) {\n  return [{ json: { eligible: false, raison: \"Contenu trop court\" } }];\n}\n\n// Extraction du serveur depuis le contenu\nconst serveurMatch = contenu.match(/Serveur\\s*:\\s*(\\S+)/);\nconst serveur = serveurMatch ? serveurMatch[1] : $('Webhook').item.json.body.host || \"inconnu\";\n\n// Texte à vectoriser\nconst texte = `\nTitre: ${ticket.name}\nServeur: ${serveur}\nDurée résolution: ${dureeMinutes} minutes\nDiagnostic et solution:\n${contenu}\n`.trim();\n\nreturn [{\n  json: {\n    eligible: true,\n    text: texte,\n    ticket_id: ticket.id,\n    titre: ticket.name,\n    serveur: serveur,\n    duree_minutes: dureeMinutes,\n    date: ticket.closedate\n  }\n}];"
      },
      "type": "n8n-nodes-base.code",
      "typeVersion": 2,
      "position": [
        992,
        384
      ],
      "id": "b6ad6443-89c2-4f5d-81a0-f0613478fd33",
      "name": "Code Validation"
    },
    {
      "parameters": {
        "method": "PUT",
        "url": "http://QDRANT_HOST:6333/collections/glpi_incidents/points",
        "sendBody": true,
        "specifyBody": "json",
        "jsonBody": "={{ $json }}",
        "options": {}
      },
      "type": "n8n-nodes-base.httpRequest",
      "typeVersion": 4.4,
      "position": [
        1680,
        384
      ],
      "id": "7e74e865-7681-401e-ae17-1c0997cb93b1",
      "name": "Qdrant insert hybride"
    },
    {
      "parameters": {
        "mode": "retrieve-as-tool",
        "toolDescription": "Recherche des incidents similaires passés dans l'historique GLPI. \nUtilise ce tool pour trouver si un incident similaire a déjà été résolu, \nquelle solution avait été appliquée, ou détecter un pattern récurrent \nsur un serveur. Passe en paramètre le type d'alerte et le serveur concerné.",
        "qdrantCollection": {
          "__rl": true,
          "value": "glpi_incidents",
          "mode": "list",
          "cachedResultName": "glpi_incidents"
        },
        "topK": 3,
        "options": {}
      },
      "type": "@n8n/n8n-nodes-langchain.vectorStoreQdrant",
      "typeVersion": 1.3,
      "position": [
        1408,
        -112
      ],
      "id": "c50e1d18-377a-4198-a720-e0840e94bc99",
      "name": "Qdrant Vector Store",
      "credentials": {
        "qdrantApi": {
          "id": "REPLACE_CRED_ID_QDRANTAPI",
          "name": "QdrantApi account"
        }
      }
    },
    {
      "parameters": {
        "model": "nomic-embed-text:latest"
      },
      "type": "@n8n/n8n-nodes-langchain.embeddingsOllama",
      "typeVersion": 1,
      "position": [
        1408,
        64
      ],
      "id": "d0a389b1-34b2-4b61-b478-a036bb1cffa9",
      "name": "Embeddings Ollama",
      "credentials": {
        "ollamaApi": {
          "id": "REPLACE_CRED_ID_OLLAMAAPI",
          "name": "Ollama account"
        }
      }
    }
  ],
  "pinData": {},
  "connections": {
    "Webhook": {
      "main": [
        [
          {
            "node": "PROBLEM OR RESOLVED",
            "type": "main",
            "index": 0
          }
        ]
      ]
    },
    "Get Token GLPI": {
      "main": [
        [
          {
            "node": "Recherche ID",
            "type": "main",
            "index": 0
          }
        ]
      ]
    },
    "Recherche ID": {
      "main": [
        [
          {
            "node": "Loki Get Log",
            "type": "main",
            "index": 0
          }
        ]
      ]
    },
    "Création de ticket": {
      "main": [
        [
          {
            "node": "Zabbix Acquittement",
            "type": "main",
            "index": 0
          }
        ]
      ]
    },
    "Loki Get Log": {
      "main": [
        [
          {
            "node": "Tranformation Log",
            "type": "main",
            "index": 0
          }
        ]
      ]
    },
    "Tranformation Log": {
      "main": [
        [
          {
            "node": "Zabbix Get Metrics",
            "type": "main",
            "index": 0
          }
        ]
      ]
    },
    "AI Agent": {
      "main": [
        [
          {
            "node": "Création de ticket",
            "type": "main",
            "index": 0
          }
        ]
      ]
    },
    "PROBLEM OR RESOLVED": {
      "main": [
        [
          {
            "node": "Get Token GLPI",
            "type": "main",
            "index": 0
          }
        ],
        [
          {
            "node": "Get Token GLPI1",
            "type": "main",
            "index": 0
          }
        ]
      ]
    },
    "Get Token GLPI1": {
      "main": [
        [
          {
            "node": "Recherche ID Ticket",
            "type": "main",
            "index": 0
          }
        ]
      ]
    },
    "Recherche ID Ticket": {
      "main": [
        [
          {
            "node": "If",
            "type": "main",
            "index": 0
          }
        ]
      ]
    },
    "SOLUTION de ticket": {
      "main": [
        [
          {
            "node": "GET Ticket complet",
            "type": "main",
            "index": 0
          }
        ]
      ]
    },
    "If": {
      "main": [
        [
          {
            "node": "SOLUTION de ticket",
            "type": "main",
            "index": 0
          }
        ]
      ]
    },
    "Zabbix Get Metrics": {
      "main": [
        [
          {
            "node": "Prometheus Get Metrics",
            "type": "main",
            "index": 0
          }
        ]
      ]
    },
    "Prometheus Get Metrics": {
      "main": [
        [
          {
            "node": "Transformation Prometheus",
            "type": "main",
            "index": 0
          }
        ]
      ]
    },
    "Transformation Prometheus": {
      "main": [
        [
          {
            "node": "AI Agent",
            "type": "main",
            "index": 0
          }
        ]
      ]
    },
    "OpenAI Chat Model": {
      "ai_languageModel": [
        [
          {
            "node": "AI Agent",
            "type": "ai_languageModel",
            "index": 0
          }
        ]
      ]
    },
    "GET Ticket complet": {
      "main": [
        [
          {
            "node": "Code Validation",
            "type": "main",
            "index": 0
          }
        ]
      ]
    },
    "Ollama Embeddings": {
      "main": [
        [
          {
            "node": "Format Qdrant",
            "type": "main",
            "index": 0
          }
        ]
      ]
    },
    "Code Validation": {
      "main": [
        [
          {
            "node": "Ollama Embeddings",
            "type": "main",
            "index": 0
          }
        ]
      ]
    },
    "Format Qdrant": {
      "main": [
        [
          {
            "node": "Qdrant insert hybride",
            "type": "main",
            "index": 0
          }
        ]
      ]
    },
    "Qdrant Vector Store": {
      "ai_tool": [
        [
          {
            "node": "AI Agent",
            "type": "ai_tool",
            "index": 0
          }
        ]
      ]
    },
    "Embeddings Ollama": {
      "ai_embedding": [
        [
          {
            "node": "Qdrant Vector Store",
            "type": "ai_embedding",
            "index": 0
          }
        ]
      ]
    },
    "Qdrant insert hybride": {
      "main": [
        [
          {
            "node": "CLOSE de ticket",
            "type": "main",
            "index": 0
          }
        ]
      ]
    }
  },
  "active": true,
  "settings": {
    "executionOrder": "v1",
    "binaryMode": "separate",
    "availableInMCP": false
  },
  "versionId": "REPLACE_WITH_VERSION_ID",
  "meta": {
    "templateCredsSetupCompleted": true,
    "instanceId": "REPLACE_WITH_N8N_INSTANCE_ID"
  },
  "id": "REPLACE_WITH_WORKFLOW_ID",
  "tags": [
    {
      "updatedAt": "2025-12-06T09:22:38.748Z",
      "createdAt": "2025-12-06T09:22:38.748Z",
      "id": "hGv6rOLsBWXGiMz8",
      "name": "Zabbix"
    }
  ]
}
