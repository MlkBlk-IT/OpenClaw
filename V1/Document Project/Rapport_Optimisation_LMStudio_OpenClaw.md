RAPPORT TECHNIQUE

Optimisation d'une Architecture LLM Locale Hybride

LM Studio + OpenClaw — Infrastructure IA Locale/Hybride

Environnement : Windows 11 Pro — Dell Latitude 5420

Intel Core i7-1185G7 (4 cœurs / 8 threads) — 31.4 GB RAM — Intel Iris Xe

Préparé pour : Malik Belkacemi

Date : 5 août 2026

Document confidentiel — usage personnel / portfolio technique


# Table des matières



# 1. Contexte et objectif initial


Ce document retrace, de manière chronologique et complète, l'ensemble du travail d'audit, de diagnostic et d'optimisation réalisé sur l'infrastructure d'inférence locale de Malik Belkacemi, combinant LM Studio (serveur d'inférence local compatible OpenAI API) et OpenClaw (orchestrateur d'agents IA multi-fournisseurs).

L'objectif n'était pas d'obtenir un seul modèle local performant, mais de construire une architecture capable de sélectionner intelligemment le meilleur modèle selon le contexte : rapidité pour les tâches simples, raisonnement pour l'analyse complexe, spécialisation pour le code, et repli vers le cloud lorsque nécessaire.


## 1.1 Environnement matériel


Composant

Valeur

Machine

Dell Latitude 5420

Processeur

Intel Core i7-1185G7 @ 3.00 GHz — 4 cœurs physiques / 8 threads logiques

Mémoire vive

31.4 GB (33 706 872 832 octets)

GPU intégré

Intel Iris Xe (partage la RAM système, sans VRAM dédiée)

Système d'exploitation

Windows 11 Pro

Environnement de virtualisation

Hyper-V + WSL2 déjà configurés


## 1.2 Méthodologie de travail adoptée


La démarche suivie tout au long de ce projet a respecté un principe d'ingénierie strict, défini avant toute intervention technique :

Ne jamais modifier une configuration sans en comprendre l'état actuel.

Toujours commencer par un audit basé sur des données réelles, jamais des suppositions.

Séparer clairement les limitations matérielles, la configuration logicielle, les capacités des modèles et les besoins réels d'usage.

Optimiser l'architecture globale plutôt qu'un seul benchmark isolé.

Ne jamais déclarer une optimisation réussie sans preuve mesurée.


# 2. État initial du système (avant intervention)


Un premier audit automatisé (script PowerShell d'extraction) a révélé un système fonctionnel mais mal caractérisé : plusieurs hypothèses initiales, y compris certaines formulées avant le début de cette session, se sont avérées incomplètes ou inexactes une fois confrontées aux données réelles de la machine.


## 2.1 Hypothèses de départ vs réalité constatée


Hypothèse initiale

Réalité confirmée par audit

2 modèles GGUF présents (DeepSeek-14B, Llama-3.2-3B)

17 fichiers .gguf réellement présents, soit 12 modèles utilisables distincts

GPU offload potentiellement bénéfique

Intel Iris Xe (iGPU partagé) — offloadRatio à 0 sur tous les modèles, choix cohérent

Contexte configuré de façon homogène

Contexte par défaut à 32768 ou 131072 selon les modèles, sans lien avec l'usage réel

Threads CPU correctement dimensionnés

cpuThreadPoolSize absent de la configuration de la majorité des modèles (valeur par défaut du moteur utilisée)

Un seul port API actif (1234)

Deux ports actifs simultanément détectés (1234 fixe, 41343 dynamique) sur le même processus


## 2.2 Problème structurel de l'audit initial


Le tout premier script d'audit produisait un rapport contenant les noms des clés de configuration (ex. : llm.load.llama.cpuThreadPoolSize) sans jamais révéler leurs valeurs réelles — un piège méthodologique classique lorsqu'un audit se base sur une recherche de mots-clés plutôt que sur une extraction structurée du contenu JSON complet. Ce rapport contenait également, par erreur d'extraction, le contenu intégral d'un ancien fichier de conversation LM Studio, polluant le rapport avec des centaines de lignes sans rapport avec la configuration système.

Leçon retenue :

Un audit basé sur la présence de mots-clés n'est pas un audit de valeurs. Toute extraction de configuration doit cibler explicitement les champs "value", pas seulement les noms de clés ("key").


# 3. Processus d'audit — itérations successives


Face à l'insuffisance du premier audit, plusieurs scripts PowerShell ciblés ont été développés successivement pour extraire les données réelles, chacun corrigeant les limites du précédent.


## 3.1 Itération 1 — Extraction des fichiers de configuration principaux


Extraction complète (Get-Content -Raw, sans filtrage par mot-clé) de trois fichiers : settings.json, http-server-config.json, http-server.json. Cette extraction a confirmé :

Le contexte par défaut au chargement était fixé à 8192 tokens (defaultContextLength), et non la valeur maximale théorique du modèle.

Un seul modèle chargé à la fois en usage interactif (unloadPreviousModelOnSelect: true).

Le chargement à la demande (Just-In-Time) était actif pour l'API, avec déchargement automatique après 20 minutes d'inactivité (jitModelTTL: 1200s).

Un garde-fou de chargement (modelLoadingGuardrails: "high") empêchant LM Studio de charger un modèle dépassant les ressources disponibles.


## 3.2 Itération 2 — Découverte des valeurs custom réelles par modèle


Un second script a extrait le contenu JSON complet et non tronqué de chaque fichier de configuration par modèle (répertoire user-concrete-model-default-config). Cette extraction a révélé les valeurs personnalisées réellement enregistrées, confirmant l'ampleur du problème :

Modèle (config trouvée avant patch)

cpuThreadPoolSize

Contexte (load + prediction)

DeepSeek-R1-Distill-Qwen-14B *

absent

32768

Llama-3.2-3B-Instruct

absent

8192

Meta-Llama-3.1-8B-Instruct

absent

32768

Mistral-7B-Instruct-v0.3 *

absent

32768

Qwen2.5-Coder-14B-Instruct *

absent

32768

Qwen3-14B-abliterated *

4

n/a

Qwen2.5-7B-Instruct

absent

32768

* Modèles marqués d'un astérisque : configurations retrouvées comme orphelines plus tard dans le processus (modèles supprimés du disque entre-temps).


## 3.3 Itération 3 — Audit de vérité terrain (disque réel)


Face à des incohérences entre les caches JSON de LM Studio et l'état réel du parc de modèles, un script d'audit "vérité terrain" a été conçu pour partir exclusivement du contenu réel du répertoire de modèles (D:\AI\LMStudio\models), plutôt que des caches ou index internes potentiellement obsolètes.

Ce script a permis d'identifier :

17 fichiers .gguf réels, dont 5 fichiers mmproj (projecteurs de vision, non des modèles autonomes).

12 modèles réellement utilisables pour l'inférence.

7 fichiers de configuration "orphelins" — correspondant à des modèles supprimés du disque, dont la configuration persistait inutilement (notamment DeepSeek-R1-Distill-Qwen-14B et Qwen2.5-Coder-14B-Instruct, qui avaient été patchés par erreur avant cette découverte).

Incident corrigé en cours de projet :

Un premier cycle de patch a été appliqué à DeepSeek-14B et Qwen2.5-Coder-14B avant de vérifier que ces modèles étaient toujours présents sur le disque. Ils avaient en réalité été supprimés par l'utilisateur. Aucun impact négatif (fichiers de configuration orphelins, sans effet), mais ceci illustre l'importance de valider l'état réel du système avant toute action corrective, plutôt que de se fier à un audit antérieur.


# 4. Parc de modèles — inventaire et rôles assignés


Douze modèles GGUF ont été catalogués et classés par rôle fonctionnel, en fonction de leur taille, de leur spécialisation annoncée et de leur empreinte mémoire estimée.

Modèle

Taille disque

Rôle assigné

Llama-3.2-3B-Instruct

1.8 GB

Rapide / réponses courtes

Qwen2.5-7B-Instruct

4.15 GB

Rapide / généraliste secondaire

Meta-Llama-3.1-8B-Instruct

4.37 GB

Généraliste

Meta-Llama-3.1-8B-Instruct-abliterated

4.58 GB

Généraliste (non censuré)

Qwen3.5-4B-Instruct-Revised

2.52 GB

Rapide secondaire

Qwen3.5-4B-Kimi-Reasoning-Luna-LoRA

2.59 GB

Raisonnement léger

Qwen2.5-Coder-7B-Instruct-Ghidra-v2

4.36 GB

Codage / rétro-ingénierie

Phi-4-reasoning-plus

8.43 GB

Raisonnement principal

Qwen3-VL-8B-Instruct-abliterated

4.47 GB + 1.08 GB (mmproj)

Vision / multimodal

Gemma4-26B-A4B (MoE)

13.72 GB + 1.11 GB (mmproj)

Généraliste large, vision, tool-use

gemma-4-E4B-it (Q8_0, MTP)

0.09 GB

Modèle très léger

nomic-embed-text-v1.5

0.14 GB

Embeddings / RAG


## 4.1 Lacune identifiée


Aucun modèle de codage généraliste de taille intermédiaire (14B) n'était disponible au moment de l'audit final — le seul modèle de codage restant (Qwen2.5-Coder-7B-Instruct-Ghidra-v2) est une variante spécialisée en rétro-ingénierie (Ghidra), pas en développement logiciel généraliste. Ce point reste ouvert pour une évolution future du parc de modèles.


# 5. Optimisation de la configuration LM Studio



## 5.1 Paramètres retenus et justification technique


Trois paramètres critiques ont été identifiés comme sources principales d'inefficacité, et corrigés systématiquement sur l'ensemble du parc de modèles actifs.


### Nombre de threads CPU (cpuThreadPoolSize)


Valeur retenue : 4 threads, sur l'ensemble des modèles. Le processeur i7-1185G7 dispose de 4 cœurs physiques et 8 threads logiques (Hyper-Threading). Pour les charges de calcul intensif matriciel propres à l'inférence LLM, l'Hyper-Threading n'apporte généralement pas de gain significatif et peut même introduire de la contention de cache. Aligner le nombre de threads sur le nombre de cœurs physiques réels constitue la pratique recommandée pour llama.cpp sur ce type de matériel.


### Longueur de contexte (contextLength)


Les valeurs par défaut observées (32768, voire 131072 tokens pour certains modèles) dépassaient largement les besoins réels d'un usage conversationnel ou de codage courant, avec un coût direct en mémoire (cache KV) et en temps de traitement du prompt initial. Les valeurs ont été redimensionnées par rôle :

Catégorie de modèle

Contexte retenu

Justification

Rapide / généraliste

8192 tokens

Suffisant pour l'essentiel des échanges conversationnels et tâches courtes

Raisonnement (Phi-4) / codage

16384 tokens

Marge nécessaire pour les chaînes de raisonnement et le code plus long

Modèle très léger (Q8, MTP)

4096 tokens

Quantization lourde déjà coûteuse en mémoire par paramètre

Modèle d'embeddings (nomic-embed)

2048 tokens

Pas de génération de texte, encodage uniquement


### Flash Attention


L'existence de cette option sur le backend Vulkan utilisé (llama.cpp-win-x86_64-vulkan-avx2, version 2.27.1) n'a été confirmée que tardivement dans le processus, via inspection directe du panneau de configuration avancé de LM Studio — elle n'apparaissait dans aucun fichier de configuration JSON audité. Une fois localisée, elle s'est révélée déjà activée (ON) par défaut. Aucune action corrective n'a donc été nécessaire sur ce point.


### Répartition GPU (offloadRatio)


Valeur maintenue à 0 (aucun déport de calcul vers l'iGPU Intel Iris Xe) sur l'ensemble des modèles. L'Iris Xe étant un GPU intégré à mémoire partagée avec des capacités de calcul limitées, un traitement 100% CPU s'est avéré être le choix cohérent pour ce matériel, sans qu'un test comparatif chiffré n'ait été mené pour le confirmer formellement.


## 5.2 Script de correction appliqué


La correction a été appliquée via un script PowerShell structuré selon un principe de sécurité systématique : sauvegarde avant modification, écriture, puis validation de l'intégrité JSON après écriture — avec restauration automatique de la sauvegarde en cas d'échec de validation.

# Structure du script de patch (extrait de principe)foreach ($t in $targets) {    Copy-Item $t.Path $backupPath -Force        # 1. Sauvegarde    [System.IO.File]::WriteAllText($t.Path, $json, UTF8)  # 2. Ecriture    try {        Get-Content $t.Path -Raw | ConvertFrom-Json | Out-Null  # 3. Validation    } catch {        Copy-Item $backupPath $t.Path -Force    # 4. Restauration si echec    }}

Douze modèles ont été patchés selon cette méthode, avec sauvegarde systématique horodatée sur le bureau de l'utilisateur avant chaque intervention.


## 5.3 Découverte tardive — désynchronisation entre fichiers JSON et interface


Point d'attention technique important :

L'inspection visuelle du panneau de configuration LM Studio a révélé que la longueur de contexte réellement appliquée au modèle Phi-4-reasoning-plus restait à 32768 tokens, malgré la modification du fichier de configuration JSON sous-jacent. Ceci indique que LM Studio ne lit pas nécessairement le fichier JSON de configuration par modèle comme source unique de vérité pour tous les paramètres — certains réglages semblent gérés séparément au niveau de l'interface ou d'un état interne non identifié dans les fichiers audités. La correction finale a donc dû être appliquée directement via l'interface graphique pour ce paramètre, en complément du patch JSON.


# 6. Concurrence et parallélisme des requêtes


Un paramètre non anticipé initialement — le nombre de requêtes traitées en parallèle par modèle chargé (Max Concurrent Predictions) — a été identifié comme facteur de ralentissement potentiel.

La valeur par défaut observée était de 4 requêtes simultanées. Sur un processeur à 4 cœurs physiques, faire cohabiter 4 flux de génération réduit la part de calcul disponible pour chaque requête individuelle, ralentissant le temps de réponse perçu pour un usage interactif où une seule requête est traitée à la fois.

La valeur a été ramenée à 1 pour l'usage interactif standard, priorisant la latence d'une requête unique plutôt que le débit agrégé de requêtes multiples simultanées.


# 7. Intégration et correction OpenClaw



## 7.1 Architecture des agents


L'architecture OpenClaw repose sur deux agents distincts, définis dans la documentation de comportement (AGENTS.md) :

Agent

Rôle

Fournisseurs

main

Agent principal, sélection automatique du meilleur fournisseur/modèle, routage intelligent

Cloud (Mistral, Groq, Google, OpenRouter) en priorité, LM Studio en repli

local

Exécution 100% locale, invoqué explicitement (@local) ou pour données sensibles

LM Studio exclusivement, aucun repli cloud


## 7.2 Incohérence critique détectée et corrigée — désalignement de contexte


Problème structurel majeur identifié :

Les valeurs de fenêtre de contexte déclarées côté OpenClaw (contextWindow / contextTokens, ex. : 131072 tokens pour Llama-3.2-3B) ne correspondaient pas aux valeurs réellement configurées côté LM Studio (8192 tokens pour ce même modèle) — un facteur d'écart allant jusqu'à 16x selon les modèles. Cette désynchronisation exposait le système à des erreurs silencieuses : OpenClaw calculant sa gestion de contexte et de troncature d'historique sur la base de valeurs erronées, avec un risque de troncature imprévisible des réponses par LM Studio en cours de génération.

Correction appliquée : alignement systématique, par script PowerShell, des champs contextWindow et contextTokens de chaque modèle déclaré côté OpenClaw sur la valeur réelle configurée côté LM Studio.


## 7.3 Correction du point de terminaison réseau (baseUrl)


Une vérification par la commande Get-NetTCPConnection a révélé que le processus LM Studio écoutait simultanément sur deux ports distincts : le port fixe configuré (1234) et un port dynamique additionnel (41343), tous deux rattachés au même identifiant de processus.

Un premier script de correction a par erreur bindé la configuration OpenClaw sur le port dynamique (41343), susceptible de changer à chaque redémarrage de LM Studio. Cette erreur a été identifiée et corrigée immédiatement : la configuration finale pointe exclusivement vers le port fixe 1234, cohérent avec le fichier de configuration http-server-config.json et avec la documentation de connaissance interne (lmstudio.md).


## 7.4 Autres corrections appliquées côté OpenClaw


Activation du préchargement (preload: true) pour le fournisseur LM Studio, afin de réduire la latence perçue lors du premier appel à un modèle.

Retrait des références mortes au modèle DeepSeek-R1-Distill-Qwen-14B (supprimé du disque), à la fois dans la liste des modèles déclarés et dans les chaînes de repli (fallbacks) des deux agents.

Ajout du modèle de codage spécialisé (Qwen2.5-Coder-7B-Instruct-Ghidra-v2) dans la chaîne de repli de l'agent local, absent initialement malgré sa pertinence pour les tâches de développement.


# 8. Surveillance proactive — activation du Heartbeat


Le fichier de configuration HEARTBEAT.md, présent dans l'architecture documentaire OpenClaw mais laissé vide/désactivé, a été activé avec une définition complète de cinq vérifications automatisées, directement motivée par les incidents rencontrés au cours du projet (notamment le désalignement de port).

Vérification

Objectif

Serveur LM Studio (port 1234)

Détecter une indisponibilité de l'API locale avant qu'elle n'affecte une requête utilisateur

Passerelle OpenClaw (port 18789)

Confirmer que le point d'entrée principal d'OpenClaw répond

Espace disque (C: et D:)

Alerte sous le seuil de 15 GB libres

Santé des fournisseurs cloud

Détection d'indisponibilité sans blocage (repli existant)

Utilisation de la mémoire vive

Alerte au-delà de 90% d'utilisation, risque de ralentissement des modèles locaux

Un test immédiat des cinq vérifications a été exécuté après activation, confirmant un état sain du système : serveur LM Studio répondant correctement, passerelle active, 68.9 GB libres sur C:, 100.6 GB libres sur D:, utilisation mémoire à 28.3%.


# 9. Incident critique — blocage de génération et résolution


Lors de la phase de validation par mesure de performance réelle, un incident bloquant est survenu : toute tentative de génération de réponse, sur plusieurs modèles distincts (Phi-4-reasoning-plus, Qwen3-VL-8B), restait figée indéfiniment à l'état "Processing Prompt... 0.00%", sans jamais produire de contenu, y compris après plusieurs minutes d'attente.


## 9.1 Diagnostic


L'inspection du Gestionnaire des tâches Windows a révélé une utilisation CPU anormalement basse (21%) pendant le blocage, alors qu'une charge de traitement de prompt normale aurait dû solliciter fortement les 4 cœurs configurés. Ce signal indiquait que le processus n'était pas en calcul actif, mais en attente bloquante.

L'analyse des journaux du serveur de développement LM Studio (Developer Logs) a révélé la cause réelle : deux extensions installées par défaut — js-code-sandbox (bac à sable d'exécution JavaScript) et rag-v1 (recherche documentaire augmentée) — étaient actives et interféraient avec le pipeline de génération. Les journaux montraient un cycle répété de connexion/déconnexion du plugin js-code-sandbox juste avant chaque tentative de traitement de prompt, sans jamais aboutir à un traitement effectif.


## 9.2 Résolution


La désactivation de ces deux extensions via leur bouton d'activation dans l'interface de chat s'est révélée insuffisante — les journaux continuaient de montrer leur enregistrement actif au niveau du serveur, suggérant un état persistant indépendant du panneau de contrôle par conversation.

La résolution définitive a consisté à désactiver ces deux extensions au niveau du système de fichiers, en renommant leurs répertoires d'installation respectifs pour empêcher tout chargement par LM Studio, indépendamment de l'état affiché dans l'interface :

# Neutralisation des plugins bloquantsRename-Item '...\plugins\lmstudio\js-code-sandbox' '...\_disabled_js-code-sandbox'Rename-Item '...\plugins\lmstudio\rag-v1' '...\_disabled_rag-v1'

Résultat confirmé :

Après cette correction, un nouveau chat créé dans LM Studio (Llama-3.2-3B-Instruct) a produit une réponse complète et correcte, avec un débit mesuré de 9.78 tokens/seconde en génération. Le modèle Qwen3-VL-8B-Instruct-abliterated a également confirmé un comportement de génération normal dans les mêmes conditions.


# 10. Mesure de performance — résultat chiffré


Conformément au principe méthodologique de ne jamais déclarer une optimisation sans preuve mesurée, un test de génération réel a été exécuté après résolution de l'incident du chapitre précédent.

Paramètre du test

Valeur

Modèle testé

Llama-3.2-3B-Instruct (Q4_K_S)

Configuration appliquée

4 threads CPU, contexte 8192, concurrence 1, Flash Attention actif

Prompt

"Explique-moi la différence entre TCP et UDP"

Tokens générés

100

Débit mesuré (génération)

9.78 tokens/seconde

Ce débit est cohérent avec les attentes pour un modèle de 3 milliards de paramètres, quantifié en Q4_K_S, exécuté en calcul CPU pur sur un processeur mobile à 4 cœurs physiques. Aucune mesure comparative "avant optimisation" n'a pu être établie de façon rigoureuse, l'incident de blocage ayant empêché toute génération fonctionnelle avant la résolution des plugins — ce point constitue une limite reconnue de cette validation.


# 11. Synthèse des actions correctives


#

Action

Statut

1

Correction du nombre de threads CPU (4, aligné sur les cœurs physiques)

Complété — 12 modèles

2

Réduction et adaptation du contexte par rôle de modèle

Complété — 12 modèles

3

Vérification de Flash Attention (déjà actif)

Confirmé

4

Réduction de la concurrence de requêtes (4 → 1)

Complété

5

Alignement des contextes déclarés OpenClaw sur LM Studio

Complété — 12 modèles

6

Correction du point de terminaison réseau (port fixe 1234)

Complété

7

Activation du préchargement des modèles côté OpenClaw

Complété

8

Retrait des références à un modèle supprimé du disque

Complété

9

Ajout du modèle de codage au routage de l'agent local

Complété

10

Activation de la surveillance automatisée (Heartbeat)

Complété — 5 vérifications

11

Neutralisation des extensions bloquant la génération

Complété — 2 extensions

12

Validation par mesure réelle du débit de génération

Confirmé — 9.78 tokens/s


## 11.1 Points laissés ouverts pour évolution future


Absence de mesure comparative rigoureuse avant/après sur l'ensemble du parc de modèles — seul Llama-3.2-3B a fait l'objet d'une mesure confirmée.

Aucun test comparatif n'a été mené sur l'intérêt réel d'un déport partiel vers l'iGPU Intel Iris Xe (offloadRatio > 0).

Absence, dans le parc actuel, d'un modèle de codage généraliste de taille intermédiaire (14B) — le seul modèle de codage restant est spécialisé en rétro-ingénierie.

Les clés d'API (Groq, Mistral, OpenRouter, Google, jeton du robot Telegram) ont été exposées en clair au cours des échanges de travail — une régénération de ces clés a été recommandée par mesure de précaution.


# 12. Conclusion


L'architecture LM Studio + OpenClaw a été auditée, corrigée et partiellement validée selon une méthodologie rigoureuse d'ingénierie système : caractérisation de l'état réel avant toute intervention, correction ciblée et documentée de chaque paramètre, et confirmation par la mesure lorsque cela a été possible.

Le projet a également mis en évidence l'importance de la vérification continue de l'état réel du système face aux hypothèses de travail — plusieurs corrections appliquées en début de processus se sont révélées porter sur des éléments obsolètes (modèles supprimés, ports incorrects), corrigées dès leur détection sans impact négatif sur le système final.

L'infrastructure est, à l'issue de ce travail, dans un état fonctionnel confirmé : serveur LM Studio opérationnel, génération de texte fonctionnelle et mesurée, configuration OpenClaw alignée et cohérente, et surveillance automatisée en place pour détecter plus rapidement toute régression future.

— Fin du rapport —
