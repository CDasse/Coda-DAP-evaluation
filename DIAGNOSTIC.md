# Diagnostic

Une section par test en échec : renseignez ses quatre champs.

## testAddingALineToAnUnknownOrderIsNotFound

**Symptôme** : Lorsque l'on essaye d'ajouter une ligne à une commande qui n'existe pas, on doit obtenir une 404 NOT FOUND. Cependant, on reçoit ici une 500 : `Unable to call method "getCreatedBy" of non-object "object"`.

**Cause** : Fichier : `Order.php` / Ligne : dans la déclaration de l'opération du POST '/orders/{id}/lines'.

**Règle du module en jeu** : Lorsque l'on ajoute un élément en base de données, le provider se charge de récupérer la donnée et le processor de faire les modifications. Or ici, le provider n'est pas déclaré, la sécurité essaye donc de vérifier une égalité sur un object inexistant (object qui doit normalement être remonté par le provider).

**Correctif** : `provider: OrderProvider::class,` dans la déclaration de l'opération du POST '/orders/{id}/lines' sinon le fichier n'est pas appelé.

## testAddingALineToAPaidOrderIsAConflict

**Symptôme** : Lorsque l'on essaye d'ajouter une ligne à une commande déjà payé, on doit être rejeté par une 409 CONFLICT. Cependant, l'opération s'execute correctement et nous recevons une 201 CREATED.

**Cause** : Fichier `OrderAddLineProcessor.php` : Ligne : `$order = $this->orderService->findOneById($uriVariables['id'])`;

**Règle du module en jeu** : Nous devons effectuer les opérations de vérification (programmation défensive) le plus tôt possible, donc ici dans le processor. On doit donc vérifier que le status de la commande n'est pas 'paid'.

**Correctif** :
```
   if ($order->getStatus() === OrderStatus::Paid)
{
   throw new OrderAlreadyPaidException();
}
``````

## testAddingALineToMyOrder

**Symptôme** : Lorsque l'on ajoute une ligne à la commande, on s'attend à récupérer un total de 2450. Cependant, on obtient la valeur 2050.

**Cause** : Fichier : `OrderService` / Ligne : `subtotal: $dish->getPrice()` de la fonction `toLine()`.

**Règle du module en jeu** : Les sous-totaux étant modifiés régulièrement, ils n'ont pas leur place dans la base de donnée. Il faut donc les calculer dans les services. Or ici, la multiplication par la quantité a été oubliée.

**Correctif** : `subtotal: $dish->getPrice() * $line->getQuantity(),`

## testAddingALineWithAZeroQuantityIsUnprocessable

**Symptôme** : Lorsque l'on essaye d'ajouter une ligne à une commande avec une quantité de zéro, on doit obtenir une 422. Cependant,  on reçoit une 201 CREATED. 

**Cause** : Fichier : `OrderAddLineInput.php` / Ligne: `#[Assert\PositiveOrNull]`

**Règle du module en jeu** : Les DTO permettent de réaliser les validations de surface. Ici on vérifie que la valeur est positive ou null, on ne devrait vérifier que sa positivité.

**Correctif** : `#[Assert\Positive]`

## testListingKitchenTicketsReturnsMine

**Symptôme** : Lorsque Bob souhaite accéder à ses tickets, il ne voit pas les deux tickets qui ont été édités pour lui.

**Cause** : Fichier : `KitchenTicketRepository.php` / Ligne : il n'y a pas de tri sur le créateur du ticket.

**Règle du module en jeu** : Il est important de filtrer les requêtes SQL sur l'utilisateur lorsque l'on récupère des données personnelles.

**Correctif** : 
```
->andWhere('k.createdBy = :user')
->setParameter('user', $user)
```

## testListingKitchenTicketsWithoutTokenIsUnauthorized

**Symptôme** : Nous souhaitons pouvoir accéder aux tickets en étant forcément connecté. Cependant, dans cette configuration, tout le monde peut y accéder.

**Cause** : Fichier : `KitchenTicket.php` / Ligne : dans la déclaration de l'opération de l'API ressource.

**Règle du module en jeu** : La sécurité intervient entre le provider et le processor et envoi les codes d'erreur. Et cette dernière ce déclare dans la déclaration de l'opération au niveau de l'entité.

**Correctif** : `security: "is_granted('ROLE_USER')"`

## testOpeningAnOrderIgnoresAnAbandonedOne

**Symptôme** : Lorsque l'on abandonne une commande (soft deleted), cela ne devrait pas nous bloquer pour en ouvrir une autre. Or c'est le cas.

**Cause** : Fichier : `SoftDeleteFilter` / Ligne : `return sprintf('%s.deleted_by_id IS NULL', $targetTableAlias)`;

**Règle du module en jeu** : Afin que toutes les requêtes rédigées dans les répositories vérifient que les données récupérées ne sont pas soft deleted, on crée un filtre commun à tous.

**Correctif** : `return sprintf('%s.deleted_at IS NULL', $targetTableAlias);`

## testPayingMyOrderMarksItPaid

**Symptôme** : Lorsque l'on paye une commande, cette dernière ne semble pas enregistrée en base comme 'payée'.

**Cause** : Fichier : `KitchenTicketService.php` / Ligne : `$this->kitchenTicketRepository->flush();`

**Règle du module en jeu** : Lorsque les tickets sont émis, il faut les persister à Doctrine mais on doit attendre que les modifications soient adoptées sur la commande pour `flush`. Sinon les modifications ne seront jamais adoptées.

**Correctif** : `$this->orderRepository->flush();` dans la fonction `pay` du fichier `OrderService` et non dans le service du ticket.

## testRefreshingTwiceWithTheSameTokenIsUnauthorized

**Symptôme** : Le jeton de rafraîchissement n'est utilisable qu'une unique fois alors que nous souhaitons pouvoir l'utiliser jusqu'à sa date d'expiration.

**Cause** : Fichier : `config/packages/gesdinet_jwt_refresh_token.yaml` / Ligne : `single_use: false`

**Règle du module en jeu** : Le jeton doit avoir une durée limitée mais le jeton de raffraichissement doit être plus long pour éviter de devoir se connecter systématiquement.

**Correctif** : `single_use: true`

## testRemovingALineFromSomeoneElsesOrderIsForbidden

**Symptôme** : Lorsque Bob essaye d'ajouter une ligne au panier d'Alice, il doit obtenir une 403 FORBIDDEN. Cependant, il reçoit une 204.

**Cause** : Fichier : `Order.php` / Ligne : `security: "is_granted('ROLE_USER') or object.getCreatedBy() == user"`

**Règle du module en jeu** : La règle de sécurité est mal configuré car il faut que les deux conditions soient respectées. La sécurité doit intervenir entre le provider et le processor et donc jeter les codes d'erreur.

**Correctif** : `security: "is_granted('ROLE_USER') and object.getCreatedBy() == user"`
