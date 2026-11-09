# Projecte 3. Activitats

## Activitat 1. Dimensionament d'un node del clúster

**Objectiu:** caracteritzar un node real del clúster, predir quins models hi poden funcionar a partir dels càlculs de la [teoria](teoria.md#maquinari-i-dimensionament) i contrastar aquestes prediccions amb mesures reals.

En aquesta activitat hauràs de:

1. Inventariar el maquinari real d’un servidor amb eines de sistema i a interpretar-lo.
2. Identificar quin recurs limita un LLM.
3. Aplicar les fórmules de dimensionament de la teoria a un cas real. 
4. Distingir entre el model que hi cap i el model útil, i entendre que la quantització és el compromís entre qualitat, memòria i velocitat.
5. Prendre i justificar decisions tècniques segons el cas d’ús i el maquinari disponible.

[:material-file-word: Descarrega l'enunciat (.docx)](fitxers/P3_Maquinari-i-dimensionament-I_2026-27.docx){ .md-button .md-button--primary download="P3_Maquinari-i-dimensionament-I_2026-27.docx" }

**Entrega:** un document per grup amb tot el procés documentat (amb captures), en format PDF a través del [Moodle](https://frontal.ies-sabadell.cat/eva/course/view.php?id=73).

## Activitat 2. Preparació entorn - Ubuntu 24.04 LTS

**Objectiu:** preparar l'entorn que farem servir durant tota la part de desplegament.

[:material-file-word: Descarrega l'enunciat (.docx)](fitxers/P3_Preparació-entorn_2026-27.docx){ .md-button .md-button--primary download="P3_Preparació-entorn_2026-27.docx" }

**Entrega:** checkpoint, cridar al professor.

## Activitat 3. Desplegament S1 - GPU A1

**Objectiu:** preparar un servidor real perquè pugui executar contenidors amb accés a la GPU, verificant-ne la seguretat, el driver i la compatibilitat amb la plataforma descrita a l’arquitectura.

En aquesta activitat hauràs de:

- Gestionar els usuaris administradors d’un servidor amb criteris de seguretat i traçabilitat.
- Aplicar el principi de mínim privilegi a la xarxa amb un tallafoc.
- Verificar que el driver de la GPU funciona i entendre com hi intervé Secure Boot.
- Interpretar les versions de driver i de CUDA, i relacionar-les amb les imatges de contenidor que es poden fer servir.
- Instal·lar Docker i donar accés a la GPU als contenidors, entenent el paper de cada capa.
- Comprovar l’estat real del sistema en lloc de donar-lo per fet, i documentar-lo amb evidències.

[:material-file-word: Descarrega l'enunciat (.docx)](fitxers/P3_Desplegament-S1_2026-27.docx){ .md-button .md-button--primary download="P3_Desplegament-S1_2026-27.docx" }

**Entrega:** checkpoint, cridar al professor.

## Activitat 4. Desplegament S2 - GPU A1

**Objectiu:** publicació segura de serveis i desplegament del motor d’inferència.

En aquesta activitat hauràs de:

- Comprovar que Docker es salta les regles d’UFW i explicar-ne el motiu.
- Publicar serveis aplicant el principi de mínim privilegi: només el proxy invers exposa ports al host.
- Comunicar contenidors entre ells a través de xarxes Docker internes, sense exposar cap port.
- Calcular si un model de llenguatge hi cap a la GPU i entendre com es reparteix la VRAM entre els pesos i la KV cache.

[:material-file-word: Descarrega l'enunciat (.docx)](fitxers/P3_Desplegament-S2_2026-27.docx){ .md-button .md-button--primary download="P3_Desplegament-S2_2026-27.docx" }

## Activitat 5. Desplegament S3 - GPU A1

**Objectiu:** posar una passarel·la d’API davant del model per controlar qui l’utilitza, com i quant.

En aquesta activitat hauràs de:

- Desplegar un servei connectat a diverses xarxes Docker, i aïllar una base de dades en una xarxa interna sense sortida a Internet.
- Definir àlies de models per separar usos diferents sobre un mateix motor.
- Definir per què cap usuari ha de parlar directament amb el model i quin paper té una passarel·la d'API. 

[:material-file-word: Descarrega l'enunciat (.docx)](fitxers/P3_Desplegament-S3_2026-27.docx){ .md-button .md-button--primary download="P3_Desplegament-S2_2026-27.docx" }
