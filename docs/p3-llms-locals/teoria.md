# Fonaments dels LLMs locals

## Per què executar un LLM en local

### Què vol dir "LLM local"

Un LLM local és un model de llenguatge que s'executa **en infraestructures que controlem nosaltres** (un portàtil, un servidor del CPD, un clúster del centre) en lloc de consumir-lo com a servei d'un proveïdor extern (API de pagament o aplicació web).

El model és, físicament, **un fitxer** (o uns quants) amb milers de milions de paràmetres (pesos). Per fer-lo servir necessitem:

1. **El fitxer del model** (els pesos).
2. **Un motor d'inferència** que carregui els pesos a memòria i generi text (llama.cpp, Ollama, vLLM…).
3. **Maquinari** amb prou memòria i ample de banda per moure aquests pesos.
4. Opcionalment, **una interfície** (API, web, CLI) perquè usuaris o aplicacions hi accedeixin.

!!! tip "Idea clau!"
    Un LLM local és **un servei més** del nostre sistema, igual que un servidor web o una base de dades. S'instal·la, es dimensiona, es monitora, s'assegura i s'actualitza.

### Cloud vs. local: comparativa

| Criteri | LLM al núvol (API) | LLM local |
|---|---|---|
| **Privadesa de les dades** | Les dades surten de l'organització cap a un tercer | Les dades no surten de la nostra xarxa |
| **Compliment normatiu (RGPD)** | Cal analitzar contractes, ubicació dels servidors, transferències internacionals | Control total; més fàcil de justificar |
| **Qualitat del model** | Accés als models més potents del mercat | Models oberts, normalment menys capaços que els millors comercials |
| **Cost** | Pagament per ús (per token). Barat per començar, car a gran escala | Inversió en maquinari + electricitat. Cost marginal per petició gairebé zero |
| **Latència** | Depèn de la xarxa i de la càrrega del proveïdor | Depèn del nostre maquinari |
| **Disponibilitat** | Depèn del proveïdor i d'Internet | Funciona sense connexió exterior |
| **Dependència de proveïdor** | Canvis de preu, de condicions o retirada de models | Els pesos són nostres; podem fixar una versió per sempre |
| **Manteniment** | Nul | Tot és responsabilitat nostra |

### Quan té sentit (i quan no)

**Té sentit quan:**

- Treballem amb **dades sensibles**: logs interns, dades d'alumnat o de clients, configuracions de xarxa, credencials que podrien aparèixer en un fitxer.
- Hi ha un **volum alt i continu** de peticions (p. ex. analitzar tots els logs cada nit).
- Necessitem **funcionar en entorns aïllats** (xarxes industrials, CPD sense sortida a Internet).
- Volem **reproductibilitat**: el mateix model i la mateixa versió durant anys.
- Volem **aprendre** com funciona la infraestructura (el nostre cas!).

**No té sentit quan:**

- Necessitem la màxima qualitat de raonament possible i no hi ha dades sensibles.
- El volum d'ús és molt baix i esporàdic (el maquinari estaria aturat).
- No disposem de maquinari adequat ni de personal per mantenir-lo.

!!! info "Model híbrid"
    En la pràctica, moltes organitzacions fan servir un **model híbrid**: models locals per a dades sensibles i tasques repetitives, i models al núvol per a tasques complexes amb dades no sensibles.

## L'ecosistema de models oberts

### "Open weights" no és el mateix que "codi obert"

| Terme | Què es publica | Exemple de situació |
|---|---|---|
| **Model tancat** | Res. Només s'hi accedeix per API | Els models comercials dels grans proveïdors |
| **Open weights** (pesos oberts) | Els pesos del model per descarregar-los i executar-los | La majoria de models "oberts" populars |
| **Codi obert complet** | Pesos + codi d'entrenament + dades d'entrenament (o com obtenir-les) | Pocs models, majoritàriament de recerca |

La majoria de models que farem servir són **open weights**: podem executar-los, però no sabem exactament amb quines dades s'han entrenat.

### Llicències: llegir abans d'utilitzar

Cada model té la seva llicència, i **no totes permeten el mateix**. Tipus habituals:

