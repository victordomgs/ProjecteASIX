# Projecte 2. Activitats

## Activitat 1. Desplegament d'OpenClaw

**Objectiu:** desplegar OpenClaw en un servidor Ubuntu i posar en marxa un agent de sistema que executi comandes de diagnòstic de manera autònoma, aplicant el que s'explica a la [teoria](teoria.md).

En aquesta activitat hauràs de:

1. Instal·lar i gestionar entorns d'execució de Node.js en Ubuntu Server.
2. Configurar el servei (*daemon*) d'OpenClaw.
3. Consumir un model LLM mitjançant una API REST.
4. Desplegar i auditar la seguretat d'un agent de sistema capaç d'executar comandes CLI de diagnòstic de forma autònoma.

> [!NOTE]
> Per fer la configuració s'ha de consultar el següent [Gemini Notebook](https://notebook.google.com/notebook/29c26043-21f0-499b-99cb-0f0c88d9ca68?utm_source=gemini_notebook&utm_medium=referral&original_referer=https:%2F%2Fgemini.google.com%23&pli=1). 

[:material-file-word: Descarrega l'enunciat (.docx)](fitxers/P2_Desplegament-OpenClaw-I_2026-27.docx){ .md-button .md-button--primary download="P2_Desplegament-OpenClaw-I_2026-27.docx" }

**Entrega:** un document per grup amb tot el procés documentat (amb captures), en format PDF a través del [Moodle](https://frontal.ies-sabadell.cat/eva/course/view.php?id=73).

## Activitat 2. Agent Camp

**Objectiu:** entendre com el Gateway orquestra diversos agents i aplicar-hi el principi de mínim privilegi, vinculant un servei de missatgeria a un agent concret.

| Tasca | Què es fa | Punts |
|---|---|---|
| 1 | Estructurar els agents, cadascun amb el seu *workspace* i sessions aïllades | 2,5 |
| 2 | Configurar les eines i permisos: qui pot fer servir `exec` i amb quin *sandboxing* | 2,5 |
| 3 | Vincular el servei de missatgeria i definir l'enrutament de canals (*channel-routing*) | 2,5 |
| 4 | Verificar les restriccions, gestionar l'emparellament d'usuaris i revisar els registres d'auditoria del Gateway | 2,5 |

> [!NOTE]
> Per fer el desplegament s'ha de consultar el següent [Gemini Notebook](https://notebook.google.com/notebook/29c26043-21f0-499b-99cb-0f0c88d9ca68?utm_source=gemini_notebook&utm_medium=referral&original_referer=https:%2F%2Fgemini.google.com%23&pli=1). .

[:material-file-word: Descarrega l'enunciat (.docx)](fitxers/P2_Agent-Camp-OpenClaw-I_2026-27.docx){ .md-button .md-button--primary download="P2_Agent-Camp-OpenClaw-I_2026-27.docx" }

**Entrega:** un document per grup amb tot el procés documentat (amb captures), en format PDF a través del [Moodle](https://frontal.ies-sabadell.cat/eva/course/view.php?id=73).
