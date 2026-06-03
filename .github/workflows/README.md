# Reusable workflows — Grupo Mabu

Workflows centralizados por **arquétipo**. Cada repo chama o que casa com o que
ele entrega, em ~3 linhas. Mantidos aqui; mudou aqui, vale pra todos.

## Pré-requisito (uma vez, no nível da org)

Settings → Actions → General → **Allow access to workflows from repositories owned by `MabuThermas`**
(necessário para repos privados consumirem estes workflows).

## Arquétipos disponíveis

| Workflow | Para | Entregável |
|----------|------|-----------|
| `node-api.yml` | APIs/serviços Node (JS/TS) | imagem container (GHCR) |
| `laravel.yml` | apps Laravel/PHP | imagem container (GHCR) |
| `tauri-desktop.yml` | app desktop Tauri (Windows) | instalador + release (auto-update) |
| `_security.yml` | **compartilhado** (Gitleaks + Trivy + Semgrep) | — (chamado pelos outros) |
| `mobile.yml` | _(pendente — definir framework)_ | _(APK/IPA / store)_ |

## Como consumir

**API Node:**
```yaml
# .github/workflows/ci.yml no repo consumidor
name: CI
on: { push: { branches: [main] }, pull_request: }
jobs:
  ci:
    uses: MabuThermas/.github/.github/workflows/node-api.yml@main
    with: { node-version: "20", publish-image: true }
    secrets: inherit
```

**Laravel:**
```yaml
jobs:
  ci:
    uses: MabuThermas/.github/.github/workflows/laravel.yml@main
    with: { php-version: "8.4", publish-image: true }
    secrets: inherit
```

**Tauri desktop (dispara em tag):**
```yaml
on: { push: { tags: ["v*"] } }
jobs:
  release:
    uses: MabuThermas/.github/.github/workflows/tauri-desktop.yml@main
    secrets: inherit
```

## Notas

- **Segurança** (`_security.yml`) é chamada por todos: Gitleaks (secret scan), Trivy
  (deps, gate high/critical), Semgrep (SAST, gate ERROR). Free/OSS.
- **`@main`** é prático mas mutável. Para produção, considere pinar por **tag**
  (`@v1`) e versionar estes workflows.
- **Deploy** é um workflow à parte (self-hosted runner on-prem) — estes arquétipos
  vão até **publicar o artefato** (imagem/instalador).
- **Tauri** builda em `windows-latest` (hospedado = 2x minutos); migrar p/ self-hosted
  Windows remove esse custo.
