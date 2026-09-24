# Trakt-Stats

Statistiques films et séries via Trakt, avec n8n.

## Workflow

[Films et séries.json](Workflow/Films%20et%20s%C3%A9ries.json) contient 45 nœuds.

Les deux branches commencent à **Loop Over Items5** et **Loop Over Items6**. Elles reconstruisent les tables Film et Serie, récupèrent les statistiques, listes à voir et historiques Trakt, puis calculent durées, genres, pays, décennies, minima, maxima et moyennes. Les nœuds Sheets et la notification Telegram de cette partie sont conservés.

## Configuration du service

Remplacer `YOUR_TRAKT_USERNAME` dans les URL et la clé Trakt dans les en-têtes des quatre nœuds HTTP. Configurer les credentials d’authentification si votre compte les exige et le destinataire Telegram. Les catégories sans année et sans pays sont conservées.

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