- **Llicències permissives estàndard** (Apache 2.0, MIT): ús comercial lliure amb poques condicions.
- **Llicències pròpies del fabricant** (p. ex. les "community licenses"): poden incloure límits d'ús comercial a partir d'un cert nombre d'usuaris, polítiques d'ús acceptable, obligació de citar el model, etc.
- **Llicències no comercials o de recerca**: només per a ús acadèmic.

!!! warning "Llicències"
    Com a administradors, abans de posar un model en producció cal **comprovar la llicència a la seva model card**. Les condicions poden canviar entre versions d'una mateixa família.

### On trobar models

- **Hugging Face** (huggingface.co): el gran repositori de models, conjunts de dades i demostracions. Hi ha els models originals i milers de variants (quantitzades, ajustades…).
- **Biblioteca d'Ollama** (ollama.com/library): models ja preparats per descarregar amb una sola comanda.
- Webs oficials dels fabricants.

**Famílies de models oberts habituals** (la llista canvia ràpidament; consulteu-ne les versions actuals): Llama (Meta), Mistral, Qwen (Alibaba), Gemma (Google), Phi (Microsoft), DeepSeek, entre d'altres.

!!! danger "Procedència dels models"
    Descarregueu models només dels comptes oficials dels fabricants o de quantitzadors amb bona reputació. Un model és un fitxer binari que carregarem als nostres servidors; cal aplicar-hi el mateix criteri que amb qualsevol programari.

### Com llegir una *model card*

La *model card* és la fitxa tècnica del model. Hi hem de buscar:

| Apartat | Què ens diu | Per què ens importa |
|---|---|---|
| Nombre de paràmetres | Mida del model (1B, 7B, 70B…) | Memòria necessària |
| Tipus (base / instruct) | Si està preparat per seguir instruccions | Per a xat cal *instruct* |
| Finestra de context | Quants tokens pot processar alhora | Documents o logs llargs |
| Idiomes | En quines llengües s'ha entrenat | Rendiment en català/castellà |
| Llicència | Condicions d'ús | Legalitat |
| Plantilla de xat | Format exacte de les converses | El motor l'ha d'aplicar bé |
| Limitacions i biaixos | Riscos coneguts | Ús responsable |

### Tipus de models

- **Base**: entrenat només a predir el següent token. Completa text, però no segueix bé instruccions.
- **Instruct / Chat**: model base ajustat per seguir instruccions i mantenir converses. **És el que farem servir habitualment.**
- **Models de codi**: especialitzats en programació (scripts, configuracions).
- **Models d'*embeddings***: no generen text; converteixen un text en un vector de números que representa el seu significat. **Són la base del P4** (bases de dades vectorials).
- **Models multimodals**: accepten imatges a més de text (p. ex. interpretar una captura d'una pantalla d'error).
- **Models de raonament**: generen una cadena de "pensament" abans de respondre. Més precisos en problemes complexos, però molt més lents (generen molts més tokens). A tenir en compte en CPU!

### Paràmetres i context: què impliquen

- **Paràmetres** (B = milers de milions): més paràmetres → més capacitat de "coneixement" i raonament, però més memòria i més lentitud.
  - 1–4B: tasques simples, classificació, resums curts. Viables en CPU.
  - 7–14B: bon equilibri per a ús general. Viables en CPU amb paciència; còmodes en GPU.
  - 30B+: qualitat alta. Requereixen molta memòria; en CPU són lents.
- **Finestra de context** (en tokens; 1 token ≈ ¾ de paraula en anglès, menys en català): és la "memòria de treball" del model en una petició. Inclou el *system prompt*, l'historial, el document que li passem i la resposta. **Més context = més memòria consumida** (ho veurem més endevant).

## Maquinari i dimensionament

### Les dues fases de la inferència

Quan enviem una petició a un LLM passen dues coses molt diferents:

1. **Processament del prompt (*prefill*)**: el model llegeix tots els tokens d'entrada de cop. És una fase **limitada per la capacitat de càlcul** (CPU/GPU). Determina el **temps fins al primer token** (TTFT).
2. **Generació (*decode*)**: el model genera la resposta **token a token**. Per a cada token nou ha de llegir **tots els pesos del model** des de la memòria. És una fase **limitada per l'ample de banda de memòria**.

!!! tip "Conseqüència important"
    Per generar text, **el factor més determinant no és la potència de la CPU, sinó la velocitat de la memòria**.

### On s'executa la inferència

