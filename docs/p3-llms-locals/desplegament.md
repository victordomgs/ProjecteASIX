# Desplegament de LLMs

<p class="lead">Quins programes executen un model, com se'n controla el comportament i com s'hi connecten altres aplicacions. Tot plegat aplicat al servidor GPU del projecte.</p>

| | |
|---|---|
| **Servidor de referència** | Ubuntu amb una NVIDIA RTX 3060 de 12 GB |
| **Continguts** | [Motors d'inferència](#motors-dinferencia) · [Paràmetres d'inferència](#parametres-dinferencia) · [Integració via API](#integracio-via-api) |

---

## El servidor GPU

Abans de triar cap eina cal saber què tenim. Apliquem el que hem vist a [Maquinari i dimensionament](teoria.md#maquinari-i-dimensionament):

| Característica | RTX 3060 (12 GB) |
|---|---|
| Arquitectura | NVIDIA Ampere |
| Memòria (VRAM) | 12 GB GDDR6 |
| Ample de banda | ≈ 360 GB/s |
| Comparació | ≈ 10 vegades l'ample de banda d'un sòcol dels servidors del clúster |

Comprovacions bàsiques al servidor:

```bash
nvidia-smi                       # model de GPU, versió del driver i de CUDA, VRAM ocupada
watch -n 1 nvidia-smi            # ús de la GPU i de la VRAM en temps real
```

!!! warning "Sense driver no hi ha GPU"
    Si `nvidia-smi` no respon, cap motor podrà fer servir la GPU i tot s'executarà a la CPU sense cap avís evident. A Ubuntu, els drivers s'instal·len amb `sudo ubuntu-drivers install` i cal reiniciar.

### Què hi cap a 12 GB

Dels 12 GB, cal descomptar uns 0,5–1 GB que reserven el driver i CUDA. La resta s'ha de repartir entre els pesos del model i la KV cache.

| Model | Mida | Hi cap? | Velocitat màxima teòrica |
|---|---|---|---|
| 3B Q4_K_M | ≈ 2 GB | Sí, amb molt de context | ≈ 180 tok/s |
| 8B Q4_K_M | ≈ 4,9 GB | Sí, amb context ampli | ≈ 73 tok/s |
| 8B Q8_0 | ≈ 8,5 GB | Sí, amb context moderat | ≈ 42 tok/s |
| 14B Q4_K_M | ≈ 9 GB | Just, amb context de 4–8K | ≈ 40 tok/s |
| 32B Q4_K_M | ≈ 19 GB | **No** | Vegeu més avall |

### Què passa si el model no hi cap: el *offloading*

Si un model no cap a la VRAM, la majoria de motors en reparteixen les **capes** entre la GPU i la CPU. El model funciona, però cada token ha de passar per totes dues parts, i **la part lenta mana**:

```text
Temps per token ≈ GB a la GPU / ample de banda GPU  +  GB a la RAM / ample de banda RAM
```

Exemple amb un 32B Q4 (19 GB): 11 GB a la GPU i 8 GB a una RAM de 40 GB/s:

```text
11 / 360 + 8 / 40 = 0,03 + 0,20 = 0,23 s per token  →  ≈ 4 tok/s
```

Amb només el 40 % del model fora de la GPU, la velocitat cau de desenes de tokens per segon a uns 4.

!!! tip "Idea clau!"
    En una GPU, l'objectiu és que **tot el model i la KV cache càpiguen a la VRAM**. Un model una mica més petit, però que hi capigui sencer, gairebé sempre és millor opció que un de més gran repartit.

---

## Motors d'inferència

Un **motor d'inferència** és el programa que fa funcionar el model:

1. Carrega els pesos a la memòria (VRAM o RAM).
2. Aplica la **plantilla de xat** del model als missatges.
3. Gestiona la **KV cache** i el context.
4. Genera els tokens segons els **paràmetres** que li indiquem.
5. Exposa el model per **línia d'ordres, web o API**.

### Ollama

**Ollama** és l'opció més senzilla per posar un model en marxa. Per sota fa servir el mateix motor que llama.cpp, i hi afegeix la gestió de models, un servei del sistema i una API.

**Instal·lació a Ubuntu:**

```bash
curl -fsSL https://ollama.com/install.sh | sh
systemctl status ollama          # s'instal·la com a servei systemd
```

**Ordres bàsiques:**

```bash
ollama pull qwen3:8b             # descarrega un model de la biblioteca
ollama run qwen3:8b              # xat interactiu (descarrega el model si cal)
ollama list                      # models descarregats
ollama ps                        # models carregats, memòria i % a GPU/CPU
ollama rm qwen3:8b               # esborra un model
```

!!! info "Noms de models"
    Els models de l'exemple són orientatius. Consulteu la [biblioteca d'Ollama](https://ollama.com/library){ target="_blank" rel="noopener" } per veure els models i les etiquetes disponibles en cada moment. L'etiqueta indica la mida i la quantització, per exemple `:8b-q4_K_M`.

**Dades del servei que un administrador ha de conèixer:**

| Element | Valor per defecte |
|---|---|
| Servei | `ollama.service` |
| Adreça i port | `127.0.0.1:11434` (només accessible des del mateix servidor) |
| Models | `/usr/share/ollama/.ollama/models` |
| Logs | `journalctl -u ollama` |
| Configuració | Variables d'entorn: `sudo systemctl edit ollama` |

Exemple de configuració del servei amb `sudo systemctl edit ollama`:

```ini
[Service]
Environment="OLLAMA_CONTEXT_LENGTH=8192"
```

```bash
sudo systemctl daemon-reload
sudo systemctl restart ollama
```

!!! warning "Context per defecte"
    Amb menys de 24 GB de VRAM, Ollama fa servir per defecte un context de **4K tokens**. Si li passeu un document o uns logs més llargs, **el model no veu el principi del text** i no dona cap error. Augmenteu-lo amb `OLLAMA_CONTEXT_LENGTH` o amb el paràmetre `num_ctx`, tenint en compte la memòria que consumeix.

**Modelfile.** Un *Modelfile* permet crear una variant d'un model amb un *system prompt* i uns paràmetres fixos. És l'equivalent d'un fitxer de configuració per a un servei:

```dockerfile title="Modelfile"
FROM qwen3:8b

PARAMETER temperature 0.2
PARAMETER num_ctx 8192

SYSTEM """
Ets un assistent d'administració de sistemes Linux.
Respons en català, de forma concisa, i sempre indiques les ordres en blocs de codi.
"""
```

```bash
ollama create asix-assistent -f Modelfile
ollama run asix-assistent
```

### llama.cpp (`llama-server`)

**llama.cpp** és el motor en C/C++ que hi ha sota Ollama, LM Studio i moltes altres eines. Fer-lo servir directament dona **més control** (cada paràmetre és una opció de la línia d'ordres), a canvi de més feina: cal compilar-lo i gestionar els fitxers GGUF a mà.

**Compilació amb suport per a GPU NVIDIA** (cal tenir instal·lat el CUDA Toolkit):

```bash
git clone https://github.com/ggml-org/llama.cpp
cd llama.cpp
cmake -B build -DGGML_CUDA=ON
cmake --build build --config Release -j
```

**Servidor:**

```bash
./build/bin/llama-server \
  -m /opt/models/model-8B-Instruct-Q4_K_M.gguf \
  -ngl 99 \
  -c 8192 \
  --host 127.0.0.1 --port 8080
```

| Opció | Significat |
|---|---|
| `-m` | Fitxer GGUF del model |
| `-ngl` | Capes que es carreguen a la GPU (`99` = totes). Amb un valor més baix es fa *offloading* |
| `-c` | Mida del context en tokens |
| `-np` | Peticions que es poden atendre en paral·lel (comparteixen el context) |
| `--host` / `--port` | Adreça i port d'escolta |
| `--api-key` | Clau que cal enviar a cada petició |

A `http://127.0.0.1:8080` hi ha una **interfície web de xat** integrada, i l'API és compatible amb OpenAI.

### LM Studio

**LM Studio** és una aplicació d'escriptori (Windows, macOS i Linux) per descarregar i provar models amb una interfície gràfica. També fa servir llama.cpp per sota i pot exposar una API local.

És molt útil per **fer proves al vostre ordinador**, però no és l'eina habitual en un servidor: està pensada sobretot per a ús interactiu.

### vLLM

**vLLM** és un motor pensat per **servir models a molts usuaris alhora** des de GPU. Les seves dues tècniques principals són:

- ***PagedAttention***: gestiona la KV cache per blocs, com la memòria virtual d'un sistema operatiu, i n'evita el malbaratament.
- ***Continuous batching***: agrupa les peticions de diferents usuaris a cada pas de generació. Com que els pesos es llegeixen una vegada per a tot el lot, el rendiment total augmenta molt.

```bash
vllm serve Qwen/Qwen3-8B-AWQ --max-model-len 8192 --gpu-memory-utilization 0.90
# API compatible amb OpenAI a http://localhost:8000/v1
```

Limitacions al nostre servidor:

- Treballa amb models en **safetensors** (FP16/BF16) o quantitzats per a GPU (**AWQ, GPTQ**), no amb GGUF de manera habitual.
- Per defecte **reserva el 90 % de la VRAM** en arrencar, encara que el model sigui petit, per tenir-hi la KV cache.
- Amb 12 GB, en la pràctica queda limitat a models de **≈ 8B quantitzats a 4 bits** o models petits en FP16.

### Comparativa

| | Ollama | llama.cpp | LM Studio | vLLM |
|---|---|---|---|---|
| **Pensat per a** | Posar models en marxa ràpidament | Control fi i entorns diversos | Proves a l'escriptori | Servei multiusuari en GPU |
| **Interfície** | CLI + API | CLI + API + web | Gràfica + API | API |
| **Formats** | GGUF (biblioteca pròpia) | GGUF | GGUF (i MLX a Mac) | safetensors, AWQ, GPTQ |
| **CPU sense GPU** | Sí | Sí | Sí | No (en la pràctica) |
| **Concurrència** | Bàsica | Moderada (`-np`) | Bàsica | **Alta** |
| **Corba d'aprenentatge** | Baixa | Mitjana | Molt baixa | Alta |
| **Al P3 el farem servir per a…** | Desplegament principal | Proves i comparatives | Proves en local | Comparativa de concurrència |

---

## Paràmetres d'inferència

### Com tria el model el següent token

A cada pas, el model no "escriu" una paraula: calcula una **probabilitat per a cada token** del vocabulari. Els paràmetres d'inferència decideixen **com es tria** el token següent a partir d'aquestes probabilitats.

```text
Context: "El servei no arrenca. Reviseu els ..."
         logs     → 62 %
         fitxers  → 18 %
         permisos → 11 %
         ...
```

- Si sempre es tria el més probable, la resposta és **predictible però repetitiva**.
- Si es tria a l'atzar segons les probabilitats, la resposta és **més variada però pot desviar-se**.

### Paràmetres principals

| Paràmetre (Ollama) | Equivalent OpenAI / llama.cpp | Què fa | Per defecte a Ollama |
|---|---|---|---|
| `temperature` | `temperature` / `--temp` | Aplana (valor alt) o concentra (valor baix) les probabilitats. `0` ≈ sempre el token més probable | 0,8 |
| `top_k` | — / `--top-k` | Només considera els *k* tokens més probables | 40 |
| `top_p` | `top_p` / `--top-p` | Només considera els tokens que sumen fins a una probabilitat *p* | 0,9 |
| `min_p` | — / `--min-p` | Descarta els tokens molt menys probables que el millor | 0,0 |
| `repeat_penalty` | — / `--repeat-penalty` | Penalitza els tokens que ja han aparegut | 1,0 (desactivat) |
| `num_predict` | `max_tokens` / `-n` | Màxim de tokens que es generen | −1 (sense límit) |
| `num_ctx` | — / `-c` | Mida del context | Segons la VRAM |
| `seed` | `seed` / `--seed` | Llavor aleatòria: amb la mateixa llavor i el mateix prompt, la mateixa resposta | — |
| `stop` | `stop` / — | Seqüències que aturen la generació | Cap |

!!! info "El context també és un paràmetre de memòria"
    `num_ctx` no només determina quant text veu el model: també fixa la mida de la **KV cache** que es reserva. Duplicar el context duplica aquesta memòria (vegeu [Quanta memòria necessita un model](teoria.md#quanta-memoria-necessita-un-model)).

### Valors recomanats segons la tasca

| Tasca | `temperature` | Altres |
|---|---|---|
| Generar scripts, ordres o configuracions | 0–0,2 | `seed` fixa si cal reproduir el resultat |
| Extreure dades en JSON o classificar logs | 0 | `num_predict` limitat i `stop` si cal |
| Resumir documentació o incidències | 0,3–0,5 | — |
| Xat d'ús general | 0,7 | Valors per defecte |
| Pluja d'idees, redacció creativa | 0,9–1,1 | `repeat_penalty` ≈ 1,1 |

!!! warning "Temperatura 0 no vol dir resposta correcta"
    Una temperatura baixa fa el model **més previsible**, però no més encertat: si el model no sap una cosa, la inventarà igual, només que sempre de la mateixa manera. A les tasques d'administració de sistemes, **cal revisar sempre** les ordres abans d'executar-les.

### On es configuren

=== "Ollama (sessió)"

    ```text
    ollama run qwen3:8b
    >>> /set parameter temperature 0.2
    >>> /set parameter num_ctx 8192
    ```

=== "Ollama (Modelfile)"

    ```dockerfile
    FROM qwen3:8b
    PARAMETER temperature 0.2
    PARAMETER num_ctx 8192
    ```

=== "llama-server"

    ```bash
    llama-server -m model.gguf -c 8192 --temp 0.2 --top-p 0.9
    ```

=== "API"

    Es poden enviar a cada petició (vegeu l'apartat següent). Els valors de la petició tenen prioritat sobre els del Modelfile.

### Plantilla de xat i *system prompt*

Un model *instruct* s'ha entrenat amb un **format de conversa concret**, amb marques especials que separen el missatge del sistema, el de l'usuari i el de l'assistent. Aquest format és la **plantilla de xat**.

- Els fitxers GGUF solen incloure la plantilla, i Ollama i llama.cpp l'apliquen automàticament.
- Si la plantilla és incorrecta (un GGUF mal convertit, una plantilla escrita a mà amb errors), el model respon de manera estranya, repeteix el text de l'usuari o no s'atura mai.
- El ***system prompt*** (vist al P1) s'insereix dins d'aquesta plantilla. En un servei, és la manera de fixar el rol, l'idioma i les restriccions per a tots els usuaris.

---

## Integració via API

Un motor d'inferència és útil de debò quan **altres programes** s'hi poden connectar: scripts de Bash, aplicacions en Python, agents com els del P2 o interfícies web. Tots ho fan mitjançant una **API HTTP**.

### API nativa d'Ollama

| Endpoint | Ús |
|---|---|
| `POST /api/generate` | Genera text a partir d'un prompt |
| `POST /api/chat` | Conversa amb una llista de missatges |
| `POST /api/embed` | Calcula *embeddings* (el farem servir al P4) |
| `GET /api/tags` | Llista els models instal·lats |
| `GET /api/ps` | Models carregats a memòria |

```bash
curl http://localhost:11434/api/chat -d '{
  "model": "qwen3:8b",
  "messages": [
    {"role": "system", "content": "Ets un administrador de sistemes Linux. Respon en català."},
    {"role": "user", "content": "Com veig quins ports estan escoltant?"}
  ],
  "stream": false,
  "options": {"temperature": 0.2, "num_ctx": 8192}
}'
```

### API compatible amb OpenAI

La majoria de motors ofereixen també una API amb el **mateix format que la d'OpenAI**. S'ha convertit en l'estàndard de facto del sector:

| Motor | URL base |
|---|---|
| Ollama | `http://localhost:11434/v1` |
| llama-server | `http://localhost:8080/v1` |
| vLLM | `http://localhost:8000/v1` |
| Proveïdors al núvol | La URL del proveïdor |

```bash
curl http://localhost:11434/v1/chat/completions \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer ollama" \
  -d '{
    "model": "qwen3:8b",
    "messages": [{"role": "user", "content": "Explica què fa: df -h"}],
    "temperature": 0.2
  }'
```

!!! tip "Per què és tan important?"
    Una aplicació escrita per a l'API d'OpenAI funciona amb **qualsevol** d'aquests motors **canviant només la URL base i el nom del model**. És el mateix mecanisme que fa servir el protocol `openai-completions` d'OpenClaw al P2: podeu connectar-hi un model local en lloc d'un del núvol.

!!! info "Clau d'API i context amb Ollama"
    En local, Ollama **ignora la clau d'API**, però moltes llibreries n'exigeixen una: n'hi ha prou amb un valor qualsevol, com `ollama`. A més, l'API compatible amb OpenAI **no permet indicar `num_ctx`** a cada petició: cal fixar-lo amb `OLLAMA_CONTEXT_LENGTH` o crear un model amb un Modelfile.

### Des de Python

Amb la llibreria oficial `openai`, només cal canviar la URL base:

```python
from openai import OpenAI

client = OpenAI(base_url="http://localhost:11434/v1", api_key="ollama")

resposta = client.chat.completions.create(
    model="qwen3:8b",
    messages=[
        {"role": "system", "content": "Ets un administrador de sistemes. Respon en català."},
        {"role": "user", "content": "Quina diferència hi ha entre systemctl stop i disable?"},
    ],
    temperature=0.2,
)
print(resposta.choices[0].message.content)
```

### Des d'un script de Bash

Exemple: resumir els errors del sistema de l'última hora. `jq` construeix el JSON de manera segura (escapa les cometes i els salts de línia dels logs) i extreu la resposta:

```bash title="resum-errors.sh"
#!/usr/bin/env bash
LOGS=$(journalctl -p err --since "1 hour ago" --no-pager | tail -n 200)

jq -n --arg logs "$LOGS" '{
  model: "qwen3:8b",
  stream: false,
  options: {temperature: 0},
  messages: [
    {role: "system", content: "Resumeix els errors en català, agrupats per servei, en una llista breu."},
    {role: "user", content: $logs}
  ]
}' | curl -s http://localhost:11434/api/chat -d @- | jq -r '.message.content'
```

!!! danger "Dades sensibles"
    Aquest script envia logs del sistema al model. Amb un model **local**, les dades no surten del servidor. Si canvieu la URL per la d'un proveïdor al núvol, els logs (amb usuaris, IP i rutes) **sortiran de l'organització**. És un dels motius principals per desplegar LLMs en local.

### *Streaming*

Per defecte, l'API **envia la resposta a trossos a mesura que es genera** (*streaming*). Així l'usuari comença a llegir de seguida, en lloc d'esperar la resposta sencera.

- **API nativa d'Ollama**: una línia JSON per fragment (NDJSON). Es desactiva amb `"stream": false`.
- **API compatible amb OpenAI**: *Server-Sent Events* (`data: {...}`), només si s'indica `"stream": true`.

En scripts que processen la resposta sencera (com l'exemple de Bash), és més còmode desactivar-lo. En una interfície de xat, cal activar-lo.

### Seguretat de l'API

Per defecte, Ollama i llama-server **només escolten a `127.0.0.1`**: només s'hi pot connectar el mateix servidor. Si els feu escoltar a totes les interfícies (`OLLAMA_HOST=0.0.0.0`, `--host 0.0.0.0`), **qualsevol equip de la xarxa podrà fer servir el model sense autenticació**, consumir-ne la GPU i, amb Ollama, fins i tot descarregar o esborrar models.

!!! warning
    No exposeu mai el port d'un motor d'inferència directament a la xarxa. A la Pràctica III el publicarem darrere d'un ***reverse proxy*** amb TLS i autenticació.

---

## Per saber-ne més

- [**Documentació d'Ollama**](https://docs.ollama.com){ target="_blank" rel="noopener" } · *Ollama*  
  Instal·lació a Linux, Modelfile, API nativa i compatibilitat amb OpenAI.
- [**llama.cpp**](https://github.com/ggml-org/llama.cpp){ target="_blank" rel="noopener" } · *GitHub · ggml-org*  
  Compilació amb CUDA i documentació de `llama-server` i de les seves opcions.
- [**Documentació de vLLM**](https://docs.vllm.ai){ target="_blank" rel="noopener" } · *vLLM*  
  Servei d'alt rendiment en GPU, PagedAttention i servidor compatible amb OpenAI.
- [**Text generation strategies**](https://huggingface.co/docs/transformers/generation_strategies){ target="_blank" rel="noopener" } · *Hugging Face*  
  Com funcionen els mètodes de selecció de tokens i els paràmetres de mostreig.
