ARCHITECTURE HYBRIDE IA

Assistant Personnel Local et Cloud via OpenClaw

Dell Latitude 5420  |  Intel Core i7 vPro  |  Iris Xe  |  32 Go RAM  |  512 Go NVMeWindows 11 Pro  |  LM Studio  |  Budget projet : 100 Go  |  2 aout 2026

________________________________________________________________________________

1. Introduction et Objectifs

Ce document presente la conception detaillee d'une architecture hybride d'intelligence artificielle destinee a un environnement professionnel de securite informatique. Il synthetise les analyses prealables, les choix techniques et le plan de deploiement en une ressource unique et actionnable.

L'architecture repose sur deux objectifs distincts mais complementaires :

Objectif operationnel : disposer d'un assistant IA personnel joignable depuis un telephone via Telegram ou l'application Kimi, capable d'agir reellement sur le poste de travail. L'assistant lit des fichiers, effectue des recherches sur le web, redige des documents, execute des commandes systeme et conserve le contexte d'une session a l'autre grace a une memoire persistante.

Objectif pedagogique : comprendre par la pratique le fonctionnement d'une architecture d'agent IA moderne, incluant l'orchestration entre sources de raisonnement local et cloud, la gestion de fallback entre fournisseurs, les permissions d'execution et la memoire vectorielle.

Le poste de travail retenu est un Dell Latitude 5420 equipe d'un processeur Intel Core i7 vPro, de 32 Go de memoire vive et d'un stockage NVMe de 512 Go. L'absence de carte graphique dediee impose l'execution des modeles locaux sur CPU uniquement, ce qui guide le choix des modeles et des formats de quantification.

2. Justification du Choix d'OpenClaw

La selection d'OpenClaw comme orchestrateur central resulte d'une analyse comparative des solutions disponibles. Plutot que de developper un agent sur mesure, ce projet s'appuie sur un outil open-source deja concu pour ce cas d'usage, reduisant le temps de mise en service et la charge de maintenance.

Criteres de selection :

Gratuit et open-source : le code est public sur GitHub, aucune licence commerciale n'est requise, et la communaute assure un suivi actif des evolutions.

Orchestration multi-provider native : le routage automatique entre modeles locaux et fournisseurs cloud, avec bascule configurable en cas d'indisponibilite, est integre sans developpement middleware supplementaire.

Execution d'actions reelles : OpenClaw agit comme un operateur systeme, capable de lire et d'ecrire des fichiers, d'executer des commandes PowerShell et d'interagir avec le systeme d'exploitation, sous reserve d'une configuration explicite des permissions.

Memoire persistante integree : le contexte des conversations, les plans d'action et les resultats sont conserves dans une structure de memoire basee sur des fichiers Markdown et YAML, assurant la continuite entre les sessions.

Integration messagerie native : la connexion avec Telegram, Discord, Slack, WhatsApp, Signal et iMessage est prise en charge nativement. L'application Kimi supporte egalement OpenClaw sans integration additionnelle.

Compatibilite LM Studio : l'agent se connecte a un serveur LM Studio local via une simple URL, sans necessiter de code d'integration specifique.

Consequence pratique : ce projet n'est pas un developpement logiciel mais un projet de configuration. L'essentiel du travail technique est deja resolu par l'outil, ce qui permet de se concentrer sur le parametrage, la securisation et l'optimisation.

3. Architecture Generale du Systeme

Le systeme repose sur une segmentation fonctionnelle claire. Les taches cognitives lourdes, necessitant un contexte etendu ou un raisonnement complexe, sont deleguees aux services cloud. L'execution d'actions sur la machine locale, l'acces aux fichiers sensibles, la preservation de la confidentialite et la memorisation persistante sont assures par l'infrastructure interne.

Schema de flux de donnees :

UTILISATEUR (telephone)

Telegram  |  Kimi App

|

v

+---------------------+

|   OPENCLAW GATEWAY   |

|  Orchestration        |

|  Execution d'outils   |

|  Memoire persistante  |

+----------+----------+

|

+----------------+----------------+

|                                 |

v                                 v

+-------------------+          +----------------------+

|   LM STUDIO        |          |   FOURNISSEURS CLOUD   |

|   (local)           |          |   (gratuits)           |

|   Modeles GGUF      |          |   Cerebras, Groq,      |

|   deja installes    |          |   OpenRouter, Mistral, |

