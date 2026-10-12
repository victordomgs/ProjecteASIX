---
hide:
  - toc
---

<div class="hero" markdown>

<span class="eyebrow">M0379 · Projecte · 2n ASIX</span>

# Aplicació i desplegament d'IA en sistemes

<p class="lead">Bloc de 49 hores que té com a objectiu desenvolupar les competències de l’alumnat en l’ús d’eines d’intel·ligència artificial i el desplegament de models d’IA mitjançant la combinació d’eines de processament de dades, virtualització de contenidors i creació de servidors, tot tenint en compte les necessitats reals i les oportunitats de negoci del sector productiu actual.
</p>

</div>

## Projectes

<div class="grid cards" markdown>

-   :material-lightbulb-on-outline:{ .lg .middle } **P0 · Introducció a la IA**

    ---

    Què és la IA, els seus paradigmes i on s'aplica. El punt de partida comú.

    [:octicons-arrow-right-24: Comença](p0-introduccio-ia/index.md)

-   :material-console-line:{ .lg .middle } **P1 · Prompt engineering**

    ---

    Prompts per generar scripts, configuracions, diagnòstics i documentació de sistemes.

    [:octicons-arrow-right-24: Veure projecte](p1-prompt-engineering/index.md)

-   :material-robot-outline:{ .lg .middle } **P2 · Agents amb OpenClaw**

    ---

    Disseny i orquestració d'agents que automatitzen tasques de manera controlada.

    [:octicons-arrow-right-24: Veure projecte](p2-openclaw/index.md)

-   :material-server-outline:{ .lg .middle } **P3 · LLMs locals**

    ---

    Desplegament de models de llenguatge en infraestructura pròpia i exposició com a servei.

    [:octicons-arrow-right-24: Veure projecte](p3-llms-locals/index.md)

-   :material-database-search-outline:{ .lg .middle } **P4 · BBDD vectorials**

    ---

    ChromaDB i recuperació d'informació (RAG) sobre documentació pròpia.

    [:octicons-arrow-right-24: Veure projecte](p4-chromadb/index.md)

-   :material-text-search:{ .lg .middle } **P5 · Anàlisi de logs amb LLM**

    ---

    Pipeline que recull logs, els analitza amb un LLM i genera alertes i informes.

    [:octicons-arrow-right-24: Veure projecte](p5-logs-llm/index.md)

-   :material-cloud-outline:{ .lg .middle } **P6 · ML al cloud (AzureML)**

    ---

    Entrenament, registre i desplegament d'un model com a endpoint a Azure.

    [:octicons-arrow-right-24: Veure projecte](p6-azureml/index.md)

-   :material-bookshelf:{ .lg .middle } **Recursos**

    ---

    Glossari, preparació de l'entorn i enllaços útils per a tot el curs.

    [:octicons-arrow-right-24: Obrir](recursos/index.md)

</div>

## Avaluació

Aquest bloc equival al **15 % de la nota final del mòdul de Projecte (M0379)** de 2n d'ASIX. Dins del bloc, la nota es reparteix així:

| Projecte | Pes al bloc | Pes a la nota final del mòdul |
|---|---|---|
| P0 · Introducció a la IA | No avaluable | — |
| P1 · Prompt engineering | No avaluable | — |
| P2 · Agents amb OpenClaw | 20 % | 3 % |
| P3 · LLMs locals | 20 % | 3 % |
| P4 · BBDD vectorials | 20 % | 3 % |
| P5 · Anàlisi de logs amb LLM | 20 % | 3 % |
| P6 · ML al cloud (AzureML) | 20 % | 3 % |
| **Total** | **100 %** | **15 %** |

Els criteris i la rúbrica de cada projecte es detallen a la seva pàgina d'avaluació.

## Ús responsable de la IA

En aquest bloc la IA és alhora l'objecte d'estudi i una eina de treball. Per fer-ne un ús professional i honest, cal seguir aquests principis:

<div class="grid cards" markdown>

-   :material-check-decagram-outline:{ .lg .middle } **Verifica sempre**

    ---

    Els models s'equivoquen i s'inventen comandes, opcions o paràmetres (al·lucinacions). Contrasta les respostes amb la documentació oficial (`man`, documentació del fabricant) abans de fer-les servir.

-   :material-console:{ .lg .middle } **Entén abans d'executar**

    ---

    No executis cap comanda ni script generat per una IA que no entenguis. Prova'l primer en un entorn de proves, mai directament en producció ni amb privilegis de `root`.

-   :material-shield-lock-outline:{ .lg .middle } **Protegeix les dades**

    ---

    No enganxis mai a un model extern contrasenyes, claus API, certificats, configuracions internes ni dades personals (RGPD). Anonimitza logs i fitxers abans de compartir-los.

-   :material-account-key-outline:{ .lg .middle } **Mínim privilegi**

    ---

    Els agents que executen accions al sistema han de tenir només els permisos imprescindibles, amb entorns aïllats i registre d'auditoria. Les claus API es guarden fora del codi i del repositori.

-   :material-text-box-check-outline:{ .lg .middle } **Sigues transparent**

    ---

    Indica a les entregues quan i com has fet servir la IA (quin model i per a què). Presentar com a propi un treball generat per IA sense declarar-ho es considera còpia.

-   :material-head-lightbulb-outline:{ .lg .middle } **Aprèn, no delegues**

    ---

    La IA t'ha d'ajudar a entendre i a treballar millor, no a substituir el teu raonament. Has de ser capaç d'explicar i defensar tot el que entreguis.

</div>

> [!WARNING]
> L'ús de la IA en activitats i proves on no estigui permès, o sense declarar-lo quan sigui obligatori, es tractarà segons els criteris del departament d'Informàtica i pot suposar la qualificació negativa de l'activitat.
