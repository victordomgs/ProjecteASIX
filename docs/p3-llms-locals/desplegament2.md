# Desplegament de LLMs (CPU)

## El nostre servidor

A l'activitat de dimensionament heu inventariat aquest maquinari:

| Dada | Valor |
|---|---|
| CPU | 2 × AMD Opteron 6370P |
| Instruccions vectorials | SSE4.2, **AVX**, FMA · **sense AVX2** |
| Memòria | 64 GB DDR3-1600 |
| Hipervisor | Proxmox VE |
| Memòria disponible | ≈ 29 GB (hi ha una VM Windows Server engegada) |

Dues coses a tenir presents:

- **Sense AVX2** el model funcionarà, però més lent que en una CPU moderna.
- **La RAM és lenta (DDR3)**. Com vau calcular, el que limita la velocitat és l'ample de banda de memòria. Per a un xat fluid, cal un model petit: **1B o 3B en Q4_K_M**.

!!! note "Arquitectura Opteron"
    Aquests Opteron tenen cada sòcol dividit en 2 nodes NUMA (4 en total, de 16 GB cadascun). Per això, a la pràctica, la velocitat pot quedar per sota de les vostres estimacions. Ho comprovareu mesurant.

---

## Què necessitem

Recordeu el bloc 1: per fer funcionar un LLM local cal:

1. **Un entorn** on instal·lar-lo → un contenidor LXC a Proxmox.
2. **Un motor d'inferència** → **Ollama**.
3. **Un model** → un model petit *instruct* en GGUF Q4_K_M.
4. **Una interfície** → el terminal (`ollama run`).

Fem servir **Ollama** perquè és la manera més senzilla: s'instal·la amb una comanda, descarrega els models per nosaltres i ja porta un xat per terminal.

---

## Crear el contenidor LXC a Proxmox

No instal·lem res directament a l'hipervisor: creem un **contenidor LXC**. És lleuger i veu les instruccions reals de la CPU (AVX, FMA).

Des de la interfície web de Proxmox, **Create CT**:

| Opció | Valor recomanat |
|---|---|
| Plantilla | Debian 12 o Ubuntu 24.04 |
| Disc | ≥ 20 GB (els models ocupen espai) |
| Cores | 8 |
| Memory | 12288 MB (12 GB) |
| Swap | **0** |
| Xarxa | DHCP o IP fixa de la xarxa del laboratori |

!!! warning "Swap a 0"
    Si el model acaba a la swap, la velocitat cau tant que el xat queda inservible. És preferible que falli i ajustar el model.

Engegueu el contenidor i obriu-ne la consola (o entreu-hi per SSH).

### Comprovar que el contenidor veu AVX

```bash
grep -o -w -E 'avx|avx2|fma' /proc/cpuinfo | sort -u
```

Ha de mostrar `avx` i `fma` (i **no** `avx2`). Si no surt `avx`, aviseu el professor.

---

## Instal·lar Ollama

Dins del contenidor:

```bash
apt update && apt install -y curl zstd
curl -fsSL https://ollama.com/install.sh | sh
```

L'script instal·la Ollama i el deixa funcionant com a **servei** (`systemd`):

```bash
systemctl status ollama     # ha d'estar "active (running)"
ollama --version
```

> L'avís *"No NVIDIA/AMD GPU detected. Ollama will run in CPU-only mode."* és normal: no tenim GPU.

---

## Descarregar un model

Busqueu un model petit a la biblioteca d'Ollama (ollama.com/library). Fixeu-vos que sigui:

- **Instruct / chat** (no *base*).
- **1B–3B** de paràmetres.
- Quantitzat a **4 bits** (l'opció per defecte d'Ollama).

```bash
ollama pull <model>:<etiqueta>     # p. ex. el model petit que vau triar a l'activitat de dimensionament
ollama list                        # models descarregats i mida
```

---

## Conversar amb el model

```bash
ollama run <model>:<etiqueta>
```

Apareix un indicador `>>>` on podeu escriure:

```
>>> Explica en català, en tres línies, què és Proxmox.
```

Ordres útils dins del xat:

| Ordre | Què fa |
|---|---|
| `/set system "Ets un assistent d'administració de sistemes. Respon sempre en català."` | Defineix el *system prompt* |
| `/set parameter num_ctx 4096` | Mida del context |
| `/clear` | Esborra l'historial de la conversa |
| `/show info` | Mostra la informació del model (paràmetres, quantització, context) |
| `/bye` | Surt del xat |

Per fer una pregunta sense entrar al mode interactiu:

```bash
ollama run <model>:<etiqueta> "Escriu una comanda per veure l'ús de disc de /var"
```

---

## Mesurar la velocitat

Amb l'opció `--verbose`, Ollama mostra estadístiques després de cada resposta:

```bash
ollama run <model>:<etiqueta> --verbose
```

```
prompt eval rate:     xx tokens/s     ← processament del prompt (prefill)
eval rate:            xx tokens/s     ← velocitat de generació (decode)
```

- **`eval rate`** és la que compareu amb la vostra estimació (`tok/s ≈ GB/s ÷ GB del model`).
- Recordeu el llindar del bloc 3: per a un xat còmode cal **≥ 5 tokens/s**.

Mentre el model respon, en una altra consola:

```bash
htop        # ús de CPU
free -h     # memòria ocupada pel model
ollama ps   # model carregat, mida a memòria i si s'executa en CPU
```

---

## Per saber-ne més

- [**Documentació d'Ollama**](https://docs.ollama.com){ target="_blank" rel="noopener" } · *Ollama*  
  Instal·lació a Linux, Modelfile, API nativa i compatibilitat amb OpenAI.
