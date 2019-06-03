---
title: React Side Effects
theme: solarized
revealOptions:
    transition: 'slide'
---

## React
### Side Effects

<p style="font-size:.7em">Slides: https://gvergnaud.github.io/react-side-effects</p>


---

## 👋

**Gabriel Vergnaud**

Héticien de la P2017

<div class="flex">
  <span>Frontend engineer&nbsp;</span><img src="resources/logo-sketchfab.png" class="simple-image" style="width:200px" />
</div>

<div class="flex">
[@gvergnaud](https://github.com/gvergnaud)<span>&nbsp;on&nbsp;</span><img
  src="resources/logo-github.png"
  class="simple-image"
  style="width:50px;" />
</div>

<div class="flex">
[@GabrielVergnaud](https://twitter.com/GabrielVergnaud)<span>&nbsp;on&nbsp;</span><img
  src="resources/logo-twitter.png"
  class="simple-image"
  style="width:50px;" />
</div>

Note:
- Qui suis je ?
  - gabriel vergnaud
  - Heticien P2017
  - developer à Sketchfab.com (On recrute!)
  - gvergnaud on github
  - GabrielVergnaud on twitter

---
## 🗺
#### I. qu'est ce que les side effects
#### II. les effects en React
#### III. gérer les erreurs
#### IV. les side effects et le state reducer pattern

---

# I
### les side effects ?
<small>mais qu'est-ce donc</small>

---

Le code qui intéragit avec le **monde extérieur**

---

<small>Mais extérieur à quoi ?</small>
<br/>
<p class="fragment">Au **scope** de la fonction</p>

---

```js
let imInTheUpperScope = true

function sideEffectFunction(params) {
  // tout ce qui est défini ici
  // fait partie du scope de la function `sideEffectFunction`

  const imPartOfTheFunctionScope = true

  // si je modifie quelque chose
  // du scope parent, alors je fais un side effect:

  imInTheUpperScope = false // /!\ Side effect!

  // Modifier les paramètres est également un side effect
  params.hello = '👋'
}

// l'ordre des définitions n'a pas d'importance
let imAlsoInTheUpperScope = true
```

`{}`

<small>Le scope de la function est determiné par ce qui est défini à l'intérieur de ses brackets</small>


---

<small>Quelques **exemples** de **side effects** :</small>

<small class="fragment">**Modifier des variables** du scope extérieur</small>
<br>
<small class="fragment">Les requètes **HTTP**</small>
<br>
<small class="fragment">L'accès à une **base de donnée**</small>
<br>
<small class="fragment">Cookies, **localStorage**</small>
<br>
<small class="fragment">**Web APIs** (DOM, WebAudio, WebGL...)</small>
<br>
<small class="fragment">**File system**</small>



Note:
## - Qu'est ce que les side effects ?
  - Tout ce qui touche au monde extérieur
    - HTTP
    - accès à une base de donnée
    - Cookies, localStorage
    - web APIs
    - File System


---

fonctions **pures** et **impures**

<small class="fragment">Une fonction pure ne produit **pas de side effect**.</small>
<br>
<small class="fragment">Une fonction pure retourne le **même resultat** si on lui donne les **mêmes arguments**.</small>
<br>
<small class="fragment">Une fonction pure est une fonction au sens **mathématique** du terme.</small>

Note:

  - Dans le monde de la programmation fonctionelle on appelle ça du code Impure
    - Une fonction pure n'a pas de side effect et donne le même résultat si on lui donne le arguments.
    - on dit qu'elle est *référentiellement transparente* (referencial transparency).

---


Slide meme math pas cool

---



# Les side effects


## - Les side effect en réact

  - Le principe de react est justement de ne pas avoir à faire de side effects pour modifier le DOM.
    On ne mute plus directement les DOM nodes, mais à la place on donne une configuration qui
    représente ce à quoi le DOM doit resembler en fonction de ses props.

  - React rend les intéractions avec le DOM **pures** et **déclaratives**.

  Cependant, lorsque l'on fait du web, on gère d'autres types de side effects que ceux du DOM.
    - Des network requests (HTTP ou WebSocket la pluspart du temps)
    - le localStorage ou les cookies pour la persistence
    - L'url et l'History pour le routing
    - les autres threads (service workers / web workers)
    - parfois on a besoin d'ajouter des event listener à la main sur la window (scroll, drag & drop, etc)
    - la console (seulement pour le débugging a priori)


  Pour ça, un hook que vous connaissez: **useEffect**

### runner du code impure
  - useEffect permet de runner du code impure lié à son component.
```js
useEffect(() => {
  // ce code sera runné de manière asynchrone, une fois que react à updater le DOM.
})
```

### dependances explicites
  - useEffect est une API très intéressante car elle permet de penser ses side effects comme une **conséquence** d'un changement de data. Les dépendances sont explicite, et le code est runné à chaque
    fois qu'une de ses dépendance change.

```js
useEffect(() => {
  // here I'll be using my dependencies to execute some effectful code.
}, [my, dependencies])
```

### cleanup callback
  - Pour certains effets, on a besoin de pouvoir les annuler lors que leur dépendances ont changé,
    comme le event listeners par exemple. Pour ça on peut retourner un callback de cleanup dans notre
    fonction d'effet.

```js
useEffect(() => {
  el.addEventListener(eventName, handler)
  return () => el.removeEventListener(eventName, handler)
}, [el, eventName, handler])
```

### custom hooks
Notre code effectful est souvent le code le plus complexe que l'on a à gérer dans notre codebase.
Les hooks de react sont très pratiques car ils nous permettent d'**abstraire** cette complexité dans des hooks
customs.

```js
const useMouse = () => {
  const [mouse, setMouse] = useState([0, 0])

  useEffect(() => {
    const handler = (e) => setMouse([e.clientX, e.clientY])
    window.addEventListener('mousemove', handler)
    return () => window.removeEventListener('mousemove', handler)
  }, [])

  return mouse
}

function MyComponent () {
  const [x, y] = useMouse() // le side effect est abstrait (contenu, encapsulé dans a fonction)
  return <div style={{ transform: `translate(${x}px, ${y}px)` }} />
}
```
en résumé, useEffect est manière d'abstraire du code impératif pour le rendre **déclaratif**, c'est à dire dépendant de la data.


## La gestion des erreurs
Les side effects sont par nature imprevisibles.
  - une request HTTP peut échouer
  - un acces au cookie peut échouer (dans un environnement sandboxé comme une iframe)
  - une manipulation de dom peut échouer (en cas de modification incompatible par une autre partie de notre codebase ou par l'utilisateur lui même, ses extensions etc)

Il faut donc considérer les cas d'erreur dans le code.


Pour ça, en javascript, on a quelque chose de pratique: **les promises**.

- permet de gérer le cas d'erreur
```js
const p = new Promise((resolve, reject) => {
  // make some effect
  // if it succeed
  resolve(someData)
  // if it fails
  reject(someError)
})
```

- permet de composer et de sequencer plusieurs side effects différents
```js
fetchUser() // returns a promise
  .then(user => fetchFriends(user)) // chain a second promise
  .then(friends => {}) // when all the side effect are done, run some code
```

Donc, pourquoi ne pas créer un *hook* pour utiliser des promises dans nos components ?


TUTO construire un promise based useSideEffect hook


```js
const usePromise = (getPromise, deps) => {
  const [type, setType] = useState('pending')
  const [data, setData] = useState(null)
  const [error, setError] = useState(null)
  useEffect(() => {
    let isCancelled = false
    setType('pending')
    setData(null)
    setError(null)

    getPromise()
      .then(data => {
        if (isCancelled) return
        setData(data)
        setType('resolved')
      })
      .catch(err => {
        if (isCancelled) return
        setError(err)
        setType('rejected')
      })

    return () => {
      isCancelled = true
    }
  }, deps)
}
```

## Side effect et state reducer pattern

Ou exécuter des side effect dans le contexte d'un state reducer ?

deux approches :
1. Changer la signature de notre réducer:

au lieu de `(state, action) => state`
on passe à `(state, action) => [state, effectCallback]`

^ l'approche de ELM et de Reason. ça pose quand même un problème: quand est ce que l'on run les side effects ?
Quand le side effect est synchrone, on peut avoir envie de l'avoir executé avant de re-render.

// CODE Sandbox ?

2. utiliser un middleware

le middleware prend une fonction qui retourne une promesse et la transform en actions:

```js

const someEffectAction = {
  type: 'SOME_EFFECT_ACTION',
  async effect() {
    // do stuff
  }
}

dispatch(someEffectAction)
```

Le middleware intercepte l'action et va remplacer la function `effect` par de la data:

```js
{
  type: 'SOME_EFFECT_ACTION',
  effect: {
    type: 'pending',
    error: null,
    data: null
  }
}
```
puis

```js
{
  type: 'SOME_EFFECT_ACTION',
  effect: {
    type: 'resolved',
    error: null,
    data: {...} // data returned by the promise
  }
}
```

ou

```js
{
  type: 'SOME_EFFECT_ACTION',
  effect: {
    type: 'rejected',
    error: Error{...},  // error thrown by the promise
    data: null
  }
}
```

C'est l'approche prise par plusieurs projets open sources, dont Hyper, le terminal de zeit.co

TUTO construire ce middleware

### Quels sont les bénéfices ?

#### optimistic updates for free

Puisque l'on peut clairement identifier les actions qui représentent des effets,
on peut créer une logique d'error handling commune.

L'optimistic update est le fait de faire *comme si* l'effect avait déjà eu lieu avant que la requete ai fini
pour donner l'impression à l'utilisateur que notre application est super rapide.

cependant, si l'action fail, il faut **rollback** les changements sur le state pour bien indiquer qu'il y a eu une erreur!

Puisque l'on sait différentier une action 'pending' d'une action 'rejected', on peut automatiser ce processus pour toutes nos actions.
il suffit de :
- stocker le diff du state au pending
- en cas d'erreur, réappliquer le state précédent
- en cas de succès, on peut enlever le diff que l'on avait stocké.

## Tester les side effects
Les side effects sont la partie compliqué à tester. Comment simplifier le testing de notre app ?

les mocks & dependency injection

On peut encore améliorer notre middleware:

Au lieux d'importer la logique de nos side effect directement dans notre code, on peut se créer
des **services** qui vont se charcher de faire les side effect pour nous, et les injecter via notre middleware.

Comme ça, si on est dans un contexte de testes automatisé, dans lequel on a pas envie de vraiment faire des requests,
on peut injecter un mock de notre service.

```js
// inside our code

const apiService = {
  getUser(id) {
    return fetch(`/users/${id}`).then(res => res.json())
  }
}

effectMiddleware(apiService)

// Inside an automated test
const apiMock = {
  getUser(id) {
    return new Promise(resolve => {
      resolve({ name: 'Gabriel', id })
    })
  }
}

effectMiddleware(apiMock)

```



### Merci

de votre attention


---

### Ressources

---

**Gabriel Vergnaud**

- [Github](https://github.com/gvergnaud)
- [Twitter](https://twitter.com/GabrielVergnaud)
- [LinkedIn](https://www.linkedin.com/in/gabriel-vergnaud-09446199)

Note:
<!--
good gifs
shipit: https://giphy.com/gifs/fail-code-boat-143vPc6b08locw
keyboard:
 - https://giphy.com/gifs/vodafone-de-done-keyboard-l378gxtNQaXcsqY7K
 - https://giphy.com/gifs/working-typing-cartoon-11BbGyhVmk4iLS
 - https://giphy.com/gifs/tmnt-teenage-mutant-ninja-turtles-cFdHXXm5GhJsc
 - https://media.giphy.com/media/3o7aCTfyhYawdOXcFW/giphy.gif
c pas sorcier
- https://gfycat.com/fr/grandwearyflea
- https://giphy.com/gifs/cannabis-jamy-gourmaud-I9LxqeAcsBRNS
- https://www.google.com/imgres?imgurl=https%3A%2F%2Fresize-europe1.lanmedia.fr%2Fr%2F622%2C311%2Cforcex%2Ccenter-middle%2Fimg%2Fvar%2Feurope1%2Fstorage%2Fimages%2Feurope1%2Fmedias-tele%2Ffred-et-jamy-devoilent-le-nouveau-cest-pas-sorcier-1353712%2F21066760-1-fre-FR%2FFred-et-Jamy-devoilent-le-nouveau-C-est-pas-sorcier.jpg&imgrefurl=https%3A%2F%2Fwww.europe1.fr%2Fmedias-tele%2Ffred-et-jamy-devoilent-le-nouveau-cest-pas-sorcier-1353712&docid=XLVABVUvKIBgHM&tbnid=F-V_Lzs40s6APM%3A&vet=10ahUKEwjD4OWLl8biAhVC8uAKHcBuA0kQMwhRKAEwAQ..i&w=622&h=311&safe=off&bih=766&biw=1440&q=image%20jamie%20fred%20et%20jamy&ved=0ahUKEwjD4OWLl8biAhVC8uAKHcBuA0kQMwhRKAEwAQ&iact=mrc&uact=8 -->

<style>
  .flex {
    display: flex;
    align-items: center;
    justify-content: center;
  }
  .flex > *:not(:first-child) {
    margin-left: 10px
  }

  img.simple-image.simple-image {
    border:none;
    box-shadow:none;
    background: none;
  }

  .white.white.white {
    color: white;
    text-shadow: 0 2px 4px rgba(0,0,0, .5);
  }

  .lower.lower.lower {
    text-transform: none;
  }

  .reveal pre {
    border-radius: 5px;
    box-shadow: 0px 8px 25px rgba(0,0,0,.25);
  }

  .reveal section img {
    border: none;
      box-shadow: 0 5px 15px rgba(0, 0, 0, 0.15)
  }

  .reveal pre code {
    padding: 30px;
    border-radius: 5px;
    font-weight: normal;
  }

  .reveal code {
    font-weight: bold;
  }
</style>
