# Projecte 3. Activitats

## Activitat 1. Dimensionament d'un node del clúster

**Objectiu:** caracteritzar un node real del clúster, predir quins models hi poden funcionar a partir dels càlculs de la [teoria](teoria.md#maquinari-i-dimensionament) i contrastar aquestes prediccions amb mesures reals.

| | |
|---|---|
| **Durada** | ≈ 1 h 15 min |
| **Agrupament** | Per parelles, un node per parella |
| **Eines** | `lscpu`, `dmidecode`, `numactl`, `sysbench`, `llama-bench` (llama.cpp) |

Els servidors del clúster són de **cap al 2009**. Això vol dir que **no podeu donar res per fet**: el tipus de memòria, la seva velocitat real i les instruccions que suporta la CPU condicionen molt el resultat. Tota la pràctica segueix el mateix esquema: **primer prediu, després mesura i finalment explica la diferència**.

!!! note "Abans de començar"
    El professor us indicarà el node assignat, la ruta on hi ha **llama.cpp compilat** i la carpeta amb els **models GGUF** (Q4_K_M) de diferents mides. Al llarg de l'enunciat es fan servir aquestes variables, que heu d'adaptar:

    ```bash
    export LLAMA=/ruta/a/llama.cpp/build/bin   # la indica el professor
    export MODELS=/ruta/als/models             # la indica el professor
    ```

---

### Part 1 · Inventari del node (15 min)

Executeu les comandes següents i ompliu la taula.

```bash
lscpu | grep -E 'Model name|Socket|Core|Thread|NUMA|MHz'
grep -o -w 'sse4_2\|avx\|avx2\|avx512f' /proc/cpuinfo | sort -u
free -h
sudo dmidecode -t memory | grep -E 'Type:|Speed|Size|Locator' | grep -v 'No Module'
numactl --hardware
```

| Dada | Valor |
|---|---|
| Model de CPU | |
| Sòcols × nuclis × fils | |
| Instruccions vectorials (SSE4.2 / AVX / AVX2 / AVX-512) | |
| Memòria total | |
| Tipus de memòria (DDR2 FB-DIMM / DDR3 / DDR4) | |
| Mòduls instal·lats (nombre i mida) | |
| Velocitat **nominal** dels mòduls (`Speed`) | |
| Velocitat **configurada** (`Configured Memory Speed` / `Configured Clock Speed`) | |
| Canals de memòria per sòcol (vegeu la fitxa de la CPU) | |
| Nodes NUMA i memòria de cada node | |

!!! warning "Velocitat nominal ≠ velocitat real"
    En plataformes d'aquesta època, la velocitat de la memòria la limita **la CPU** i **el nombre de mòduls per canal**: com més ranures ocupades, més baixa la freqüència. Si `dmidecode` mostra *Unknown*, busqueu la velocitat màxima de memòria a la fitxa del model de CPU (web del fabricant) i indiqueu-ne la font.

**Preguntes:**

1. La CPU té AVX o AVX2? Per què és important per a llama.cpp? (Repasseu l'apartat [Quina velocitat podem esperar](teoria.md#quina-velocitat-podem-esperar) de la teoria.)
2. Coincideix la velocitat nominal dels mòduls amb la configurada? Si no, quina és la causa?

---

### Part 2 · Ample de banda de memòria: teòric i mesurat (15 min)

**2.1 · Càlcul teòric.** Calculeu l'ample de banda **per sòcol** i **total** amb la velocitat configurada:

```text
Ample de banda (GB/s) = MT/s × 8 bytes × canals / 1000
```

**2.2 · Mesura real.** Mesureu l'ample de banda d'**un sol node NUMA**, fixant la CPU i la memòria al mateix sòcol. Substituïu `N` pel nombre de fils d'un sòcol:

```bash
numactl --cpunodebind=0 --membind=0 \
  sysbench memory --memory-block-size=1M --memory-total-size=50G \
  --memory-oper=read --threads=N run
```

!!! info
    Si el node té un sol sòcol, ometeu `numactl`. Si `sysbench` no hi és, instal·leu-lo amb `sudo apt install sysbench`.

| | Teòric | Mesurat | % del teòric |
|---|---|---|---|
| Un sòcol (node NUMA 0) | | | |
| Total del servidor | | — | — |

**Preguntes:**

3. Quin percentatge del teòric obteniu? És més alt o més baix que el 60–80 % de referència de la teoria? Per què creieu que passa?
4. Repetiu la mesura amb `--membind=1` mantenint `--cpunodebind=0` (CPU en un sòcol i memòria a l'altre). Què passa? Què implica per a un LLM?

---

### Part 3 · El model més gran que és **útil** (10 min)

Al vostre node segurament hi **cap** un model molt gran, però això no vol dir que sigui **útil**. Treballareu amb tres condicions alhora:

- Reserveu **4 GB** per al sistema i una KV cache per a **8K tokens** de context (≈ 1 GB per a un model de 8B; menys per als petits).
- El model ha de cabre a la memòria d'**un sol node NUMA** (justifiqueu per què amb el resultat de la pregunta 4).
- La velocitat de generació estimada ha de ser **≥ 5 tok/s**, el mínim per a una lectura còmoda.

Feu servir la taula de mides de l'apartat [Quanta memòria necessita un model](teoria.md#quanta-memoria-necessita-un-model) de la teoria i l'ample de banda **mesurat**:

```text
Tokens/s màxims ≈ Ample de banda mesurat (GB/s) / Mida del model (GB)
```

| Condició | Model més gran que la compleix |
|---|---|
| Només hi cap a la memòria total | |
| Hi cap en un node NUMA | |
| Hi cap en un node NUMA **i** ≥ 5 tok/s | |

**Pregunta:**

5. Quina diferència hi ha entre el model més gran que hi cap i el més gran que és útil? Quina conclusió en traieu sobre la RAM d'aquests servidors?

---

### Part 4 · Predicció i mesura amb `llama-bench` (20 min)

**4.1 · Predicció.** Abans de mesurar res, ompliu la columna *Predicció* amb la fórmula anterior per als quatre models.

**4.2 · Mesura.** Executeu `llama-bench` per a cada model. Mesura dues coses per separat: `pp512` (processament d'un prompt de 512 tokens) i `tg128` (generació de 128 tokens).

```bash
numactl --cpunodebind=0 --membind=0 \
  $LLAMA/llama-bench -m $MODELS/<model>.gguf -p 512 -n 128 -t N
```

!!! tip
    Comenceu pel model més petit. Amb el de 14B la prova pot trigar uns quants minuts: aprofiteu per avançar la Part 5 mentre s'executa.

| Model (Q4_K_M) | Mida (GB) | Generació: predicció (tok/s) | Generació: `tg128` mesurat | % de la predicció | Prompt: `pp512` mesurat |
|---|---|---|---|---|---|
| ≈ 1B | | | | | |
| ≈ 3B | | | | | |
| ≈ 8B | | | | | |
| ≈ 14B | | | | | |

**4.3 · Fils.** Amb el model de 3B, proveu diferents nombres de fils en una sola execució:

```bash
numactl --cpunodebind=0 --membind=0 \
  $LLAMA/llama-bench -m $MODELS/<model-3B>.gguf -p 512 -n 128 -t 1,2,4,8
```

**Preguntes:**

6. La fórmula prediu bé la generació (`tg`)? I el processament del prompt (`pp`)? Per què una fase s'ajusta a la fórmula i l'altra no?
7. Com evolucionen `pp` i `tg` en augmentar els fils? Arriba un punt en què més fils no milloren la generació? Relacioneu-ho amb l'ample de banda.
8. Quina relació hi ha entre l'absència d'AVX i els valors de `pp` que obteniu?

---

### Part 5 · Conclusions: dos casos d'ús (15 min)

**5.1 · Xat intern.** El departament vol oferir un xat amb LLM a **25 usuaris**, i cadascú necessita almenys **5 tok/s**. Fent una estimació conservadora (la velocitat del node es reparteix entre els usuaris simultanis):

- Quants usuaris simultanis pot atendre **un node** amb el model de 1B? I amb el de 3B?
- Quants nodes del clúster caldrien per a 25 usuaris? Dibuixeu l'arquitectura: un servidor de models per node i un **balancejador de càrrega** al davant.
- És una solució raonable? Quina alternativa proposaríeu?

**5.2 · Anàlisi nocturna de logs.** Cada nit cal analitzar un fitxer de **10.000 línies** de logs (≈ 35 tokens per línia). El context és de 8K tokens: trossegeu el fitxer en blocs de **6.000 tokens** d'entrada, i cada bloc genera un resum de **200 tokens**.

```text
Temps per bloc  ≈ tokens d'entrada / pp  +  tokens generats / tg
Temps total     ≈ nombre de blocs × temps per bloc
```

| Model | Blocs | Temps total en 1 node | Cap en una finestra nocturna de 8 h? |
|---|---|---|---|
| 3B | | | |
| 8B | | | |

- Quina fase domina el temps total: el processament del prompt o la generació? Per què és diferent del cas del xat?
- Proposeu dues maneres de reduir el temps sense canviar de maquinari (penseu en el clúster i en el pre-processament dels logs abans d'enviar-los al model).
- Quin model recomaneu per a aquest cas? Justifiqueu-ho.

**5.3 · Conclusió final.** En un màxim de 10 línies: per a què serveix aquest clúster com a infraestructura d'LLMs i per a què no? Què n'espereu del servidor GPU? Guardeu les vostres mesures: les comparareu amb les del servidor GPU a la Pràctica IV.

---

### Avaluació

| Part | Criteri | Punts |
|---|---|---|
| 1 | Inventari complet i correcte, preguntes 1–2 | 1,5 |
| 2 | Càlcul teòric, mesura real i interpretació (preguntes 3–4) | 2 |
| 3 | Aplicació correcta de les tres condicions i pregunta 5 | 1,5 |
| 4 | Predicció, mesures, comparativa i preguntes 6–8 | 3 |
| 5 | Càlculs dels dos casos d'ús, propostes justificades i conclusió | 2 |

**Entrega:** un document per parella amb les taules completes, les captures de les comandes i les respostes, en format PDF a través del [Moodle](https://frontal.ies-sabadell.cat/eva/course/view.php?id=73).
