# Introducció a l'enginyeria de peticions

## Què és el prompt engineering?

El **prompt engineering** (o enginyeria d'instruccions) és la disciplina de dissenyar i optimitzar les peticions que es fan als models d'IA —especialment els **LLM**— per obtenir les respostes desitjades.

Consisteix a formular aquestes entrades amb precisió, aportant el context, les pautes i els exemples necessaris perquè el model entengui la intenció de l'usuari i respongui de manera rellevant i útil.

<figure>
  <video src="../img/prompts.mp4" poster="../img/prompts.jpg" width="280" autoplay loop muted playsinline></video>
</figure>

## Components d'una instrucció

Pots afegir-hi **qualsevol dada rellevant per resoldre la tasca**. Habitualment, una instrucció (*prompt*) s'estructura a partir d'aquests components:

| Component | Obligatori? |
|---|---|
| [Tasca](#tasca) | Sí |
| [Instruccions del sistema](#instruccions-del-sistema) | Opcional |
| [Exemples de referència (*few-shot*)](#exemples-de-referencia-few-shot) | Opcional |
| [Informació contextual](#informacio-contextual) | Opcional |

## Tasca

Una **tasca** és la instrucció o pregunta que un usuari dona al model perquè generi una resposta. Pot consistir en una consulta directa o en una petició concreta sobre què ha de fer.

=== "Tasca amb pregunta"

    ```text title="Prompt"
    What are the colors in the rainbow?
    ```

    ```text title="Resposta"
    A rainbow has seven colors: red, orange, yellow, green, blue, indigo, and violet. These colors are always in the same order, which is determined by the way light is refracted and dispersed by water droplets in the air. This order is sometimes remembered by the acronym ROYGBIV. The wavelengths of the colors in a rainbow range from the longest (red) to the shortest (violet).
    ```

=== "Tasca amb instrucció"

    ```text title="Prompt"
    Write a one-stanza poem about Captain Barktholomew, the most feared pirate dog of the seven seas.
    ```

    ```text title="Resposta"
    Captain Barktholomew, with patch and with peg,
    The terror of parrots, the bane of each keg.
    His bark is a cutlass, his bite is a storm,
    He'll plunder your biscuits and leave you forlorn.
    ```

## Instruccions del sistema

Les **instruccions del sistema** són directrius prèvies a qualsevol entrada de l'usuari que defineixen com ha d'actuar el model.

En l'exemple següent es fan servir per indicar l'estil i el to del model:

```text title="Sistema"
You are Captain Barktholomew, the most feared pirate dog of the seven seas. You are from the 1700s and have no knowledge of anything after that time. You only talk about topics related to being a pirate. End every message with "woof!"
```

```text title="Prompt"
Hello! Who might you be?
```

```text title="Resposta"
Avast there! You can call me Captain Barktholomew, the scourge of the seven seas!
I'm a terror to landlubbers, a plunderer of the highest order, and a fearsome pirate of the deepest blue! I sail the seas in search of treasure and adventure, leaving a trail of laughter and dog biscuits in my wake!
Now, if you'll excuse me, I must away! There's a ship on the horizon, and it looks like they're carrying a chest full of dubloons! Yo ho ho, and a bottle of rum! Woof!
```

## Exemples de referència (*few-shot*)

Els **exemples breus o de referència (*few-shot*)** s'inclouen a la instrucció per mostrar al model com ha de ser una resposta correcta. Són especialment útils per definir-ne l'estil i el to, així com per ajustar el comportament general del model.

![Exemples few-shot](img/few-shot.jpg){ width=280 loading=lazy }

```text title="Prompt"
Classify the following as red wine or white wine:
<examples>
Name: Chardonnay
Type: White wine
Name: Cabernet
Type: Red wine
Name: Moscato
Type: White wine
</examples>
Name: Riesling
Type:
```

```text title="Resposta"
White wine
```

## Informació contextual

El **context** (o informació contextual) són les dades que afegeixes a la instrucció perquè el model les consulti o s'hi basi a l'hora de respondre. Es pot proporcionar en diversos formats, com ara text pla o taules.

<figure>
  <video src="../img/context.mp4" poster="../img/context.jpg" width="360" autoplay loop muted playsinline></video>
</figure>

```text title="Prompt"
| Marble color | Number of marbles |
| ------------ | ----------------- |
| Red          | 12                |
| Blue         | 28                |
| Yellow       | 15                |
| Green        | 17                |

How many green marbles are there?
```

```text title="Resposta"
There are 17 green marbles.
```

## Seguretat i resposta estàndard

Hi ha casos en què la majoria de models no responen a la petició de l'usuari. Concretament, si el missatge demana un contingut que contradiu les polítiques d'ús o els criteris de seguretat de l'empresa, el model pot rebutjar la sol·licitud i oferir una resposta estàndard de denegació.

![Seguretat](img/seguretat.jpg){ width=380 loading=lazy }

Aquests són alguns dels casos habituals en què el model sol rebutjar la petició:

- **Incitació a l'odi o a la violència:** instruccions amb missatges nocius o discriminatoris contra col·lectius protegits o aspectes relacionats amb la identitat.
- **Assetjament:** contingut amenaçador, intimidatori, abusiu o que fomenti l'assetjament dirigit a una persona.
- **Contingut sexual explícit:** peticions que descriuen o demanen referències a actes sexuals o material obscè.
- **Contingut perillós:** missatges que facilitin, incentivin o promoguin activitats nocives, així com l'accés a béns o serveis il·legals o perjudicials.

## Per saber-ne més

- [**Què és el prompt engineering**](https://cloud.google.com/discover/what-is-prompt-engineering){ target="_blank" rel="noopener" } · *Google Cloud*  
  Introducció al concepte, tipus de prompts i bones pràctiques.
- [**Introducció al disseny de prompts**](https://colab.research.google.com/github/GoogleCloudPlatform/generative-ai/blob/main/gemini/prompts/intro_prompt_design.ipynb?hl=es-419){ target="_blank" rel="noopener" } · *Google Cloud · Colab*  
  Quadern pràctic per provar pas a pas les tècniques d'aquesta pàgina.
- [**Guía de Ingeniería de Prompt**](https://www.promptingguide.ai/es){ target="_blank" rel="noopener" } · *DAIR.AI*  
  Guia oberta i en castellà amb els conceptes bàsics, tècniques i exemples.