|                     |          |   GitHub Models        |

+-------------------+          +----------------------+

|                                 |

+----------------+----------------+

|

v

Reponse ou action executee

Resultat renvoye a l'utilisateur

ISOLATION : Le lab Hyper-V reste strictement isole. OpenClaw

tourne sur le systeme hote Windows 11 Pro. Aucune connexion

reseau n'est etablie entre l'agent et le segment du lab.

Principe de routage : OpenClaw selectionne le provider selon la nature de la requete. Les taches privees, courtes ou necessitant une action locale restent sur le laptop. Les taches necessitant une recherche web ou un raisonnement lourd sont envoyees au cloud. La decision est prise selon la configuration etablie lors de l'installation.

4. Infrastructure Locale

4.1. Moteur d'inference : LM Studio

LM Studio est la plateforme d'execution des modeles de langage au format GGUF. Elle expose une API compatible OpenAI sur le port 1234 de la machine locale, permettant a OpenClaw de solliciter les modeles charges en memoire via des requetes HTTP standard. Le format de quantification Q4_K_M est retenu pour offrir le meilleur compromis entre qualite de sortie et consommation de ressources sur CPU.

4.2. Modeles de langage deployes

Modele

Parametres

RAM (Q4_K_M)

Debit CPU

Licence

Fonction principale

Qwen3-Coder-14B-Abliterated (bartowski)

14 milliards

9 Go

6 a 12 tokens/s

Apache 2.0

Generation de code, scripting, raisonnement technique en francais

Meta-Llama-3.1-8B-Instruct-Abliterated (mlabonne)

8 milliards

5 a 6 Go

10 a 18 tokens/s

Llama 3.1

Traitement general, dialogue, taches courantes

Qwen3.5-4B-Instruct

4 milliards

3 a 4 Go

15 a 25 tokens/s

Apache 2.0

Traitement rapide, resume, requetes simples

Phi-4-Mini-3.8B-Instruct

3,8 milliards

3 Go

15 a 25 tokens/s

MIT

Mode de secours ultra-leger, toujours charge en memoire

La charge memoire totale des modeles actifs simultanement est estimee entre 18 et 22 Go, laissant une marge de 10 a 14 Go pour le systeme d'exploitation, les outils de securite et les machines virtuelles Hyper-V.

Note sur les modeles abliterated : ces modeles ont leurs refus retires par modification directe des poids du reseau, et non par un simple ajustement d'instructions. Leur comportement sur les cas limites est moins encadre que celui des modeles standards ; l'usage reste sous la responsabilite de l'utilisateur.

4.3. Base de donnees vectorielle

Une instance ChromaDB hebergee localement assure le stockage et la recherche semantique des embeddings. Les documents indexes, les resultats d'execution et l'historique des conversations sont vectorises a l'aide du modele all-MiniLM-L6-v2 et conserves sur le volume dedie D:\OpenClaw\vector_db. L'empreinte disque est estimee entre 5 et 10 Go.

5. Fournisseurs de Services Cloud

Cinq fournisseurs sont integres au systeme selon une hierarchie de priorite fondee sur le volume de requetes autorise, la vitesse de reponse et la politique de confidentialite des donnees.

Rang

Fournisseur

Modeles disponibles

Limites d'utilisation

Compatible OpenAI

Entrainement sur donnees

1

Cerebras

Llama 3.3 70B

30 RPM, 1 million tokens/jour

Oui

Non

2

Groq

Llama 3.3 70B, Qwen3-32B, Mixtral

30 RPM, 1 000 a 14 400 RPD

Oui

Non

3

OpenRouter

Qwen, DeepSeek, Llama, Gemma (28+)

20 RPM, 50 RPD (1 000 avec 10 USD)

Oui

Non

4

Mistral AI

Mistral Small, Large, Codestral

1 milliard tokens/mois, 2 RPM

Oui

Oui (experimental)

5

GitHub Models

GPT-4o, Claude 3.5, Llama, Phi (100+)

15 RPM, 150 a 1 000 RPD

Oui

Non

Option

Google AI Studio

Gemini Flash, Gemma 3/4

1 500 requetes/jour

Oui

Oui (hors UE)

Strategie de fallback : en cas d'epuisement du quota ou d'indisponibilite d'un fournisseur, le systeme bascule automatiquement vers le fournisseur de rang immediatement inferieur. Le modele local Phi-4-Mini constitue le dernier recours en cas d'indisponibilite totale des services externes.

