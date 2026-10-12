# Estratègia de prompting

## Introducció

Encara que no hi ha una única manera de redactar instruccions, sí que existeixen estratègies habituals per condicionar i millorar les respostes del model. Tot i així, fer proves i avaluacions constants és clau per aconseguir el millor rendiment.

Els models de llenguatge (LLM) s'entrenen amb volums massius de text per aprendre patrons i associacions lingüístiques. Quan reben una instrucció (*prompt*), prediuen com ha de continuar el text, actuant com una eina d'autocompletat molt avançada. Per això, en dissenyar les instruccions, convé tenir presents els factors que influeixen en aquestes prediccions.

## Flux de treball

L'enginyeria d'instruccions **és un procés iteratiu i empíric** clau per optimitzar el rendiment del model. En dissenyar cada instrucció, cal definir amb precisió els objectius i el resultat esperat, i avaluar-los de manera sistemàtica per detectar oportunitats de millora.

El flux de treball típic s'estructura de la manera següent:

![Flux de treball de l'enginyeria d'instruccions](img/flux-treball.png){ loading=lazy }

## Com crear una instrucció eficaç

<div class="grid cards" markdown>

-   **Contingut**

    ---

    Per resoldre una tasca, el model necessita totes les dades pertinents: instruccions clares de l'acció, exemples de referència, informació de context i altres requisits específics.

-   **Estructura**

    ---

    Encara que s'aporti tota la informació necessària, organitzar-la de manera clara ajuda el model a interpretar-la millor. L'ordre dels elements, les etiquetes i l'ús de delimitadors influeixen directament en la qualitat de la resposta.

</div>

## Components d'una instrucció

### Components essencials

| Component | Descripció | Exemple |
| --- | --- | --- |
| **Objectiu** | Què vols que assoleixi el model. Sigues específic i inclou-hi també els objectius generals. També s'anomena «missió». | El teu objectiu és ajudar els estudiants amb problemes matemàtics sense donar-los la resposta directament. |
| **Instruccions** | Indicacions pas a pas sobre com dur a terme la tasca. També s'anomenen «tasca», «passos» o «pautes». | 1. Comprèn què planteja el problema. 2. Identifica on s'ha encallat l'alumne. 3. Proporciona una pista per al pas següent del problema. |

### Components opcionals

| Component | Descripció | Exemple |
| --- | --- | --- |
| **Instruccions del sistema** | Directrius tècniques o d'entorn per controlar o modificar el comportament del model al llarg d'un conjunt de tasques. En moltes API de models s'especifiquen mitjançant un paràmetre dedicat. | Ets un expert en programació especialitzat a renderitzar codi per a interfícies frontend. Quan descrigui un component web que vull desenvolupar, mostra el codi HTML i CSS necessari. No incloguis explicacions per a aquest codi. Afegeix també alguns suggeriments de disseny de la interfície. |
| **Rol / persona** | Com qui o què actua el model. També es coneix com a «rol» o «visió». | Ets un tutor de matemàtiques que ajuda els alumnes amb els deures. |
| **Restriccions** | Condicions que el model ha de complir en generar una resposta, incloent-hi allò que pot i no pot fer. També s'anomenen «barreres», «límits» o «controls». | No donis la resposta directament a l'alumne; ofereix-li pistes sobre el pas següent. Si està del tot encallat, proporciona-li els passos detallats per arribar a la solució. |
| **To** | El to de la resposta. També es pot condicionar especificant un rol determinat. També s'anomena «estil», «veu» o «mode». | Respon d'una manera informal i tècnica. |
| **Context** | Qualsevol informació que el model necessiti consultar per dur a terme la tasca. També s'anomena «informació de fons», «documents» o «dades d'entrada». | Una còpia de les planificacions didàctiques de matemàtiques de l'alumne. |
| **Passos de raonament** | Indicar al model que exposi el seu raonament pas a pas, cosa que en pot millorar la capacitat analítica i lògica. També es coneix com a *chain-of-thought*. | Explica el teu raonament pas a pas. |
| **Exemples de mostra (*few-shot*)** | Mostres de com ha de ser la resposta per a una instrucció determinada. També s'anomenen «exemplars» o «mostres». | **Entrada:** Estic intentant calcular quantes pilotes de golf caben en una caixa d'un metre cúbic. He convertit el volum a centímetres cúbics i l'he dividit pel volum d'una pilota, però el sistema diu que la resposta és incorrecta. **Sortida:** Les pilotes de golf són esfèriques i no es poden encaixar amb una eficiència del 100%. Els teus càlculs no tenen en compte la densitat màxima d'empaquetament d'esferes. |
| **Format de la resposta** | El format concret en què vols rebre la resposta: JSON, taula, Markdown, paràgraf, llista, paraules clau, resum executiu, etc. També s'anomena «estructura», «presentació» o «disseny». | Formata la resposta en Markdown. |
| **Resum** | Repetició concisa dels punts clau al final de la instrucció, especialment les restriccions i el format de sortida. | No donis la resposta directament; ofereix pistes. Dona sempre format Markdown a la teva resposta. |
| **Mecanismes de protecció (*guardrails*)** | Mesures per ancorar les respostes a la missió del model i evitar desviaments o usos indeguts. També s'anomenen «regles de seguretat». | — |

