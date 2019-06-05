---
title: React Side Effects
theme: solarized
revealOptions:
    transition: 'slide'
---

### React
## Side Effects

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

## Les side effect en react

---

```js
// in native: side effect
const view = () => {
  document.body.innerHTML = `
    <div class="container">
      <p>Hello world!</p>
    </div>
  `
}
```

```jsx
// in React: no side effect
const View = () => {
  return (
    <div className="container">
      <p>Hello world!</p>
    </div>
  )
}
```

React rend les intéractions avec le DOM **pures** et **déclaratives**

Note:
- Le principe de react est justement de ne pas avoir à faire de side effects pour modifier le DOM.
    On ne mute plus directement les DOM nodes, mais à la place on donne une configuration qui
    représente ce à quoi le DOM doit resembler en fonction de ses props.

---

<small>Oui mais...</small>

Le web ce n'est pas seulement intéragir avec le DOM

---

quelques side effects, du dévelopement frontend


<small class="fragment">Les network requests (HTTP ou WebSocket la plupart du temps)</small>

<small class="fragment">L'`url` et l'`History` pour le routing</small>

<small class="fragment">Le `localStorage` ou les `cookies` pour la persistence</small>

<small class="fragment">Les `event listener` globaux (scroll, drag & drop, etc)</small>

<small class="fragment">les autres `threads` (service workers / web workers)</small>

<small class="fragment">la `console` (seulement pour le débugging a priori)</small>

Note: 
Cependant, lorsque l'on fait du web, on gère d'autres types de side effects que ceux du DOM.
    - Des network requests (HTTP ou WebSocket la pluspart du temps)
    - le localStorage ou les cookies pour la persistence
    - L'url et l'History pour le routing
    - les autres threads (service workers / web workers)
    - parfois on a besoin d'ajouter des event listener à la main sur la window (scroll, drag & drop, etc)
    - la console (seulement pour le débugging a priori)

---

pour tout ça, on utilise **useEffect**

---

```jsx
const App = () => {
  useEffect(() => {
    // effects
  })
}
```

<small class="fragment">On place le code relatif aux effects dans un **callback**</small>
<small class="fragment">Ce callback sera runné de manière **asynchrone**, une fois que react à updaté le DOM.</small>


---

```js
const App = () => {
  useEffect(() => {
    // i can use 'user' and 'isAdmin'
  }, [user, isAdmin])
}
```

<small class="fragment">Les dépendances sont données en **deuxième paramètre**</small>
<small class="fragment">À chaque fois que l'une de ses dépendances va **changer**, le callback sera **re-exécuté**</small>
<small class="fragment">Le tableau de dépendances est **optionelle**</small>
<small class="fragment">si il n'est pas fourni, le callback sera exécuté après chaque render</small>


Note: 
### dependances explicites
  - useEffect est une API très intéressante car elle permet de penser ses side effects comme une **conséquence** d'un changement de data. Les dépendances sont explicite, et le code est runné à chaque
    fois qu'une de ses dépendance change.

---

```js
const App = () => {
  useEffect(() => {
    window.addEventListener('scroll', handler)
    return () => window.removeEventListener('scroll', handler)
  }, [handler])
}
```

<small class="fragment">À l'intérieur du useEffect, je peux retourner une **fonction de cleanup**</small>
<small class="fragment">Elle sera exécutée si notre composant est **retiré du DOM**</small>
<small class="fragment">ou si les dépendances ne sont **plus à jour**</small>

Note:
### cleanup callback
  - Pour certains effets, on a besoin de pouvoir les annuler lors que leur dépendances ont changé,
    comme le event listeners par exemple. Pour ça on peut retourner un callback de cleanup dans notre
    fonction d'effet.

---

<iframe src="https://codesandbox.io/embed/wonderful-hypatia-f3g60?fontsize=14&view=editor" title="exercice use effect" allow="geolocation; microphone; camera; midi; vr; accelerometer; gyroscope; payment; ambient-light-sensor; encrypted-media" style="width:100%; height:500px; border:0; border-radius: 4px; overflow:hidden;" sandbox="allow-modals allow-forms allow-popups allow-scripts allow-same-origin"></iframe>

---

### Créer un custom hook


---

Il est possible de **composer** plusieurs hooks ensemble<br> 
pour créer un **nouveau** hook.


---

<iframe src="https://codesandbox.io/embed/exercice-use-effect-wz4pz?fontsize=14&view=editor" title="exercice custom hook" allow="geolocation; microphone; camera; midi; vr; accelerometer; gyroscope; payment; ambient-light-sensor; encrypted-media" style="width:100%; height:500px; border:0; border-radius: 4px; overflow:hidden;" sandbox="allow-modals allow-forms allow-popups allow-scripts allow-same-origin"></iframe>

---

Les custom hooks permettent d'**encapsuler** la complexité de notre code.

---

<small class="">En résumé</small>