Google AI Studio est exclu de la selection principale car la plateforme entraine sur les donnees des utilisateurs situes hors de l'Union europeenne, de l'Espace economique europeen, de la Suisse et du Royaume-Uni.

6. Exemples Concrets d'Utilisation

Exemple 1 — Synthese documentaire

Requete Telegram : 'Fais-moi une synthese a jour 2026 sur [sujet], en te basant sur mes documents C:/dossier/doc1.pdf et doc2.pdf, avec verification des sources sur internet.'

Deroulement : OpenClaw lit les deux fichiers localement, lance une recherche web via un fournisseur cloud, redige la synthese, sauvegarde le resultat sur le laptop et indexe le tout dans la memoire pour une requete future.

Exemple 2 — Action systeme

Requete Telegram : 'Verifie l'espace disque disponible sur le laptop et liste les dix plus gros fichiers du dossier Telechargements.'

Deroulement : commande PowerShell generee et executee localement, resultat renvoye directement sur Telegram sans passage par le cloud, aucune donnee personnelle transmise a l'exterieur.

Exemple 3 — Recherche et redaction a distance

Requete Kimi, en deplacement : 'Resume les nouveautes de la semaine sur [technologie], et enregistre le resultat dans mes notes de projet.'

Deroulement : recherche cloud, redaction, ecriture locale du fichier de notes, sans que l'utilisateur ait besoin d'etre devant le laptop.

Exemple 4 — Continuite de memoire

Requete une semaine plus tard : 'Rappelle-moi ce qu'on avait trouve sur [sujet] la derniere fois.'

Deroulement : l'agent interroge sa memoire vectorielle locale, indexee lors de l'echange precedent, et repond sans relancer de recherche.

7. Plan de Deploiement

L'ensemble de la mise en service est prevu en une seule session d'environ quatre-vingt-dix minutes a deux heures.

Etape

Action

Duree

1

