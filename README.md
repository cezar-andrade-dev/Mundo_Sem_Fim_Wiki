# 🌍 YouTube Travel Analytics

Pipeline completo para extrair, processar e visualizar dados de canais de viagem no YouTube usando transcrições de vídeos e análise com LLM.

---

## 📋 Índice

- [Visão Geral](#visão-geral)
- [Funcionalidades](#funcionalidades)
- [Arquitetura](#arquitetura)
- [Pré-requisitos](#pré-requisitos)
- [Instalação](#instalação)
- [Configuração](#configuração)
- [Uso](#uso)
- [Estrutura do Projeto](#estrutura-do-projeto)
- [Schema do Banco de Dados](#schema-do-banco-de-dados)
- [Dashboard](#dashboard)
- [Roadmap](#roadmap)
- [Contribuindo](#contribuindo)

---

## Visão Geral

O **YouTube Travel Analytics** é um pipeline de dados que:

1. Coleta metadados e transcrições de todos os vídeos de um canal do YouTube
2. Usa um LLM (Claude) para extrair destinos visitados, duração em cada local, tipo de viagem e tom emocional
3. Geocodifica os destinos extraídos para coordenadas geográficas
4. Expõe os dados via API REST (FastAPI)
5. Apresenta um dashboard interativo com mapa mundi, timeline de viagens e estatísticas gerais

O projeto foi desenhado para canais de viagem, mas a arquitetura é adaptável para qualquer tipo de conteúdo que envolva extração de entidades a partir de transcrições.

---

## Funcionalidades

- **Coleta automatizada** de vídeos de qualquer canal público do YouTube via YouTube Data API v3
- **Transcrição automática** via `youtube-transcript-api`; fallback para OpenAI Whisper em vídeos sem legenda
- **Extração com IA** de destinos, durações, atividades e sentimento usando Claude API
- **Geocodificação** de destinos para coordenadas lat/lng via Google Geocoding API
- **Cache local** para evitar reprocessamento de vídeos já analisados
- **API REST** para consulta dos dados pelo dashboard
- **Dashboard interativo** com mapa de calor, timeline e cards de estatísticas

---

## Arquitetura

```
┌─────────────────────────────────────────────────────────────┐
│                        COLETA DE DADOS                       │
│                                                              │
│  YouTube Data API v3   youtube-transcript-api   Whisper      │
│   (metadados, stats)     (legendas/CC)         (fallback)    │
└────────────────────────────┬────────────────────────────────┘
                             │
┌────────────────────────────▼────────────────────────────────┐
│                     PROCESSAMENTO                            │
│                                                              │
│      Pipeline Python        SQLite + JSON       Cache        │
│    (limpeza, chunking)   (armazenamento)       (local)       │
└────────────────────────────┬────────────────────────────────┘
                             │
┌────────────────────────────▼────────────────────────────────┐
│                      ANÁLISE COM LLM                         │
│                                                              │
│  Extração de destinos   Timeline e duração   Temas/insights  │
│  + Geocodificação       (por vídeo)          (sentimento)    │
└────────────────────────────┬────────────────────────────────┘
                             │
┌────────────────────────────▼────────────────────────────────┐
│                    DASHBOARD INTERATIVO                       │
│                                                              │
│   Mapa mundi (Leaflet)   Timeline (D3.js)   Stats cards      │
│   FastAPI (backend)      React + Vite       Tailwind CSS     │
└─────────────────────────────────────────────────────────────┘
```

---

## Pré-requisitos

- Python 3.11+
- Node.js 20+
- Conta Google Cloud com YouTube Data API v3 habilitada
- Chave de API do Google Geocoding
- Chave de API da Anthropic (Claude)
- (Opcional) ffmpeg instalado localmente — necessário apenas para o fallback com Whisper

---

## Instalação

### 1. Clone o repositório

```bash
git clone https://github.com/seu-usuario/youtube-travel-analytics.git
cd youtube-travel-analytics
```

### 2. Backend (Python)

```bash
cd backend

# Crie e ative o ambiente virtual
python -m venv .venv
source .venv/bin/activate      # Linux/macOS
.venv\Scripts\activate         # Windows

# Instale as dependências
pip install -r requirements.txt
```

### 3. Frontend (React)

```bash
cd frontend
npm install
```

---

## Configuração

Crie o arquivo `.env` na raiz do projeto (ou em `backend/`) com base no `.env.example`:

```env
# YouTube Data API v3
YOUTUBE_API_KEY=sua_chave_aqui

# Anthropic (Claude)
ANTHROPIC_API_KEY=sua_chave_aqui

# Google Geocoding API
GEOCODING_API_KEY=sua_chave_aqui

# Canal alvo (ID ou @handle)
YOUTUBE_CHANNEL_ID=@nomeDoCanal

# Configurações do pipeline
MAX_VIDEOS=100             # Limite de vídeos a processar (0 = todos)
WHISPER_MODEL=base         # tiny | base | small | medium | large
CACHE_DIR=.cache           # Diretório de cache local
DB_PATH=data/analytics.db  # Caminho do banco SQLite
```

### Como obter as chaves de API

**YouTube Data API v3**
1. Acesse [Google Cloud Console](https://console.cloud.google.com)
2. Crie um projeto ou selecione um existente
3. Habilite a **YouTube Data API v3**
4. Gere uma chave em *APIs & Services → Credentials*

**Anthropic (Claude)**
1. Acesse [console.anthropic.com](https://console.anthropic.com)
2. Gere uma API key em *API Keys*

**Google Geocoding API**
1. No mesmo projeto do Google Cloud, habilite a **Geocoding API**
2. Use a mesma chave ou gere uma separada

---

## Uso

### Pipeline completo (recomendado)

Execute todas as etapas em sequência:

```bash
cd backend
python -m pipeline.run --channel @nomeDoCanal
```

### Etapas individuais

Você pode rodar cada etapa separadamente para inspecionar os resultados:

```bash
# 1. Coleta metadados e transcrições do canal
python -m pipeline.collect --channel @nomeDoCanal

# 2. Processa e limpa as transcrições coletadas
python -m pipeline.process

# 3. Envia as transcrições para análise com LLM
python -m pipeline.analyze

# 4. Geocodifica os destinos extraídos
python -m pipeline.geocode
```

### Iniciar a API

```bash
cd backend
uvicorn api.main:app --reload --port 8000
```

A documentação interativa da API estará disponível em `http://localhost:8000/docs`.

### Iniciar o dashboard

```bash
cd frontend
npm run dev
```

O dashboard estará disponível em `http://localhost:5173`.

---

## Estrutura do Projeto

```
youtube-travel-analytics/
│
├── backend/
│   ├── pipeline/
│   │   ├── __init__.py
│   │   ├── run.py              # Orquestrador do pipeline completo
│   │   ├── collect.py          # Coleta via YouTube API + transcript
│   │   ├── process.py          # Limpeza e chunking das transcrições
│   │   ├── analyze.py          # Extração de entidades com LLM
│   │   └── geocode.py          # Geocodificação dos destinos
│   │
│   ├── api/
│   │   ├── __init__.py
│   │   ├── main.py             # App FastAPI
│   │   └── routes/
│   │       ├── videos.py       # GET /videos, GET /videos/{id}
│   │       ├── destinations.py # GET /destinations, GET /destinations/map
│   │       └── stats.py        # GET /stats/summary
│   │
│   ├── models/
│   │   ├── video.py            # Pydantic models para vídeos
│   │   └── destination.py      # Pydantic models para destinos
│   │
│   ├── db/
│   │   ├── connection.py       # Conexão SQLite
│   │   └── schema.sql          # DDL do banco de dados
│   │
│   ├── prompts/
│   │   └── extract_destinations.txt  # Prompt de extração para o LLM
│   │
│   ├── requirements.txt
│   └── .env.example
│
├── frontend/
│   ├── src/
│   │   ├── components/
│   │   │   ├── Map.jsx         # Mapa mundi com Leaflet
│   │   │   ├── Timeline.jsx    # Timeline de viagens com D3
│   │   │   ├── StatsCards.jsx  # Cards com estatísticas
│   │   │   └── VideoList.jsx   # Lista de vídeos analisados
│   │   │
│   │   ├── pages/
│   │   │   └── Dashboard.jsx   # Página principal
│   │   │
│   │   ├── hooks/
│   │   │   └── useAnalytics.js # Hook para consumir a API
│   │   │
│   │   └── main.jsx
│   │
│   ├── package.json
│   └── vite.config.js
│
├── data/                       # Gerado em runtime (gitignore)
│   └── analytics.db
│
├── .cache/                     # Cache local (gitignore)
├── .env.example
├── .gitignore
└── README.md
```

---

## Schema do Banco de Dados

### Tabela `videos`

| Coluna | Tipo | Descrição |
|---|---|---|
| `id` | TEXT PK | ID do vídeo no YouTube |
| `title` | TEXT | Título do vídeo |
| `published_at` | TEXT | Data de publicação (ISO 8601) |
| `duration_seconds` | INTEGER | Duração total do vídeo |
| `view_count` | INTEGER | Visualizações |
| `like_count` | INTEGER | Likes |
| `transcript_raw` | TEXT | Transcrição bruta |
| `transcript_clean` | TEXT | Transcrição pós-processada |
| `processed_at` | TEXT | Timestamp do processamento |

### Tabela `destinations`

| Coluna | Tipo | Descrição |
|---|---|---|
| `id` | INTEGER PK | Auto-increment |
| `video_id` | TEXT FK | Referência ao vídeo |
| `name` | TEXT | Nome do destino como mencionado |
| `country` | TEXT | País (normalizado) |
| `city` | TEXT | Cidade (se identificada) |
| `region` | TEXT | Região/estado (se identificado) |
| `lat` | REAL | Latitude |
| `lng` | REAL | Longitude |
| `days_mentioned` | INTEGER | Dias mencionados na transcrição |
| `trip_type` | TEXT | aventura / cultural / praia / gastronomia / etc. |
| `sentiment` | TEXT | positivo / neutro / negativo |
| `confidence` | REAL | Confiança da extração (0.0–1.0) |

### Tabela `pipeline_log`

| Coluna | Tipo | Descrição |
|---|---|---|
| `video_id` | TEXT PK | ID do vídeo |
| `collect_status` | TEXT | ok / error / skipped |
| `analyze_status` | TEXT | ok / error / skipped |
| `geocode_status` | TEXT | ok / error / skipped |
| `last_updated` | TEXT | Timestamp da última atualização |

---

## Dashboard

O dashboard é composto por quatro seções principais:

**Mapa mundi interativo**
Exibe todos os destinos visitados com pins clicáveis e camada de heatmap proporcional ao número de visitas. Ao clicar em um destino, abre um painel lateral com os vídeos que mencionam aquele local.

**Timeline de viagens**
Visualização cronológica dos vídeos com as viagens associadas. Permite filtrar por ano, país ou tipo de viagem. Construída com D3.js.

**Cards de estatísticas**
Resumo quantitativo do canal: total de países visitados, continentes explorados, destino mais frequente, tempo médio por destino, vídeo com mais destinos em uma única viagem.

**Lista de vídeos**
Tabela paginada com todos os vídeos processados, mostrando os destinos extraídos, datas e links diretos para o YouTube.

---

## Roadmap

- [ ] Suporte a múltiplos canais simultâneos
- [ ] Atualização incremental automática (novos vídeos)
- [ ] Exportação dos dados em CSV e GeoJSON
- [ ] Comparação entre canais
- [ ] Detecção de rotas de viagem entre destinos consecutivos
- [ ] Integração com Notion para relatórios automáticos
- [ ] Deploy containerizado com Docker Compose

---

## Contribuindo

Contribuições são bem-vindas. Por favor, abra uma issue antes de submeter um pull request descrevendo o que você pretende alterar.

1. Fork o repositório
2. Crie uma branch: `git checkout -b feature/minha-feature`
3. Commit suas mudanças: `git commit -m 'feat: adiciona suporte a X'`
4. Push para a branch: `git push origin feature/minha-feature`
5. Abra um Pull Request

---

> Desenvolvido com Python, FastAPI, React e Claude API.
