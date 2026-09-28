# Migración de `clubeconomistas-rd`

Documento de traspaso del repositorio `planeacionln/clubeconomistas-rd`. Acompaña al
archivo **`clubeconomistas-rd.bundle`**, que contiene el repositorio **completo**: los
61 commits y las 7 ramas.

## 1. Qué contiene el bundle

| Rama | Estado | Contenido |
|---|---|---|
| `main` | Producción | Sitio completo (último merge: PR #6) |
| `claude/economists-club-site-redesign-xe0ep1` | Fusionada (PR #5, #6) | — |
| `claude/dominican-economy-stats-wp8p8d` | Fusionada (PR #3, #4) | — |
| `claude/economist-club-premium-redesign-1hypin` | Fusionada (PR #1, #2) | — |
| `claude/actualizar-datos-html-gyp66i` | **Sin fusionar** | Actualización de indicadores en `index.html` |
| `claude/series-robert-joel-cir0u7` | **Sin fusionar** | Tarea de econometría VAR/SVAR (`tareas/econometria-var-svar/`, 36 archivos) |
| `claude/repository-migration-token-25qnwb` | Esta rama | `main` + este documento |

Las dos ramas sin fusionar solo existen en el bundle, fuera de `main`. Si las quieres
conservar, migra todas las ramas (paso 3), no solo `main`.

## 2. Verificar la integridad

```bash
sha256sum clubeconomistas-rd.bundle   # compárala con la de clubeconomistas-rd.bundle.sha256
# git bundle verify debe ejecutarse dentro de un repositorio git (por ejemplo, tras el paso 3a):
git -C clubeconomistas-rd bundle verify ../clubeconomistas-rd.bundle
```

## 3. Restaurar en un repositorio nuevo

```bash
# a) Clonar desde el bundle (queda con main extraída)
git clone clubeconomistas-rd.bundle clubeconomistas-rd
cd clubeconomistas-rd

# b) Traer todas las ramas del bundle como ramas locales
git fetch ../clubeconomistas-rd.bundle 'refs/heads/*:refs/heads/*' --update-head-ok

# c) Apuntar al nuevo destino (GitHub, GitLab, Bitbucket, etc.) y subir todo
git remote set-url origin https://github.com/<NUEVO_DUEÑO>/<NUEVO_REPO>.git
git push origin --all
```

## 4. Lo que NO viaja en el bundle (y cómo tratarlo)

Un bundle transporta el **código y su historial**. No incluye lo siguiente:

1. **Issues, Pull Requests y sus comentarios** (PR #1–#6). Viven en GitHub, no en git.
   Los mensajes de merge del historial sí los referencian.
2. **Base de datos Supabase** — proyecto `hguipwuccxugnnvkpyti`
   (`https://hguipwuccxugnnvkpyti.supabase.co`). `index.html` usa las tablas:
   `aportes`, `auditoria`, `club_jobs`, `club_news_cache`, `club_opiniones`,
   `club_site_content`, `repositorio`. El esquema de `club_opiniones` está en el
   `README.md`. **Si borras el repo, Supabase sigue vivo**: los datos no se pierden,
   pero si también quieres migrarlos, respáldalos aparte (`pg_dump` o el panel de
   Supabase → Database → Backups).
   - La URL y la *anon key* están escritas directamente en `index.html`
     (`SUPABASE_URL` / `SUPABASE_ANON`). Si cambias de proyecto Supabase, esos son los
     dos valores a sustituir.
3. **Despliegue en Vercel.** `vercel.json` (cabeceras de caché para `/data`) y la
   función `api/proxy.js` están en el código, pero el **proyecto de Vercel está
   vinculado a este repositorio de GitHub**. Al migrar: Vercel → Project → Settings →
   Git → *Connect* al nuevo repositorio. Si borras el repo antes, el sitio sigue
   publicado con el último deploy, pero deja de actualizarse.
4. **`deploy-keys-local/`** está en `.gitignore`: esas llaves nunca se subieron y no
   están en el bundle (a propósito).

## 5. Inventario del código

```
index.html                 App completa (React + Recharts vía CDN), ~2.1 MB
api/proxy.js               Proxy serverless (Vercel) con lista blanca de hosts
vercel.json                Cabeceras de caché
data/                      ~30 MB: 57 series CE-SER-2026-XXXX + catálogos
scripts/build_bcrd_catalog.py
reglamento-interno-club-economistas-dominicanos.pdf
README.md                  Documentación técnica del sitio
```

## 6. Orden recomendado antes de eliminar el repositorio original

1. Verificar el bundle (paso 2) y guardarlo en **al menos dos lugares** (p. ej. disco
   local + nube).
2. Restaurarlo en el destino (paso 3) y confirmar que están las 7 ramas:
   `git branch -a`.
3. Respaldar Supabase si aplica (punto 4.2).
4. Reconectar Vercel al nuevo repositorio y confirmar que despliega.
5. Solo entonces: GitHub → `planeacionln/clubeconomistas-rd` → Settings →
   *Danger Zone* → **Delete this repository**. Es irreversible (GitHub permite
   restaurarlo solo durante 90 días, y solo si no era un fork).
