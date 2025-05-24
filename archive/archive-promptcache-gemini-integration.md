# PromptCache System Integration with Gemini 2.5 Models - TimesTrader

**Data**: 2025-05-23  
**Tipo**: Sistema de Otimização de Performance  
**Status**: Planejado para Implementação

## Resumo Executivo

Este documento detalha as recomendações para implementação de um sistema PromptCache avançado no TimesTrader, integrando os novos modelos Gemini 2.5 (Flash e Pro) com suas capacidades nativas de caching para otimizar performance e reduzir custos operacionais.

## Contexto e Motivação

### Problemas Identificados

- **Alto custo de API calls** para modelos de IA (TimesNet, PPO, XGBoost)
- **Latência elevada** em operações de inferência
- **Redundância de prompts** similares em operações de trading
- **Necessidade de otimização** para operações em tempo real

### Oportunidades com Gemini 2.5

- **Implicit Caching**: 75% de economia de custos automaticamente
- **Context Caching**: Cache explícito para contextos longos
- **Performance superior**: Gemini 2.5 Flash (1024 tokens min) e Pro (2048 tokens min)
- **Integração com Supabase**: Uso do MCP para gerenciamento eficiente

## Arquitetura Recomendada

### Componentes Principais

1. **Cache Manager Central**

   - Gerenciamento unificado de cache para todos os componentes IA
   - Interface padrão para TimesNet, PPO agents, XGBoost
   - Configuração flexível por componente

2. **Supabase Persistent Backend**

   - Armazenamento persistente de prompts e respostas
   - Metadados de uso e performance
   - Integração com Supabase MCP

3. **Embedding-Based Similarity Engine**

   - Detecção de prompts similares usando embeddings
   - Threshold configurável para matches
   - Ranking de similaridade para múltiplos matches

4. **Intelligent Invalidation System**
   - Invalidação baseada em mudanças de mercado
   - TTL configurável por tipo de prompt
   - Cascata de invalidações para prompts dependentes

### Fluxo de Dados

```
Prompt Request → Similarity Check → Cache Hit/Miss →
    ↓ (Miss)                        ↓ (Hit)
Gemini API Call → Store in Cache → Return Response
```

## Estratégias de Implementação

### Fase 1: Arquitetura e Design

- **Task 21.1**: Design do sistema e interfaces
- **Duração**: 1-2 semanas
- **Entregáveis**: Diagramas de arquitetura, especificações de API

### Fase 2: Backend Persistente

- **Task 21.2**: Implementação do Supabase backend
- **Duração**: 2-3 semanas
- **Entregáveis**: Schema de banco, APIs de cache

### Fase 3: Sistema de Embeddings

- **Task 21.3**: Similarity engine
- **Duração**: 2 semanas
- **Entregáveis**: Sistema de embeddings, algoritmos de similaridade

### Fase 4: Invalidação Inteligente

- **Task 21.4**: Logic de invalidação
- **Duração**: 1-2 semanas
- **Entregáveis**: Sistema de triggers, políticas de invalidação

### Fase 5: Integração

- **Task 21.5**: Integração com componentes existentes
- **Duração**: 2-3 semanas
- **Entregáveis**: Integração completa, testes end-to-end

## Especificações Técnicas

### Modelos Gemini 2.5 Suportados

1. **Gemini 2.5 Flash**

   - Minimum context: 1024 tokens
   - Uso: Operações rápidas, validações XGBoost
   - Cache implicit: Automático

2. **Gemini 2.5 Pro**
   - Minimum context: 2048 tokens
   - Uso: Análises complexas TimesNet, PPO training
   - Cache explicit: Context caching API

### Schema Supabase

```sql
-- Tabela principal de cache
CREATE TABLE prompt_cache (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    prompt_hash TEXT UNIQUE NOT NULL,
    prompt_text TEXT NOT NULL,
    response_text TEXT NOT NULL,
    model_used TEXT NOT NULL,
    created_at TIMESTAMP DEFAULT NOW(),
    last_accessed TIMESTAMP DEFAULT NOW(),
    access_count INTEGER DEFAULT 1,
    ttl_seconds INTEGER,
    embedding VECTOR(768), -- Para similarity search
    metadata JSONB
);

-- Índices para performance
CREATE INDEX idx_prompt_cache_hash ON prompt_cache(prompt_hash);
CREATE INDEX idx_prompt_cache_embedding ON prompt_cache USING ivfflat(embedding);
CREATE INDEX idx_prompt_cache_created ON prompt_cache(created_at);
```

### Configuração de Cache

```python
class PromptCacheConfig:
    # Gemini 2.5 específico
    gemini_flash_min_tokens: int = 1024
    gemini_pro_min_tokens: int = 2048

    # Cache behavior
    similarity_threshold: float = 0.85
    default_ttl_hours: int = 24
    market_sensitive_ttl_hours: int = 4

    # Performance
    max_cache_size_mb: int = 1000
    embedding_batch_size: int = 50
```

## Benefícios Esperados

### Performance

- **75% redução** em custos de API (implicit caching)
- **80-90% redução** em latência para cache hits
- **50% melhoria** no throughput do sistema

### Operacional

- **Redução de dependência** externa de APIs
- **Maior robustez** do sistema
- **Métricas detalhadas** de uso e performance

### Financeiro

- **Economia significativa** em custos de API
- **ROI positivo** em 2-3 meses
- **Escalabilidade** de custos controlada

## Considerações de Implementação

### Segurança

- Criptografia de prompts sensíveis
- Controle de acesso baseado em roles
- Auditoria de uso de cache

### Monitoramento

- Métricas de hit/miss ratio
- Alertas para performance degradation
- Dashboard de custos e uso

### Manutenção

- Rotinas de limpeza de cache
- Backup e restore de dados
- Versionamento de schema

## Próximos Passos

1. **Aprovação da arquitetura** proposta
2. **Início da Fase 1** (Design detalhado)
3. **Setup do ambiente** de desenvolvimento
4. **Configuração do Supabase** MCP
5. **Desenvolvimento iterativo** por fases

## Dependências

- **Task 2**: Infraestrutura Supabase configurada
- **Task 4**: TimesNet implementado
- **Modelos Gemini 2.5**: Acesso à API configurado
- **Supabase MCP**: Servidor configurado no Cursor

## Riscos e Mitigações

### Risco: Complexidade de Similaridade

- **Mitigação**: Implementação gradual, testes extensivos

### Risco: Performance do Supabase

- **Mitigação**: Otimização de índices, monitoramento contínuo

### Risco: Cache Invalidation Bugs

- **Mitigação**: Logs detalhados, testes de regressão

---

**Documentado por**: IA Assistant  
**Última atualização**: 2025-05-23  
**Task relacionada**: #21 - Implementar Sistema de PromptCache
