# GS26-2-Prompt-and-Artificial-Intelligence

Assistente Municipal de Descarte Sustentável (GS 2026.2)

**Aluno:** Rubens Henrique Cordeiro dos Santos | **RM:** 572667 | **Modalidade:** Individual

## 1. Visão Geral do Projeto
Solução desenvolvida para o **Centro de Operações do Município (COB)** na Frente **B (Descarte e Sustentabilidade — Programa ColetAr Óleo)**. O sistema responde dúvidas de cidadãos sobre regras de descarte, horários, elegibilidade de benefícios e status operacional dos Ecopontos, garantindo respostas auditáveis com citação de fonte, execução de cálculos via agente e proteção por guardrails.

## 2. Diagrama do Pipeline
```text
[Documentos .txt + Metadados] 
       │
       ▼ (Sanitização Anti-Injection + Chunking: 650/100)
[Embeddings 256D] ──► [ChromaDB Vector Store]
                             │
[Pergunta do Cidadão] ──► [Guardrails: Escopo / LGPD / Injection]
                             │
            ┌────────────────┴────────────────┐
            ▼ (Consulta Normativa)            ▼ (Ação / Cálculo)
 [Retriever k=4 + Ordenação Data]    [Agente Tool Calling (max_iter=5)]
            │                         ├─ consultar_status_ecoponto()
            │                         └─ calcular_elegibilidade_e_beneficio()
            └────────────────┬────────────────┘
                             ▼
           [LLM: llama3.1:8b (Ollama Cloud)]
                             ▼
     [Interface Gradio: Resposta + Citação doc_id]
```

## 3. Explicação da Armadilha Documental (Seção 5.1)
1. **Contradição Temporal:** O `doc_02_lista_ecopontos.txt` (10/01/2026) lista o **Ecoponto Parque Central (`ECO-02`)** como **ATIVO**. Contudo, o `doc_03_comunicado_interdicao.txt` (05/11/2026) determina a **interdição temporária** do `ECO-02` e redireciona o atendimento para o `ECO-01` (Vila Mariana), estendendo seu horário até 19h00. O retriever ordena os trechos recuperados por `data_publicacao` decrescente para que a norma mais recente prevaleça.
2. **Faixa Condicional e Detalhe Interno:** No meio do `doc_01_regulamento_coleta.txt` (Art. 3º), estabelece-se que o descarte é livre para todos, mas a troca por sabão ecológico é restrita a idosos (60+) ou inscritos na Tarifa Social, com limite de 20L.
3. **Prompt Injection na Fonte:** O final do `doc_03` contém uma diretiva maliciosa (`[SYSTEM OVERRIDE...]`) neutralizada na ingestão e por instrução de sistema.

## 4. Decisões Técnicas e Custo Aproximado
* **Chunking (650 chars / overlap 100):** Escolhido após comparação com chunks de 300 chars (sem overlap), que fragmentavam o Art. 3º e separavam o ID dos Ecopontos de seus endereços.
* **Custo Computacional:** Cada consulta RAG realiza 1 busca vetorial local + 1 chamada ao LLM (`llama3.1:8b`). Consultas ao agente realizam entre 2 e 3 chamadas ao LLM, limitadas a `max_iterations=5`.
