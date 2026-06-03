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
| `mobile-react-native.yml` | app React Native (bare) | APK/AAB + IPA → interno/loja |
| `mobile-tauri.yml` | app mobile Tauri v2 | APK/AAB + IPA → interno/loja |
| `_security.yml` | **compartilhado** (Gitleaks + Trivy + Semgrep) | — (chamado pelos outros) |

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

## Mobile — o que você precisa prover

Os workflows mobile (`mobile-react-native`, `mobile-tauri`) têm o **Android pronto** e o
**iOS estruturado** (assinatura Apple marcada com `TODO`, porque é específica do ambiente).
Para funcionar fim-a-fim, configurar como **secrets** do repo/org:

**Android (assinar AAB/APK):**
- `ANDROID_KEYSTORE_BASE64` (o `.jks` em base64), `ANDROID_KEYSTORE_PASSWORD`,
  `ANDROID_KEY_ALIAS`, `ANDROID_KEY_PASSWORD`.
- Distribuição: `FIREBASE_APP_ID_ANDROID` + `FIREBASE_SERVICE_ACCOUNT` (interno) **ou**
  `PLAY_SERVICE_ACCOUNT_JSON` (loja).

**iOS (assinar IPA — exige conta Apple Developer):**
- `APPLE_CERTIFICATE_BASE64` + `APPLE_CERTIFICATE_PASSWORD`,
  `APPLE_PROVISIONING_PROFILE_BASE64`, `APP_STORE_CONNECT_API_KEY`.
- iOS roda em **macOS (10x minutos)** — usar com parcimônia.

**Fase sugerida:** Android + distribuição interna primeiro (simples/barato) → depois iOS
→ depois publicação nas lojas. Se o React Native for **Expo** (managed), o caminho muda
para **EAS Build** — avisar para ajustar.
