<h1 align="center">🍊 Datestre</h1>

<p align="center">
  <strong>Plataforma de inteligência sobre relacionamentos</strong><br>
  <sub>Aplicativo Flutter cross-platform com IA integrada para análise contextual de relatos sobre dates</sub>
</p>

<p align="center">
  <img src="https://img.shields.io/badge/status-beta-orange?style=flat-square">
  <img src="https://img.shields.io/badge/plataforma-Android%20%7C%20iOS-blue?style=flat-square">
  <img src="https://img.shields.io/badge/testers-30%2B-brightgreen?style=flat-square">
  <img src="https://img.shields.io/badge/testes-492%20automatizados-success?style=flat-square">
  <img src="https://img.shields.io/badge/c%C3%B3digo-privado-lightgrey?style=flat-square">
</p>

> **Repositório de vitrine.** O código-fonte do Datestre é privado. Este README documenta arquitetura, decisões técnicas e funcionalidades para fins de portfolio.

---

## Sobre

Datestre é uma rede social brasileira pra entender o que rolou no date — antes, durante e depois. Combina relatos editorial-grade com **IA proprietária** que lê os textos e devolve análise contextual: padrões de comportamento, sinais de alerta, leitura de prints de conversa.

Não é app de matchmaking. É **leitor de padrões** — pra usuária ler o que ela mesma viveu, não pra app sentenciar quem é certo ou errado.

## Funcionalidades

- **Mural** — feed editorial com posts da comunidade, reações expressivas (Amei, Brilhou, Red Flag, Livramento, etc) e comentários por categoria
- **Filtro de Dignidade** — IA analisa relato (texto ou print de conversa) e devolve leitura: nível de red flag, padrão clínico detectado, veredito tentativo
- **Aviso Privado** — quando você posta, a IA detecta padrões recorrentes nos seus relatos e te alerta (só você vê)
- **Arquivos** — organizador de momentos por pessoa, com timeline visual, padrão recorrente, alertas configuráveis
- **Modo Acompanhado** — feature de segurança em dates: contato de confiança, code-word, fake call, GPS opcional, ligação de emergência
- **Jornada** — raio-X mensal do comportamento do usuário com conquistas e relatórios editoriais

## Stack

### Mobile
- **Flutter / Dart** — cross-platform Android + iOS, single codebase
- **Provider** + **StreamBuilder** — gerenciamento de estado
- 492 testes automatizados (unit + widget + integration)

### Backend serverless
- **Firebase Authentication** — login com claims customizadas para painel admin
- **Cloud Firestore** — banco NoSQL com 100+ regras de segurança granulares e queries geoespaciais indexadas
- **Cloud Functions (Node.js)** — moderação de imagem (Vision API), proxy seguro pra IA, webhooks de notificação, limpezas agendadas
- **Cloud Storage** — upload de imagens com moderação SafeSearch antes de gravar URL
- **Firebase Cloud Messaging** — push notifications transacionais
- **Remote Config** — feature flags, rollout gradual, experimentação

### IA
- **Anthropic Claude** — Haiku (filtro/insights rápidos) + Sonnet (análises profundas)
- **Cache híbrido** — reduz custo de API em ~60% sem perder personalização
- **Fallback léxico offline** — análise por palavra-chave quando a API está fora, garantindo disponibilidade

### Outros
- **Google Cloud Vision** — moderação de imagem (SafeSearch + OCR opcional)
- **GitHub Actions** — CI/CD
- **Sentry / Crashlytics** — observabilidade

## Arquitetura

```
┌─────────────────────────────────────────────┐
│  Mobile App (Flutter — Android + iOS)       │
│  Provider state · 492 testes · App Check    │
└────────────┬────────────────────────────────┘
             │
┌────────────▼────────────────────────────────┐
│  Firebase Authentication                    │
│  Custom claims (admin) · LGPD opt-in        │
└────────────┬────────────────────────────────┘
             │
   ┌─────────┼─────────────────────────┐
   │         │                         │
┌──▼───┐  ┌──▼─────────┐  ┌────────────▼──┐
│Fire- │  │ Cloud      │  │ Cloud Storage │
│store │  │ Functions  │  │ + Vision API  │
│      │  │ (Node.js)  │  │ (moderação)   │
└──┬───┘  └────┬───────┘  └───────────────┘
   │           │
   │           ▼
   │    ┌──────────────┐
   │    │  Anthropic   │
   │    │  Claude API  │
   │    │ (Haiku+Sonnet│
   │    │  + cache)    │
   │    └──────────────┘
   ▼
100+ regras Firestore granulares · queries
geoespaciais indexadas · soft-delete LGPD
```

## Decisões técnicas que valem destacar

### 1. IA com cache híbrido + fallback léxico

Cada chamada à API da Anthropic custa. Implementei:
- **Cache de respostas** indexado por hash do input semântico (não match exato — match conceitual via embeddings)
- **Fallback offline** — quando a API falha (timeout, quota, erro 5xx), o sistema cai num analisador léxico baseado em dicionário de padrões de comportamento (light, mas funcional)

Resultado: **~60% de cache hit rate** em produção e **disponibilidade efetiva de ~99,5%** mesmo quando a API tem incidente.

### 2. Conformidade LGPD nativa

Não foi feature retroativa — foi arquitetural desde o dia 1.
- **Consentimento explícito por feature** (não checkbox genérico)
- **Soft-delete com retenção configurável** (default 180 dias)
- **Exportação de dados pessoais** sob solicitação do titular (Cloud Function dedicada)
- **Auditoria de acesso** via Firestore Rules + log de queries sensíveis
- **Pseudonimização de confessionais** — autoria preservada anônima mesmo em consultas admin

### 3. Push notifications com dedup transacional

Race condition clássica de notificações: 3 callsites disparam `checkAndAwardBadges` quase simultaneamente, cada um manda push da mesma badge. Solução: **lock per-user em memória** via Map de Futures in-flight, coalescendo execuções paralelas.

### 4. Moderação de imagem em 2 camadas

Cloud Vision SafeSearch antes de gravar URL no Firestore (fail-open com audit log em incidentes). Threshold calibrado pra app adulto 18+: bloqueia explicitamente pornografia (`adult >= VERY_LIKELY`) e gore (`violence >= VERY_LIKELY`), mas deixa passar "racy" (biquíni, decote, cerveja) — fricção sem ganho.

## Status

- ✅ Beta lançado em **maio/2026** com 30+ testers reais
- ✅ Distribuição via Firebase App Distribution (Android)
- ⏳ iOS aguardando Apple Developer Program enrollment
- ⏳ Versão 1.0 estável prevista pra Q3/2026

## Sobre o desenvolvimento

Projeto solo desenvolvido por **Mauricio Floquet** desde out/2024:
- 100% das decisões de arquitetura, código, design e operação
- Workflow profissional: Git/GitHub com versionamento semântico, CI/CD, ambiente isolado de produção
- Datestre é também o **case real** que valida minha capacidade técnica ponta-a-ponta

## Quer saber mais?

- 💼 [LinkedIn — Mauricio Floquet](https://www.linkedin.com/in/mauriciofloquet)
- 📧 [mauriciofloquet23@gmail.com](mailto:mauriciofloquet23@gmail.com)
- 🌐 [datestre.web.app](https://datestre.web.app) — política de privacidade e termos

---

<sub>📌 Este repositório é vitrine pública. Código-fonte privado por motivos de segurança e propriedade intelectual.</sub>