| Plataforma | Memòria | Ample de banda típic | Comentari |
|---|---|---|---|
| CPU servidor antic + DDR3 (3 canals, 1066–1333 MT/s) | RAM del sistema | ≈ 25–32 GB/s per sòcol | Servidors de cap al 2009-2012 |
| CPU + DDR4 (doble canal, 3200 MT/s) | RAM del sistema | ≈ 50 GB/s | Molta memòria, poca velocitat |
| CPU servidor + DDR4 (8 canals, 3200 MT/s) | RAM del sistema | ≈ 200 GB/s | Molt millor gràcies als canals |
| CPU + DDR5 | RAM del sistema | ≈ 80–100 GB/s (escriptori) | Millora respecte DDR4 |
| Memòria unificada (Apple Silicon) | Compartida CPU/GPU | 100–800 GB/s segons el xip | Bon equilibri |
| GPU dedicada | VRAM | 400–3000+ GB/s | Molt ràpida, però VRAM limitada i cara |

**Càlcul de l'ample de banda teòric de la RAM:**

```text
Ample de banda (GB/s) = MT/s × 8 bytes × nombre de canals / 1000
DDR3-1333, 3 canals:  1333 × 8 × 3 / 1000 = 32 GB/s
DDR4-3200, 2 canals:  3200 × 8 × 2 / 1000 = 51,2 GB/s
DDR4-3200, 8 canals:  3200 × 8 × 8 / 1000 = 204,8 GB/s
```

!!! note
    El valor real sempre és inferior al teòric (habitualment un 60–80 %).

### Quanta memòria necessita un model

La memòria total ≈ **pesos del model + KV cache + sobrecost del motor**.

#### a) Pesos del model

```text
Memòria pesos (GB) ≈ paràmetres (en milers de milions) × bits per pes / 8
```

| Model | FP16 (16 bits) | Q8 (≈ 8,5 bits) | Q4_K_M (≈ 4,8 bits) |
|---|---|---|---|
| 3B | ≈ 6 GB | ≈ 3,2 GB | ≈ 1,8 GB |
| 8B | ≈ 16 GB | ≈ 8,5 GB | ≈ 4,8 GB |
| 14B | ≈ 28 GB | ≈ 15 GB | ≈ 8,5 GB |
| 32B | ≈ 64 GB | ≈ 34 GB | ≈ 19 GB |
| 70B | ≈ 140 GB | ≈ 75 GB | ≈ 42 GB |

#### b) KV cache (la memòria del context)

Per no recalcular-ho tot a cada token, el model guarda informació de cada token del context: la **KV cache**. Creix **linealment amb la mida del context** i amb el nombre d'usuaris simultanis.

```text
KV cache (bytes) = 2 × capes × caps_KV × dimensió_cap × bytes_per_valor × tokens_context
```

Exemple orientatiu amb un model de ~8B típic (32 capes, 8 caps KV, dimensió 128, FP16):

```text
Per token: 2 × 32 × 8 × 128 × 2 bytes = 131.072 bytes ≈ 128 KB
Context de 8.192 tokens  → ≈ 1 GB
Context de 32.768 tokens → ≈ 4 GB
4 usuaris × 8.192 tokens → ≈ 4 GB
```

!!! info
    Els valors de capes i caps es troben al fitxer de configuració del model (`config.json` a Hugging Face). No cal memoritzar la fórmula, però sí entendre que **context llarg i concurrència consumeixen memòria**.

#### c) Sobrecost

Reserveu un **10–20 % addicional** per al motor, buffers i sistema operatiu.

### Quina velocitat podem esperar

Durant la generació, cada token requereix llegir tots els pesos. Per tant:

```text
Tokens/s màxims (teòric) ≈ Ample de banda de memòria (GB/s) / Mida del model a memòria (GB)
```

| Escenari | Ample de banda | Model 8B Q4 (≈ 4,8 GB) | Model 32B Q4 (≈ 19 GB) |
|---|---|---|---|
| DDR3 3 canals (1 sòcol) | ≈ 30 GB/s | ≈ 6 tok/s (màx.) | ≈ 1,6 tok/s (màx.) |
| DDR4 2 canals | ≈ 50 GB/s | ≈ 10 tok/s (màx.) | ≈ 2,6 tok/s (màx.) |
| DDR4 8 canals | ≈ 200 GB/s | ≈ 40 tok/s (màx.) | ≈ 10 tok/s (màx.) |
| GPU ≈ 1000 GB/s | ≈ 1000 GB/s | ≈ 200 tok/s (màx.) | ≈ 50 tok/s (màx.) |

