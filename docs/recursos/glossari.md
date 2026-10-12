# Glossari

Termes clau que apareixen al llarg dels projectes, en ordre alfabètic.

## A

**Agent d'IA**
:   Sistema basat en un LLM capaç de planificar subtasques, fer servir eines (executar comandes, llegir fitxers, cridar API) i actuar de manera autònoma per assolir un objectiu. Vegeu el [Projecte 2](../p2-openclaw/index.md).

**Al·lucinació**
:   Resposta d'un model que sembla correcta i convincent però és falsa o inventada: una comanda que no existeix, una opció inexistent, una dada o una font falsa.

**API (Application Programming Interface)**
:   Interfície que permet a un programa comunicar-se amb un altre. Els models d'IA al núvol s'acostumen a consumir mitjançant una API REST, autenticant-se amb una clau API.

**Aprenentatge automàtic (*machine learning*, ML)**
:   Branca de la IA en què els sistemes aprenen patrons a partir de dades en lloc de seguir regles programades explícitament.

**Aprenentatge per reforç**
:   Tipus d'aprenentatge en què un agent aprèn per prova i error, rebent recompenses quan l'acció és correcta i penalitzacions quan no ho és.

**Aprenentatge supervisat / no supervisat**
:   En el supervisat, el model s'entrena amb dades etiquetades (entrada i resposta correcta). En el no supervisat, busca patrons o agrupacions en dades sense etiquetar.

**AzureML (Azure Machine Learning)**
:   Servei de Microsoft Azure per entrenar, registrar, desplegar i gestionar models d'aprenentatge automàtic al núvol. Vegeu el [Projecte 6](../p6-azureml/index.md).

## B

**Base de dades vectorial**
:   Base de dades dissenyada per emmagatzemar *embeddings* i fer-hi cerques per similitud semàntica. N'és un exemple ChromaDB. Vegeu el [Projecte 4](../p4-chromadb/index.md).

**Biaix**
:   Tendència sistemàtica d'un model a afavorir o perjudicar determinats resultats o col·lectius, normalment heretada de les dades d'entrenament.

## C

**Cadena de pensament (*chain-of-thought*, CoT)**
:   Tècnica de prompting que demana al model que raoni pas a pas abans de donar la resposta final. Millora els resultats en problemes lògics o de diagnòstic.

**ChromaDB**
:   Base de dades vectorial de codi obert, lleugera i fàcil de desplegar, molt utilitzada en sistemes RAG.

**Clau API (*API key*)**
:   Credencial secreta que identifica i autoritza qui fa les crides a una API. No s'ha de publicar mai al codi ni als repositoris.

**Context**
:   Informació que s'afegeix a una instrucció perquè el model la consulti o s'hi basi en respondre: documents, taules, logs, etc.

## E

***Embedding***
:   Representació d'un text (o imatge) com un vector de números que en captura el significat. Textos amb significat semblant tenen vectors propers.

***Endpoint***
:   Adreça (URL) a través de la qual s'accedeix a un servei. En ML, un model desplegat s'exposa com un *endpoint* al qual s'envien peticions de predicció.

## F

***Few-shot* / *zero-shot***
:   *Zero-shot*: demanar una tasca al model sense cap exemple. *Few-shot*: incloure-hi uns quants exemples d'entrada i sortida perquè el model imiti el format i l'estil.

***Fine-tuning* (ajust fi)**
:   Procés de reentrenar un model ja entrenat amb un conjunt de dades específic per especialitzar-lo en una tasca o domini.

**Finestra de context**
:   Quantitat màxima de text (mesurada en *tokens*) que un model pot tenir en compte alhora, sumant la instrucció, l'historial i la resposta. Quan se supera, el model «oblida» el principi de la conversa.

## G

**Gateway (OpenClaw)**
:   Procés central d'OpenClaw que s'executa en segon pla, connecta els canals de missatgeria, gestiona les sessions i orquestra els agents i les crides als models.

***Guardrails* (mecanismes de protecció)**
:   Regles i controls que limiten el comportament d'un model o agent per evitar respostes o accions no desitjades.

## I