<small class="fragment">**useEffect** est manière de rendre **déclaratif** du code effectful et impératif.</small>

<small class="fragment">**Déclaratif** car le code est dépendant de la data.</small>

Note:
en résumé, useEffect est manière d'abstraire du code impératif pour le rendre **déclaratif**, c'est à dire dépendant de la data.

---

### La gestion des erreurs

---

Les side effects sont par nature **imprevisibles**.

<small class="fragment">une request HTTP **peut échouer**</small>
<small class="fragment">un acces au cookie **peut échouer** (environement sandboxé, iframes...)</small>
<small class="fragment">une manipulation de DOM **peut échouer** (plusieurs modifications incompatibles)</small>

Note:
Les side effects sont par nature imprevisibles.
  - une request HTTP peut échouer
  - un acces au cookie peut échouer (dans un environnement sandboxé comme une iframe)
  - une manipulation de dom peut échouer (en cas de modification incompatible par une autre partie de notre codebase ou par l'utilisateur lui même, ses extensions etc)

---

Il **faut** donc considérer les cas d'erreur dans le code.

---

#### Les promises

---
```js
const p = new Promise((resolve, reject) => {
  // make some effect
  // if it succeed
  resolve(someData)
  // if it fails
  reject(someError)
})
```
<small class="fragment">**2 branches** : une pour le succès et une pour l'erreur</small>

Note:
Pour ça, en javascript, on a quelque chose de pratique: **les promises**.

- permet de gérer le cas d'erreur

---


```js
fetchUser() // returns a promise
  .then(user => fetchFriends(user)) // chain a second promise
  .then(friends => {}) // when all the side effect are done, run some code
```

<small class="fragment">on peut *séquencer* plusieurs effets</small>

Note:
- permet de composer et de sequencer plusieurs side effects différents

---


<small>
Donc, pourquoi ne pas créer un *hook* pour utiliser des promises dans nos components ?
</small>


---

<iframe src="https://codesandbox.io/embed/cocky-breeze-0dxfk?fontsize=14&view=editor" title="exercice use promise" allow="geolocation; microphone; camera; midi; vr; accelerometer; gyroscope; payment; ambient-light-sensor; encrypted-media" style="width:100%; height:500px; border:0; border-radius: 4px; overflow:hidden;" sandbox="allow-modals allow-forms allow-popups allow-scripts allow-same-origin"></iframe>

Note:

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

  return [type, data, error]
}
```


---

### Side effect et state reducer pattern

---

On a un problème. 

<small class="fragment">puisque notre reducer est une fonction **pure**, elle ne doit pas avoir de side effect.</small>

Note:
pottentiel gif: problème

---

Son type est trop restrictif

`(state, action) => state`

---

Mais pourquoi pas le changer ?

`(state, action) => [state, callback]`

---

<iframe src="https://codesandbox.io/embed/interesting-yonath-bfou0?fontsize=14&view=editor" title="exercice use effect reducer " allow="geolocation; microphone; camera; midi; vr; accelerometer; gyroscope; payment; ambient-light-sensor; encrypted-media" style="width:100%; height:500px; border:0; border-radius: 4px; overflow:hidden;" sandbox="allow-modals allow-forms allow-popups allow-scripts allow-same-origin"></iframe>

---

C'est l'approche de plusieurs langages fonctionelles comme **Elm** ou **Reason**

Note:
deux approches :
1. Changer la signature de notre réducer:

au lieu de `(state, action) => state`
on passe à `(state, action) => [state, effectCallback]`

^ l'approche de ELM et de Reason. ça pose quand même un problème: quand est ce que l'on run les side effects ?
Quand le side effect est synchrone, on peut avoir envie de l'avoir executé avant de re-render.

// CODE Sandbox ?

---

Deuxième solution : 

Utiliser des **middlewares**

---

C'est l'approche de **Redux**

La plus courante en javascript

---

<iframe src="https://codesandbox.io/embed/quirky-bird-ux76p?fontsize=14&view=editor" title="exercice : effect middleware" allow="geolocation; microphone; camera; midi; vr; accelerometer; gyroscope; payment; ambient-light-sensor; encrypted-media" style="width:100%; height:500px; border:0; border-radius: 4px; overflow:hidden;" sandbox="allow-modals allow-forms allow-popups allow-scripts allow-same-origin"></iframe>

Note:

Commencer par expliquer ce qu'est un middleware
- faire un logger middleware
- simple thunk middleware
- effect middleware avec promises

Ou exécuter des side effect dans le contexte d'un state reducer ?



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

---

Quel sont les bénéfices de cette approche ? 


<small class="fragment">Nos effets deviennent **inspectables** car ils sont représentés par des actions</small>
<small class="fragment">On peut implémenter des **optimistic updates** automatiques</small>

Note:
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

---

### Merci

de votre attention


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

  .reveal {
    font-size: 35px;
  }

  .reveal small {
    font-size: .7em;
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