Són **límits superiors teòrics**; a la pràctica s'obté menys. Referència d'usabilitat: una persona llegeix a uns **5–10 tokens/s**, de manera que per a un xat interactiu cal superar aquest llindar.

**Altres factors en CPU:**

- **Instruccions vectorials** (AVX2, AVX-512): llama.cpp les aprofita i milloren molt el rendiment del *prefill*.
- **Nombre de fils**: més nuclis ajuden al *prefill*, però la generació se satura aviat per l'ample de banda. Més fils no sempre és més ràpid.
- **NUMA**: en servidors amb diversos sòcols, que el model estigui a la memòria d'un sòcol i el càlcul a un altre penalitza.

### Mètriques de rendiment

| Mètrica | Què mesura | Fase que la determina |
|---|---|---|
| **TTFT** (temps fins al primer token) | Quant triga a començar a respondre | *Prefill* (càlcul) |
| **Tokens/s de generació** | Velocitat d'escriptura de la resposta | *Decode* (ample de banda) |
| **Tokens/s de processament del prompt** | Velocitat llegint l'entrada | *Prefill* (càlcul) |
| **Concurrència / *throughput*** | Tokens/s totals servint diversos usuaris | Motor + memòria |

### Inventari del nostre maquinari

Comandes útils per caracteritzar un node del clúster:

```bash
lscpu                                   # model de CPU, nuclis, fils, sòcols, NUMA
lscpu | grep -o 'avx[^ ]*' | sort -u    # instruccions vectorials disponibles
free -h                                 # memòria total i disponible
sudo dmidecode -t memory | grep -E 'Size|Speed|Locator' | grep -v 'No Module'
                                        # mòduls de RAM, velocitat i ranures ocupades
numactl --hardware                      # topologia NUMA (si està instal·lat)
nvidia-smi                              # (fase GPU) model de GPU, VRAM, ús
```

!!! note
    Nosaltres tenim accés a la interfície gràfica de Proxmox i molts d'aquests valors es poden trobar allà.

## Quantització i formats

### Què és quantitzar

Els models s'entrenen normalment amb pesos de **16 bits** (FP16 o BF16). **Quantitzar** és representar aquests pesos amb **menys bits** (8, 6, 5, 4, 3…), de manera semblant a com es comprimeix una imatge amb pèrdua.

| | Avantatge | Inconvenient |
|---|---|---|
| Menys bits | Menys memòria, **més velocitat** (menys dades a moure) | Pèrdua de qualitat |
| Més bits | Més fidelitat al model original | Més memòria i més lent |

**Regles pràctiques:**

- **8 bits**: pèrdua de qualitat pràcticament inapreciable.
- **5–6 bits**: pèrdua molt petita. Bon compromís si hi ha memòria.
- **4 bits (Q4_K_M)**: el **punt dolç** més utilitzat. Pèrdua petita i gran estalvi.
- **3 bits o menys**: degradació notable, sobretot en models petits.
- Un **model gran quantitzat a 4 bits** sol ser millor que un **model petit a 16 bits** amb la mateixa memòria.
- Els **models petits** (1–3B) pateixen més la quantització agressiva que els grans.

### Formats de fitxer

| Format | Ús principal | Motors | Notes |
|---|---|---|---|
| **safetensors** | Format original dels models a Hugging Face (normalment FP16/BF16) | Transformers, vLLM | Format segur (no executa codi en carregar-se, a diferència dels antics `.bin`/pickle) |
| **GGUF** | Models quantitzats per a CPU i GPU, un sol fitxer autocontingut | llama.cpp, Ollama, LM Studio | **El que farem servir a la fase CPU** |
| **GPTQ / AWQ** | Quantització a 4 bits optimitzada per a GPU | vLLM, Transformers | Rellevants a la fase GPU |

Un fitxer **GGUF** conté els pesos quantitzats i també les metadades: arquitectura, plantilla de xat, vocabulari… Per això és tan pràctic: un sol fitxer i a funcionar.

