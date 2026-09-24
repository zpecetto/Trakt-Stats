# Trakt-Stats

Statistiques de films et séries avec n8n pour le site **[Trakt](https://app.trakt.tv/)**. Le workflow lit les données de votre profil avec l’API Trakt, puis les traite dans les Data Tables et Google Sheets.

## Workflow

[Films et séries.json](Workflow/Films%20et%20s%C3%A9ries.json) contient 45 nœuds.

Les deux branches commencent à **Loop Over Items5** et **Loop Over Items6**. Elles reconstruisent les tables Film et Serie, récupèrent les statistiques, listes à voir et historiques Trakt, puis calculent durées, genres, pays, décennies, minima, maxima et moyennes. Les nœuds Sheets et la notification Telegram de cette partie sont conservés.

## Configuration du service

Remplacer `YOUR_TRAKT_USERNAME` dans les URL et la clé Trakt dans les en-têtes des quatre nœuds HTTP. Configurer les credentials d’authentification si votre compte les exige et le destinataire Telegram. Les catégories sans année et sans pays sont conservées.

## API utilisées pour les films et séries

### API Trakt

Le site utilisateur est [app.trakt.tv](https://app.trakt.tv/). Les quatre nœuds HTTP interrogent l’API JSON à l’adresse `https://api.trakt.tv`, avec la version `2` dans les en-têtes.

| Nœud n8n | Requête GET, après l’URL de base | Utilisation dans le workflow |
|---|---|---|
| Trakt API - Get Stats | `/users/{username}/stats` | Statistiques globales du profil |
| Trakt API - Get Stats1 | `/users/{username}/watchlist?extended=full` | Films et séries de la liste à voir |
| Trakt API - Get Stats2 | `/users/{username}/history/movies?extended=full` | Historique des visionnages de films |
| Trakt API - Get Stats3 | `/users/{username}/history/episodes?extended=full` | Historique des épisodes vus, ensuite regroupés par série |

Remplacer `YOUR_TRAKT_USERNAME` dans les quatre URL par le nom d’utilisateur Trakt. Les calculs de durée, genres, pays et périodes sont ensuite effectués par les nœuds Code. Les identifiants IMDb, TMDB et TVDB sont des champs des réponses Trakt.

**Clé et authentification :** créer une application depuis votre compte Trakt, puis remplacer `YOUR_TRAKT_API_KEY` par son **Client ID** dans l’en-tête `trakt-api-key`. Les autres en-têtes de l’export sont `trakt-api-version: 2` et `Content-Type: application/json`.

Si l’accès au profil nécessite une autorisation OAuth, configurer les credentials correspondants dans n8n et l’en-tête `Authorization: Bearer <access_token>`. Le Client ID identifie l’application ; le jeton OAuth autorise l’accès au compte. L’export ne contient aucun jeton et n’inclut pas de parcours de connexion ou de renouvellement OAuth.

**Pagination et détails :** les nœuds de liste à voir et d’historique font avancer `page` avec `{{ $pageCount + 1 }}`. Le nœud des épisodes définit aussi `limit=100`. Le paramètre `extended=full` demande les informations détaillées exploitées par les calculs. Vérifier la récupération de toutes les pages lors de la première exécution, notamment sur une grande collection.

Documentation officielle : [API Trakt](https://docs.trakt.tv/), [en-têtes](https://docs.trakt.tv/docs/required-headers) et [authentification / création d’application](https://docs.trakt.tv/reference/auth).

### API de sortie

- **[Google Sheets API](https://developers.google.com/sheets/api)** : les huit nœuds Google Sheets ajoutent ou mettent à jour les lignes de statistiques dans l’onglet `N8N`. Sélectionner les credentials Google Sheets et un document auquel ce compte a accès.
- **[Telegram Bot API](https://core.telegram.org/bots/api#sendmessage)** : le nœud **Send a text message2** envoie la notification via `sendMessage`. Configurer les credentials du bot et l’identifiant du destinataire.

Les Data Tables sont stockées dans n8n. Le nœud commun **HTTP Request1** contrôle l’accessibilité de votre instance avant la collecte.

## Démarrage et configuration

Chaque extraction planifiée reprend le début d’Ultime : **Schedule Trigger**, **Date & Time**, **Configuration Globale**, **Loop Over Items4**, **HTTP Request1**, **If5**. La branche d’échec conserve **Restauration Tunnel1**, **Wait1** et son retour dans la boucle.

1. Importer le JSON dans n8n. L’export reste désactivé tant que la configuration n’est pas terminée.
2. Remplacer l’URL `https://YOUR_N8N_HOST.example.invalid/` de **HTTP Request1** par celle de votre instance. Le succès est déterminé par un HTTP 200.
3. Reconnecter les credentials SSH du nœud de restauration et adapter la commande ngrok à votre installation. Sans tunnel, remplacer cette commande par votre propre mécanisme de restauration ou retirer explicitement cette branche.
4. Adapter l’horaire du planificateur : l’export reprend le vendredi à 17 h. Choisir le fuseau horaire de l’instance ou du workflow.
5. Remplacer les champs `YOUR_...`, renseigner les cookies privés lorsque nécessaires et sélectionner les credentials des services utilisés.
6. Créer les tables décrites dans [DataTables](DataTables/README.md), puis les sélectionner dans chaque nœud Data Table. Les CSV sont vides, avec leurs seuls en-têtes.
7. Pour les sorties Sheets, créer un onglet **N8N** à partir de [GoogleSheets/N8N.csv](GoogleSheets/N8N.csv), sélectionner votre document et vos credentials dans tous les nœuds Sheets. Les intitulés des colonnes et les colonnes de correspondance doivent rester identiques.
8. Vérifier une exécution complète avant activation. Les branches de collecte peuvent vider et reconstruire leurs tables cibles ; utiliser des tables dédiées.

Les identifiants de documents, tables, dossiers, comptes, cookies, tokens, données épinglées et historiques d’exécution personnels ont été retirés. Les identifiants internes des nœuds ont été régénérés. Les workflows séparés et le workflow complet sont des alternatives : éviter de lancer simultanément plusieurs versions qui reconstruisent les mêmes tables ou contrôlent le même tunnel.

## Vérification

Le JSON, les connexions, les références entre nœuds, la syntaxe JavaScript, les expressions complètes et les entrées des nœuds Merge ont été contrôlés localement. Aucun service personnel ni workflow de production n’a été exécuté. Les credentials et l’intégration réelle doivent être vérifiés dans l’instance cible.
