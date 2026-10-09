# Diagnostic

Une section par test en échec : renseignez ses quatre champs.

## testAddingALineToAnUnknownOrderIsNotFound

**Symptôme** :

**Cause** :

**Règle du module en jeu** :

**Correctif** :

## testAddingALineToAPaidOrderIsAConflict

**Symptôme** :

**Cause** :

**Règle du module en jeu** :

**Correctif** :

## testAddingALineToMyOrder

**Symptôme** :

**Cause** :

**Règle du module en jeu** :

**Correctif** :

## testAddingALineWithAZeroQuantityIsUnprocessable

**Symptôme** :

**Cause** :

**Règle du module en jeu** :

**Correctif** :

## testListingKitchenTicketsReturnsMine

**Symptôme** : Lorsque Bob souhaite accéder à ses tickets, il ne voit pas les deux tickets qui ont été édités pour lui.

**Cause** : Fichier : KitchenTicketRepository.php / Ligne : il n'y a pas de tri sur le créateur du ticket.

**Règle du module en jeu** : Il est important de filtrer les requêtes SQL sur l'utilisateur lorsque l'on récupère des données personnelles.

**Correctif** : ->andWhere('k.createdBy = :user') + ->setParameter('user', $user)

## testListingKitchenTicketsWithoutTokenIsUnauthorized

**Symptôme** : Nous souhaitons pouvoir accéder aux tickets en étant forcément connecté.

**Cause** : Fichier : KitchenTicket.php / Ligne : dans l'opération de l'API ressource.

**Règle du module en jeu** : La sécurité intervient entre le provider et le processor et envoi les codes d'erreur.

**Correctif** : security: "is_granted('ROLE_USER')"

## testOpeningAnOrderIgnoresAnAbandonedOne

**Symptôme** :

**Cause** :

**Règle du module en jeu** :

**Correctif** :

## testPayingMyOrderMarksItPaid

**Symptôme** :

**Cause** :

**Règle du module en jeu** :

**Correctif** :

## testRefreshingTwiceWithTheSameTokenIsUnauthorized

**Symptôme** : Le jeton de rafraîchissement n'est utilisable qu'une unique fois alors que nous souhaitons pouvoir l'utiliser jusqu'à sa date d'expiration.

**Cause** : Fichier gesdinet_jwt_refresh_token.yaml / Ligne single_use: false

**Règle du module en jeu** : Le jeton doit avoir une durée limitée mais le jeton de raffraichissement doit être plus long pour éviter de devoir se connecter systématiquement.

**Correctif** : single_use: true

## testRemovingALineFromSomeoneElsesOrderIsForbidden

**Symptôme** : Lorsque Bob essaye d'ajouter une ligne au panier d'Alice, il doit obtenir une 403, cependant il reçoit une 204.

**Cause** : Fichier Order.php / Ligne security: "is_granted('ROLE_USER') or object.getCreatedBy() == user"

**Règle du module en jeu** : La règle de sécurité est mal configuré car il faut que les deux conditions soient respectées. La sécurité doit intervenir entre le provider et le processor et donc jeter les codes d'erreur.

**Correctif** : security: "is_granted('ROLE_USER') and object.getCreatedBy() == user"