### Interpretar els noms dels fitxers GGUF

Exemple de nom de fitxer: `NomModel-8B-Instruct-Q4_K_M.gguf`

| Part | Significat |
|---|---|
| `8B` | 8.000 milions de paràmetres |
| `Instruct` | Versió ajustada per seguir instruccions |
| `Q4` | Quantització a ~4 bits |
| `_K` | Mètode *k-quant*: quantitza per blocs amb escales, més precís que els mètodes antics |
| `_M` | Variant *medium*: algunes capes importants es guarden amb més precisió (`_S` = small, `_L` = large) |

**Taula de referència de quantitzacions GGUF habituals:**

| Tipus | Bits/pes aprox. | Model 8B | Qualitat | Recomanació |
|---|---|---|---|---|
| F16 | 16 | ≈ 16 GB | Original | Referència, poc pràctic en CPU |
| Q8_0 | ≈ 8,5 | ≈ 8,5 GB | Pràcticament idèntica | Si sobra memòria |
| Q6_K | ≈ 6,6 | ≈ 6,6 GB | Molt alta | Bon equilibri |
| Q5_K_M | ≈ 5,7 | ≈ 5,7 GB | Alta | Bon equilibri |
| **Q4_K_M** | ≈ 4,8 | ≈ 4,9 GB | Bona | **Opció per defecte** |
| Q3_K_M | ≈ 3,9 | ≈ 4 GB | Acceptable / irregular | Només si no hi cap res més |
| Q2_K | ≈ 3 | ≈ 3,2 GB | Baixa | No recomanat |

També trobareu les quantitzacions **IQ** (p. ex. `IQ4_XS`), que fan servir tècniques més sofisticades per obtenir millor qualitat amb pocs bits, a canvi de ser una mica més lentes en alguns maquinaris.

A **Ollama**, la quantització apareix a l'etiqueta del model, per exemple `model:8b-instruct-q4_K_M`. Si no se n'especifica cap, Ollama acostuma a descarregar una variant a 4 bits.

### Com triar la quantització: procediment

1. Calculeu la **memòria disponible** per al model (memòria total − SO − altres serveis − KV cache necessària).
2. Trieu el **model més gran** que hi càpiga en **Q4_K_M**.
3. Si sobra memòria, pugeu a **Q5_K_M / Q6_K** abans que a un model més gran a Q3.
4. Comproveu que la **velocitat estimada** (bloc 3.4) sigui suficient per al cas d'ús.
5. **Valideu amb proves reals**: el càlcul és una estimació; la decisió final es pren mesurant (es treballa a les activitats finals del projecte).

## Per saber-ne més

- [**The Open Source AI Definition**](https://opensource.org/ai/open-source-ai-definition){ target="_blank" rel="noopener" } · *Open Source Initiative*  
  Definició oficial de què ha de complir un model per considerar-se de codi obert, més enllà de publicar els pesos.
- [**Model Cards**](https://huggingface.co/docs/hub/model-cards){ target="_blank" rel="noopener" } · *Hugging Face*  
  Què conté una model card i com s'hi indiquen la llicència, el model base, els idiomes i les limitacions.
- [**LLM Inference Performance Engineering: Best Practices**](https://www.databricks.com/blog/llm-inference-performance-engineering-best-practices){ target="_blank" rel="noopener" } · *Databricks*  
  Fases de prefill i decode, mètriques com el TTFT i per què l'ample de banda de memòria limita la generació.
- [**A Visual Guide to Quantization**](https://newsletter.maartengrootendorst.com/p/a-visual-guide-to-quantization){ target="_blank" rel="noopener" } · *Maarten Grootendorst*  
  Explicació molt visual de la quantització: tipus de dades, mètodes i compromís entre mida i qualitat.
- [**GGUF**](https://huggingface.co/docs/hub/gguf){ target="_blank" rel="noopener" } · *Hugging Face*  
  Descripció del format GGUF i taula amb tots els tipus de quantització (Q4_K_M, Q8_0, IQ…).
- [**llama.cpp**](https://github.com/ggml-org/llama.cpp){ target="_blank" rel="noopener" } · *GitHub · ggml-org*  
  Repositori del motor d'inferència en C/C++ que fan servir Ollama i LM Studio, amb instruccions de compilació i eines com llama-bench.
