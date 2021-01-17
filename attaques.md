### "Clickjacking"

Cette attaque est dangereuse car elle est facilement detectée par des outils de scanning utilisables par des attaquants sans grande connaissance, dans notre cas elle a été immédiatement detectée par _"Owasp ZAP"_ lors de notre analyse.

##### Applications contre notre site

Cette attaque permet à un site malveillant d'afficher un fragment de notre messagerie et de confondre cette dernière une autre action. La victime pense effectuer une action sur le site malveillant mais une action se produit en réalité sur notre messagerie. Plus d'informations disponibles [à cette adresse](https://portswigger.net/web-security/clickjacking).

Cette attaque diffère des attaques CSRF car elle demande une action utilisateur, pas uniquement de forger une requête qui sera executée par ce dernier.

##### Vérification de l'attaque

Afin de vérifier que cette attaque était possible, nous avons mis en ligne une page très simple qui avait pour but d'inclure notre messagerie dans une balise `<iframe>` depuis un autre dommage.

![](img/clickjackingproof.png)

Nous pouvons ici constater que notre messagerie a été incluse, les attaques de ce type sont donc possibles.

##### Cause de l'attaque

L'en-tête `X-Frame-Options` n'est pas inclus dans les réponses du serveur web, conduisant à cette vulnérabilité. De plus, l'en-tête `Content-Security-Policy` qui est généralement utilisé de paire avec `X-Frame-Options` et permet de mitiger les attaques de ce type ainsi que les attaques XSS est également absent.

##### Mitigation de l'attaque

Les en-têtes correspondantes doivent être ajoutées, on utilisera `X-Frame-Options: deny` ainsi que `Content-Security-Policy: frame-ancestors 'self';` afin de s'assurer que cela ne soit plus une menace.

Dans notre cas, nous avons ajouté un hook python qui va ajouter des en-têtes à toutes les réponses.

```python
@APP.after_request
def add_headers(response):
    response.headers["X-Frame-Options"] = "DENY"
    response.headers["Content-Security-Policy"] = "frame-ancestors 'self';"
    return response
```

##### Vérification

Après cette modification, on peut réessayer l'inclusion.

![](img/clickjackingfixed.png)

Nous pouvons constater que l'inclusion de notre application n'est plus possible depuis un autre domaine.

Dans le cas où nous souhaitions rendre cela possible depuis un site partenaire, nous pouvons utiliser `X-Frame-Options: allow-from https://normal-website.com/`.



### "CSRF" attacks

Tout comme l'attaque précédente, les attaques CSRF représentent un risque notamment par la facilité de détecter ces dernières (absence de jeton mitigateur dans les formulaires). Dans notre cas, l'utilisation encore une fois de _"Owasp ZAP"_ nous a signalé la chose rapidement.

##### Applications contre notre site

Des données sont transmises en `POST` dans plusieurs cas sur l'application, on peut détailler la liste des _"routes"_ concernées

* `/compose` - un attaquant peut usurper l'identité d'un utilisateur pour envoyer un message
* `/user/username` - un attaquant peut changer le mot de passe d'un utilisateur, le désactiver, ou le rendre administrateur. Il faut cependant que la victime possède les privilèges administrateurs pour accéder à cet URL.
* `/userAdd` - un attaquant peut créer un compte sur la messagerie si la victime possède les droits administrateurs.
* `/changePassword` - un attaquant peut changer le mot de passe de l'utilisateur
* `/login` - un attaquant peut .... se connecter légalement au site web s'il connaît les identifiants d'un utilisateur. Oui bon, intérêt limité par ici.
* `/signup` - un attaquant peut ... créer un compte comme depuis le formulaire d'inscription classique. Ici aussi l'intérêt semble limité.

Il semble donc que les modifications qu'un attaquant peut forcer un utilisateur à faire via CRSF sont graves, en particulier il peut se promovoir administrateur si la victime qui clique son lien est administrateur (ce qui lui donne plein pouvoirs) ou peut se faire passer pour un utilisateur s'il envoie des messages en son nom, ce qui peut être très grave.

##### Vérification de l'attaque

Afin de vérifier l'attaque, nous allons forger un URL permettant à un attaquant de forcer la victime à s'envoyer un message.

Le compte de notre victime est le compte administrateur par défault _admin_ et le compte utilisé par l'attaquant pour recevoir le message est `sicriss`. On imagine dans notre scénario d'attaque que la victime est un professeur de STI et que l'attaquant va s'envoyer un message qui lui indique que son cours est validé (un peu facile me direz-vous).

On créé une page HTML très basique avec un envoi de requête forgé

```html
<html>
  <body>
    <form action="http://localhost:5000/compose" method="POST">
      <input type="hidden" name="recipient" value="Sicriss" />
      <input type="hidden" name="title" value="Annonce" />
      <input type="hidden" name="body" value="Vous+avez+réussi+le+cours+de+STI+!" />
    </form>
    <script>
      document.forms[0].submit();
    </script>
  </body>
</html> 
```

En vérifiant la chose depuis un accès sur le fichier HTML, nous sommes redirigés sur la page `/login`.

![](img/loginpage.png)

##### Pourquoi l'attaque n'a pas été possible de cette manière ? 

L'attaque a été rendue impossible à cause des options spécifiées sur le cookie de session `SameSite : strict`.

La requête a été forgée correctement mais le cookie de session n'a pas été inclu dans cette dernière en raison de cette policy, ce qui a fait échouer notre tentative de CSRF.

Il serait possible à un attaquant de parvenir à lancer l'attaque s'il était sur le même domaine, cependant cela nous semble être très spécifique (si l'attaquant a déjà le contrôle sur le domaine, pas besoin de passer par une telle attaque).

Pour l'exercice nous avons procédé au test depuis un URL du même domaine (`localhost:81` vers `locahost:5000`) et obtenu cette fois-ci un résultat valide (voir ci-dessous).

![](img/csrfworking.png)

##### Mitigation de l'attaque

L'attaque a été ici bloquée par l'attribut `SameSite` sur le cookie de session. 

```python
res.set_cookie(
	key='auth',
    value=jwt_encode({'session': session_id, 'exp': expiry}),
    expires=expiry,
    samesite='Strict',)
```

Une autre méthode (plus classique) consiste à utiliser des tokens CSRF dans les requêtes concernées par la vulnérabilité ([détails](https://portswigger.net/web-security/csrf)) mais il semblerait que dans notre cas ce ne soit pas nécessaire, toutes les URLs concernées étant protegées par la session utilisateur.