## Plantilla d'instrucció

Aquesta plantilla mostra com es podria veure una instrucció ben estructurada:

```xml title="plantilla-prompt.txt"
<OBJECTIVE_AND_PERSONA>
You are a [insert a persona, such as a "math teacher" or "automotive expert"]. Your task is to...
</OBJECTIVE_AND_PERSONA>

<INSTRUCTIONS>
To complete the task, you need to follow these steps:
1.
2.
...
</INSTRUCTIONS>

------------- Optional Components ------------

<CONSTRAINTS>
Dos and don'ts for the following aspects
1. Dos
2. Don'ts
</CONSTRAINTS>

<CONTEXT>
The provided context
</CONTEXT>

<OUTPUT_FORMAT>
The output format must be
1.
2.
...
</OUTPUT_FORMAT>

<FEW_SHOT_EXAMPLES>
Here we provide some examples:
1. Example #1
Input:
Thoughts:
Output:
...
</FEW_SHOT_EXAMPLES>

<RECAP>
Re-emphasize the key aspects of the prompt, especially the constraints, output format, etc.
</RECAP>
```

## Llista de verificació d'una instrucció

Si una instrucció no funciona com s'espera, és molt probable que hi hagi un problema en com s'ha redactat o definit. Aquesta llista ajuda a identificar possibles problemes i millorar-ne el rendiment.

<figure>
  <video src="../img/aireading.mp4" poster="../img/aireading.jpg" width="320" autoplay loop muted playsinline></video>
</figure>

### Problemes de redacció

