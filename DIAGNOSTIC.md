# Diagnostic

Une section par test en échec : renseignez ses quatre champs.

## testAddingALineToAnUnknownOrderIsNotFound

**Symptôme** :

**Cause** :

**Règle du module en jeu** :

**Correctif** :

## testAddingALineToAPaidOrderIsAConflict

**Symptôme** : Lorsque l'on essaye d'ajouter une ligne à une commande déjà payé, on doit être rejeté par une 409, or l'opération passe en 201

**Cause** : Fichier OrderAddLineProcessor.php : Ligne : $order = $this->orderService->findOneById($uriVariables['id']);

**Règle du module en jeu** : Nous devons effectuer les opérations de vérification (programmation défensive) le plus tôt possible.

**Correctif** :
```if ($order->getStatus() === OrderStatus::Paid)
{
throw new OrderAlreadyPaidException();
}
``````

## testAddingALineToMyOrder

**Symptôme** :

**Cause** :

**Règle du module en jeu** :

**Correctif** :

## testAddingALineWithAZeroQuantityIsUnprocessable

**Symptôme** : Lorsque l'on essaye d'ajouter une ligne à une commande avec une quantoté de 0, on doit obtenir une 422 or on reçoit une 201. 

**Cause** : Fichier : OrderAddLineInput.php / Ligne: #[Assert\PositiveOrNull]

**Règle du module en jeu** : Les DTO permettent de réaliser les validations de surface.

**Correctif** : #[Assert\Positive]

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

**Symptôme** : Lorsque l'on abandonne une commande (soft deleted), cela ne devrait pas nous bloquer pour en ouvrir une autre. Or c'est le cas.

**Cause** : Fichier : SoftDeleteFilter / Ligne : return sprintf('%s.deleted_by_id IS NULL', $targetTableAlias);

**Règle du module en jeu** : Afin que toutes les requêtes rédigées dans les répository vérifie que les données récupérées ne sont pas soft delted, on crée un filtre commun à tous.

**Correctif** : return sprintf('%s.deleted_at IS NULL', $targetTableAlias);

## testPayingMyOrderMarksItPaid

**Symptôme** : Lorsque l'on paye une commande, cette dernière ne semble pas enregistrée en base comme payée.

**Cause** : Fichier : KitchenTicketService.php / Ligne : $this->kitchenTicketRepository->flush();

**Règle du module en jeu** : Lorsque les tickets sont émis, ils fuat les persistes mais on doit attendre que les modifications soient adoptées sur la commande pour flush. Sinon les modifications ne sont jamais adoptées.

**Correctif** : `$this->orderRepository->flush();` dnas la fonction `pay` du fichier `OrderService`.

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