**IA feble / IA forta**
:   La IA feble està dissenyada per a una tasca concreta. La IA forta, encara teòrica, igualaria la intel·ligència humana en qualsevol tasca.

**IA generativa**
:   Models capaços de generar contingut nou (text, imatges, codi, àudio) a partir d'una instrucció.

**Inferència**
:   Ús d'un model ja entrenat per obtenir una resposta o predicció a partir d'una entrada nova.

**Injecció d'instruccions (*prompt injection*)**
:   Atac en què un usuari o una dada externa introdueix instruccions que fan que el model ignori les seves directrius originals (per exemple, revelar el *prompt* del sistema o claus).

**Instruccions del sistema (*system prompt*)**
:   Directrius prèvies a l'entrada de l'usuari que defineixen el rol, el to i les restriccions del model durant tota la conversa.

## L

**LLM (*Large Language Model*)**
:   Model de llenguatge de grans dimensions entrenat amb volums massius de text per predir la continuació més probable d'un text. És la base de xats com ChatGPT, Gemini o Claude.

**LLM local**
:   Model de llenguatge executat en infraestructura pròpia en lloc d'un servei al núvol, de manera que les dades no surten de l'organització. Vegeu el [Projecte 3](../p3-llms-locals/index.md).

## M

**Mínim privilegi**
:   Principi de seguretat segons el qual un usuari, servei o agent només ha de tenir els permisos estrictament necessaris per fer la seva feina.

**MLOps**
:   Conjunt de pràctiques per desplegar, monitoritzar i mantenir models d'aprenentatge automàtic en producció, de manera semblant al DevOps.

**Model fundacional**
:   Model de gran escala entrenat amb dades molt generals que serveix de base per a moltes tasques diferents.

## P

**Paràmetres**
:   Valors interns (pesos) que el model ajusta durant l'entrenament. La mida d'un model s'expressa sovint en milers de milions de paràmetres (per exemple, 7B).

***Prompt* (instrucció)**
:   Text d'entrada que s'envia a un model per obtenir-ne una resposta.

***Prompt engineering* (enginyeria d'instruccions)**
:   Disciplina de dissenyar i optimitzar els *prompts* per obtenir respostes precises, útils i en el format desitjat. Vegeu el [Projecte 1](../p1-prompt-engineering/index.md).

## Q

**Quantització**
:   Tècnica que redueix la precisió numèrica dels pesos d'un model (per exemple, de 16 a 4 bits) perquè ocupi menys memòria i s'executi en maquinari més modest, amb una petita pèrdua de qualitat.

## R

**RAG (*Retrieval-Augmented Generation*)**
:   Arquitectura que, abans de generar la resposta, recupera fragments rellevants d'una base de coneixement (sovint una base de dades vectorial) i els afegeix al context del model. Redueix les al·lucinacions i permet respondre sobre documentació pròpia.

**Regex (expressió regular)**
:   Patró de text que permet cercar, validar o extreure informació de cadenes de manera determinista. Alternativa clàssica a l'IA per processar logs.

## S

***Sandboxing***
:   Execució d'un procés en un entorn aïllat i restringit perquè no pugui afectar la resta del sistema.

**Sobreajustament (*overfitting*)**
:   Quan un model aprèn massa bé les dades d'entrenament, incloent-hi el soroll, i després funciona malament amb dades noves.

**SIEM (*Security Information and Event Management*)**
:   Plataforma que recull, normalitza i correlaciona logs i esdeveniments de seguretat de diferents fonts per detectar incidents.

## T

**Temperatura**
:   Paràmetre que controla l'aleatorietat de les respostes d'un model. Valors baixos donen respostes més previsibles; valors alts, més variades i creatives.

***Token***
:   Unitat mínima de text que processa un model (una paraula, part d'una paraula o un signe). Els límits de context i els costos de les API es mesuren en *tokens*.

**Transformer**
:   Arquitectura de xarxa neuronal presentada el 2017 a l'article *Attention Is All You Need*, basada en el mecanisme d'atenció. És la base dels LLM actuals.

## X

**Xarxa neuronal**
:   Model format per capes de nodes interconnectats, inspirat en el cervell, que aprèn patrons a partir de dades. És la base del paradigma connexionista.