- **Errors tipogràfics:** revisa les paraules clau que defineixen la tasca (per exemple, *sumarize* en lloc de *summarize*), els termes tècnics o els noms propis, ja que les faltes d'ortografia poden empitjorar el rendiment del model.
- **Gramàtica:** si una frase és difícil d'interpretar, conté oracions encavallades, té discordances entre subjecte i verb o resulta forçada, és probable que el model no entengui correctament la instrucció.
- **Puntuació:** comprova l'ús de comes, punts, cometes i altres signes; una puntuació incorrecta pot fer que el model malinterpreti la instrucció.
- **Ús d'argot no definit:** evita utilitzar termes tècnics, acrònims o sigles d'un àmbit concret com si tinguessin un significat universal, tret que es defineixin explícitament.
- **Claredat:** si et sorgeixen dubtes sobre l'abast, els passos concrets o les assumpcions implícites, el més probable és que la instrucció no sigui prou clara.
- **Ambigüitat:** evita termes subjectius o relatius sense una definició concreta i mesurable. Defineix restriccions objectives (per exemple, «redacta un resum de 3 frases o menys» en comptes d'«escriu un resum breu»).
- **Manca d'informació clau:** si la tasca exigeix conèixer un document, una política d'empresa, l'historial d'un usuari o un conjunt de dades, inclou aquesta informació de manera explícita.
- **Tria inadequada de paraules:** revisa la instrucció per detectar expressions innecessàriament complexes, vagues o recarregades, ja que podrien confondre el model.
- **Segona revisió:** si el model continua sense oferir bons resultats, demana a una altra persona que revisi la instrucció.

### Problemes amb les instruccions i els exemples

- **Manipulació explícita:** elimina qualsevol text aliè a la tasca que intenti condicionar el rendiment mitjançant apel·lacions emocionals, afalacs o pressió artificial. Els primers models podien respondre millor a frases com «passaran coses molt greus si no ho fas bé», però en els models actuals aquesta pràctica ja no funciona i, sovint, empitjora el resultat.
- **Instruccions i exemples contradictoris:** detecta possibles contradiccions lògiques, ja sigui entre diferents pautes o entre una pauta i els exemples aportats.
- **Instruccions i exemples redundants:** comprova que no s'estigui repetint la mateixa ordre amb paraules lleugerament diferents sense aportar cap informació nova.
- **Instruccions i exemples irrellevants:** si pots eliminar una instrucció o un exemple sense afectar la resposta del model, vol dir que és prescindible.
- **Manca d'exemples de referència (*few-shot*):** si la tasca és complexa, exigeix un format concret o demana un to molt determinat, inclou exemples que mostrin clarament una entrada tipus i la sortida esperada.
- **Manca d'especificació del format de sortida:** no deixis que el model dedueixi l'estructura de la resposta. Defineix el format de manera explícita i reflecteix-lo també en els exemples.
- **Absència de definició de rols:** si vols que el model adopti un perfil concret, estableix aquest rol a les instruccions del sistema.

### Problemes de disseny d'instruccions i de sistemes

- **Casuística no especificada:** defineix com gestionar els casos límit (*edge cases*), les entrades inesperades o la manca de dades, en lloc de pressuposar que la informació sempre estarà present i ben estructurada.
- **Tasca fora de l'abast del model:** evita demanar accions per a les quals el model té limitacions fonamentals conegudes.
- **Sobrecàrrega de tasques:** si demanes massa operacions en una sola passada (per exemple: resumir, extreure entitats, traduir i redactar un correu), el resultat perdrà qualitat. Divideix-les en instruccions independents encadenades.
- **Formats de dades no estàndard:** quan el resultat l'hagi de processar una màquina, utilitza formats estàndard com JSON, XML, Markdown o YAML. Si necessites un format propietari, demana una sortida estàndard i fes la conversió després amb codi.
- **Ordre incorrecte de la cadena de pensament (*chain-of-thought*):** evita exemples on el model doni la resposta final abans d'haver desenvolupat el raonament.
- **Mode de pensament (*thinking*) enfront de raonament manual:** amb models que tenen pensament intern, prescindeix d'instruccions massa detallades sobre com ha de raonar i comprova si el seu procés nadiu dona millors resultats.
- **Lògica dispersa o referències encreuades:** evita estructures condicionals o no lineals que obliguin el model a relacionar fragments d'instruccions escampats pel text.
- **Risc d'injecció d'instruccions (*prompt injection*):** aplica mecanismes de protecció i delimitadors clars quan incorporis dades externes o entrades d'usuaris no fiables, ja que poden ser una vulnerabilitat de seguretat greu.

> [!TIP]
> En administració de sistemes, aquesta llista és especialment útil quan demanes scripts o configuracions: especifica la distribució i versió, el format de sortida (Bash, YAML, JSON...) i com s'han de gestionar els errors.
