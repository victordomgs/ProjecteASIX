# Aplicació i desplegament d'IA en sistemes · 2n ASIX

Material del bloc d'IA (49 h) del mòdul de Projecte (M0379) de 2n d'ASIX.

🌐 Web: https://victordomgs.github.io/ProjecteASIX/

## Estructura

- `docs/` — contingut del web (un directori per projecte: `index`, `teoria`, `activitats`, `avaluacio`)
- `projectes/` — codi de suport (docker-compose, scripts, dades d'exemple)
- `mkdocs.yml` — configuració i menú del web

## Treballar en local

```bash
python -m venv .venv && source .venv/bin/activate
pip install -r requirements.txt
mkdocs serve        # http://127.0.0.1:8000
```

## Publicació

Cada `push` a `main` publica el web amb GitHub Actions.
Cal activar-ho un cop a **Settings → Pages → Source: GitHub Actions**.

## Llicència

Contingut: [CC BY-SA 4.0](https://creativecommons.org/licenses/by-sa/4.0/deed.ca). Codi de `projectes/`: MIT.