Verifier que LM Studio tourne (http://127.0.0.1:1234) et noter les modeles charges

5 min

2

Installer OpenClaw (npm install -g openclaw@latest, puis openclaw onboard --install-daemon)

10 min

3

Configurer le backend local (URL LM Studio + identifiants des modeles) via l'assistant interactif

15 min

4

Creer les comptes Cerebras, Groq, OpenRouter, Mistral, GitHub Models et ajouter les cles API

15-20 min

5

Creer le bot Telegram via @BotFather, recuperer le token, appairer avec OpenClaw

5 min

6

Verifier l'integration OpenClaw deja presente sur Kimi

Deja en place

7

Activer les permissions de fichiers et commandes, restreintes a des repertoires precis

10 min

8

Test complet : lecture fichier local, recherche web, redaction, sauvegarde

15-20 min

8. Mesures de Securite

La capacite d'OpenClaw a executer des commandes systeme impose la mise en oeuvre de controles stricts des la configuration initiale.

Principe du moindre privilege : les competences d'action sont activees selectivement. Seuls les repertoires explicitement declares sont accessibles en lecture et en ecriture. L'acces au disque complet est explicitement refuse.

Isolation reseau : aucune interface reseau de l'agent ne penetre le segment du laboratoire Hyper-V. Les machines virtuelles restent invisibles depuis le poste hote pour ce qui concerne le trafic de l'agent.

Protection des cles d'API : les jetons d'authentification sont conserves dans le fichier de configuration locale, chiffre si possible, et ne sont jamais transmis a des tiers ni exposes dans les journaux d'execution.

Journalisation : toutes les actions systeme declenchees par l'agent sont consignees avec horodatage, identifiant de requete et contenu de la commande, facilitant l'audit en cas d'incident.

Sandboxing des scripts : l'execution PowerShell s'effectue dans un contexte restreint, avec validation prealable du contenu du script contre une liste de motifs autorises. Pour toute commande generee automatiquement, une validation manuelle est requise tant que la confiance dans le comportement de l'agent n'est pas etablie.

Revue periodique : les permissions accordées sont revues a mesure que l'usage evolue, afin de maintenir la surface d'attaque au minimum.

9. Budget des Ressources

L'enveloppe de stockage allouee au projet est de 100 Go sur le volume D:. La repartition previsionnelle est la suivante :

Modeles de langage (4 fichiers GGUF)     :  30 a 35 Go

Base de donnees vectorielle et embeddings :   5 a 10 Go

Journaux d'execution et cache local       :   5 a 10 Go

OpenClaw et dependances                   :   2 a  5 Go

Scripts de configuration et outils        :   1 a  2 Go

Marge de securite                         :  40 a 50 Go

---------------------------------------------------------

Total estime                              :  83 a 112 Go

Enveloppe allouee                         :     100 Go

Statut                                    :  Realisable

La consommation memoire vive est estimee entre 18 et 22 Go pour les modeles actifs, laissant 10 a 14 Go disponibles pour le systeme d'exploitation et les outils de securite. Ce ratio est juge confortable pour un fonctionnement stable et prolonge.

10. Recapitulatif

INFRASTRUCTURE LOCALE (LM Studio)

- Qwen3-Coder-14B-Abliterated  :  Code, scripting, raisonnement technique

- Llama-3.1-8B-Instruct      :  Traitement general, dialogue

- Qwen3.5-4B-Instruct        :  Traitement rapide, taches courantes

- Phi-4-Mini-3.8B-Instruct   :  Mode de secours, toujours charge

FOURNISSEURS CLOUD (Gratuits, ordre de priorite)

1. Cerebras        :  1 million tokens/jour, haut debit

2. Groq            :  Reponse ultra-rapide, ideal code

3. OpenRouter      :  Diversite de modeles, unification d'API

4. Mistral AI      :  Volume mensuel eleve

5. GitHub Models   :  Acces aux modeles frontieres

CANAUX D'INTERACTION

- Telegram (OpenClaw natif)

- Application Kimi (integration existante)

BASE DE DONNEES

- ChromaDB locale sur D:\OpenClaw\vector_db

- Embeddings : all-MiniLM-L6-v2

ORCHESTRATION

- OpenClaw avec routage automatique et fallback en cascade

ENVELOPPE RESSOURCES

- Stockage : 100 Go SSD  |  Consommation estimee : 83-112 Go

- Memoire  :  32 Go RAM  |  Consommation estimee : 18-22 Go

DELAI DE MISE EN SERVICE ESTIME

- Quatre-vingt-dix minutes a deux heures en une session unique

11. Prochaines Etapes

Avant de proceder a l'installation, deux decisions restent a arbitrer :

Confirmation du parc de modeles : faut-il conserver les modeles actuellement charges dans LM Studio, ou remplacer l'un d'eux par la version Qwen3-14B abliterated pour beneficier d'une capacite de raisonnement accrue ?

Fourniture du kit de configuration : sur validation du parc de modeles, fournir la commande d'installation d'OpenClaw et le fichier de configuration JSON complet, pre-rempli avec les cinq fournisseurs cloud et les parametres LM Studio, pret a etre deploye.

Annexe — Sources Consultees

OpenClaw — fonctionnement et installation

Documentation officielle : docs.openclaw.ai/install

Package npm : npmjs.com/package/openclaw

Guide d'installation complet (AnotherWrapper, mars 2026) : anotherwrapper.com/blog/how-to-install-openclaw

Guide d'installation Docker/npm/script (BetterLink Blog, juin 2026) : eastondev.com

Modeles locaux abliterated

Guide des modeles abliterated 2026 (DEV Community, avril 2026) : dev.to/purpledoubled

Qwen3-14B-abliterated-GGUF (Hugging Face, richardyoung) : huggingface.co/richardyoung/Qwen3-14B-abliterated-GGUF

Qwen3-14B-abliterated-GGUF (Hugging Face, mlabonne via bartowski) : huggingface.co/bartowski/mlabonne_Qwen3-14B-abliterated-GGUF

Comparatif modeles CPU-only 2026 (PromptQuorum, juillet 2026) : promptquorum.com/local-llms/best-cpu-only-llm

Fournisseurs cloud gratuits

Comparatif des API LLM gratuites 2026 (OpenRouter Blog, juin 2026) : openrouter.ai/blog/tutorials/free-llm-apis-compared

Comparatif rate limits et quotas (AgentDeals, mars 2026) : agentdeals.dev/free-llm-apis

Liste des API gratuites permanentes (GitHub) : github.com/amardeeplakshkar/awesome-free-llm-apis

Comparatif fournisseurs gratuits 2026 (Klymentiev, juin 2026) : klymentiev.com/blog/free-llm-api

______________________________________________________________________

Document de conception final — Architecture Hybride IA — 2 aout 2026
