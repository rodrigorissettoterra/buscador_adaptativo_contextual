# Buscador Adaptativo Contextual

Um buscador experimental que **não trabalha com uma lista fixa de fontes**. Em vez disso, interpreta o contexto do termo pesquisado, descobre fontes dinamicamente na web, classifica os tipos de site encontrados, seleciona um conjunto diversificado de cinco fontes relevantes e pesquisa novamente dentro desses domínios para apresentar páginas específicas, links diretos, relevância semântica e uma síntese rastreável.

A ideia central do projeto é simples:

> **a consulta deve ajudar a determinar não apenas quais resultados mostrar, mas também quais fontes fazem mais sentido para aquele contexto.**

O projeto foi desenvolvido para execução em Python, com foco inicial no Google Colab, e utiliza modelos semânticos locais, sem exigir uma API paga de LLM.

---

## Visão geral

Em vez de consultar sempre as mesmas fontes, o sistema executa um pipeline adaptativo:

```text
TERMO DO USUÁRIO
        ↓
INTERPRETAÇÃO DO CONTEXTO
        ↓
BUSCA AMPLA NA WEB
        ↓
CONSULTAS AUXILIARES CONTEXTUAIS
        ↓
CLASSIFICAÇÃO DOS TIPOS DE FONTE
        ↓
RANKING SEMÂNTICO DOS DOMÍNIOS
        ↓
SELEÇÃO DIVERSIFICADA DOS 5 SITES
        ↓
BUSCA DENTRO DE CADA DOMÍNIO
        ↓
EXTRAÇÃO DAS PÁGINAS
        ↓
RANKING SEMÂNTICO DOS RESULTADOS
        ↓
DEDUPLICAÇÃO
        ↓
RESULTADOS COM LINKS DIRETOS
        ↓
SÍNTESE EXTRATIVA + REFERÊNCIAS
```

Isso permite que uma consulta científica, técnica, prática, comercial ou baseada em opinião encontre conjuntos de fontes diferentes.

---

## O que foi desenvolvido

### 1. Interpretação semântica do contexto

A consulta do usuário é comparada semanticamente com diferentes tipos de intenção de busca.

Atualmente, o projeto considera:

- Informação geral
- Pesquisa científica
- Notícias / atualidade
- Prática / tutorial
- Técnica / documentação
- Compra / comparação
- Opinião / comunidade

A intenção detectada não limita a pesquisa. Ela é usada apenas para orientar a descoberta e ajustar levemente a importância de determinados tipos de fonte.

---

### 2. Expansão contextual da consulta

Além do termo original, o sistema cria automaticamente consultas auxiliares relacionadas ao contexto identificado.

Exemplo conceitual:

```text
Consulta original:
uso de metodologias ativas em instituições públicas

Consultas auxiliares possíveis:
uso de metodologias ativas em instituições públicas estudo pesquisa evidências
uso de metodologias ativas em instituições públicas artigo revisão literatura
```

Os resultados são combinados e deduplicados antes da avaliação dos domínios.

---

### 3. Descoberta dinâmica das fontes

O projeto não possui cinco sites previamente definidos.

A busca inicial encontra páginas de diferentes domínios. Esses resultados são agrupados por site e cada domínio recebe uma pontuação baseada em sinais como:

- relevância semântica para a consulta;
- posição dos resultados na descoberta;
- quantidade de aparições do domínio;
- presença em diferentes consultas auxiliares;
- tipo estimado de fonte;
- adequação ao contexto inferido.

Os cinco sites selecionados podem mudar completamente de uma pesquisa para outra.

---

## Classificação automática do tipo de fonte

O buscador tenta identificar o perfil de cada domínio encontrado.

As categorias atuais são:

| Tipo de fonte | Exemplos de perfil |
|---|---|
| Oficial / Governo | ministérios, secretarias, órgãos públicos |
| Acadêmica / Universidade | universidades, faculdades, repositórios institucionais |
| Científica / Periódico | revistas científicas, periódicos e bases acadêmicas |
| Notícias / Editorial | jornais, revistas e portais editoriais |
| Comunidade / Fórum | fóruns, comunidades e sites de perguntas e respostas |
| Comercial | lojas, marketplaces e páginas de produtos |
| Independente / Blog | blogs pessoais e produtores independentes |
| Especializada / Nicho | sites focados em uma área ou assunto específico |

Essa classificação é **heurística e probabilística**. Ela serve principalmente para diversificar as fontes e não deve ser interpretada como certificação absoluta de autoridade ou qualidade.

---

## Relevância semântica

O projeto utiliza o modelo multilíngue:

```text
sentence-transformers/paraphrase-multilingual-MiniLM-L12-v2
```

Os embeddings são usados para comparar semanticamente:

- consulta e intenções de busca;
- consulta e resultados encontrados;
- consulta e páginas extraídas;
- candidatos a domínio;
- frases utilizadas na síntese final.

O modelo é executado localmente no ambiente Python.

---

## Seleção diversificada das cinco fontes

A seleção final não corresponde simplesmente aos cinco maiores scores.

O algoritmo utiliza uma estratégia inspirada em **Maximum Marginal Relevance (MMR)**, considerando:

```text
RELEVÂNCIA SEMÂNTICA
        +
CONTEXTO
        +
TIPO DE FONTE
        +
DIVERSIDADE
        +
SINAL DE NICHO
```

Isso reduz a chance de retornar cinco grandes portais com conteúdos muito semelhantes quando existem fontes relevantes e complementares.

Um blog ou site pequeno não entra apenas por ser independente. Ele precisa continuar demonstrando boa aderência ao tema pesquisado.

---

## Modos de busca

O projeto possui três modos de seleção.

### Relevância máxima

Interfere pouco no ranking contextual.

É indicado quando o objetivo principal é privilegiar os candidatos mais bem pontuados, com pouca influência da diversidade.

### Equilibrada

Modo padrão recomendado.

Combina relevância com diversidade de fontes e concede um bônus moderado a fontes de nicho quando elas são realmente relevantes.

### Descoberta / web independente

Aumenta o peso da diversidade e do sinal de nicho.

É útil para encontrar:

- blogs especializados;
- pequenos produtores de conteúdo;
- sites independentes;
- fontes de nicho;
- conteúdos que podem ficar menos visíveis em buscas tradicionais.

---

## Busca dentro das fontes selecionadas

Depois que os cinco domínios são escolhidos, o sistema executa uma nova busca restrita a cada site.

Exemplo:

```text
site:dominio.com termo pesquisado
```

Assim, o domínio funciona apenas como uma **fonte candidata**. O resultado apresentado ao usuário é sempre uma **página específica**, com:

- título;
- URL direta;
- tipo de fonte;
- resumo ou conteúdo extraído;
- relevância semântica para o tema pesquisado.

---

## Extração de conteúdo

Quando possível, o sistema acessa a página encontrada e extrai seu conteúdo principal utilizando `trafilatura`.

Antes da extração, o notebook verifica `robots.txt`.

O sistema não tenta contornar:

- bloqueios do site;
- login;
- paywalls;
- restrições de acesso;
- páginas não permitidas por `robots.txt`.

Quando o conteúdo completo não pode ser extraído, o resultado ainda pode ser apresentado utilizando o resumo retornado pelo mecanismo de busca.

---

## Ranking dos resultados

Depois da busca interna nos cinco sites, as páginas encontradas recebem uma nova avaliação semântica considerando:

- título;
- resumo da busca;
- conteúdo principal extraído, quando disponível.

Os resultados são então ordenados por relevância para o termo original.

---

## Deduplicação

A deduplicação é feita em três níveis:

1. URL canônica;
2. título normalizado;
3. similaridade semântica entre resultados.

Isso ajuda a reduzir páginas repetidas, versões duplicadas ou conteúdos praticamente idênticos.

---

## Síntese rastreável

A síntese final é propositalmente **extrativa**.

Em vez de gerar livremente um texto com um LLM, o sistema seleciona frases reais das páginas encontradas com base em relevância semântica.

Cada trecho recebe um identificador de fonte.

Exemplo:

```text
- Trecho relevante encontrado na página original. [8fa13c22]
- Outro trecho relevante de uma segunda fonte. [d72a09fe]
```

A tabela de referências permite localizar a página original correspondente a cada identificador.

Essa abordagem prioriza:

- rastreabilidade;
- fidelidade ao conteúdo original;
- menor risco de informações inventadas.

---

## Apresentação dos resultados

A interface do notebook apresenta cada página encontrada em cards contendo:

- posição no ranking;
- título da página;
- link direto e clicável;
- domínio;
- tipo de fonte;
- relevância semântica;
- texto encontrado;
- status da extração.

Também são exibidas separadamente as cinco fontes escolhidas e a página representativa que contribuiu para a seleção de cada domínio.

---

## Tecnologias utilizadas

- Python
- Google Colab
- DDGS
- Sentence Transformers
- `paraphrase-multilingual-MiniLM-L12-v2`
- Trafilatura
- tldextract
- Requests
- NumPy
- Pandas
- ipywidgets
- openpyxl

---

## Estrutura conceitual do score de domínio

A pontuação considera diferentes sinais, incluindo:

```text
relevância semântica do melhor resultado
+
relevância média do domínio
+
posição na busca
+
recorrência
+
cobertura entre consultas auxiliares
+
ajuste contextual pelo tipo de fonte
```

Na seleção final das cinco fontes são aplicados ainda:

```text
bônus de diversidade
+
bônus moderado de nicho
-
penalização por fontes muito semelhantes
-
penalização por repetição excessiva do mesmo tipo
```

---

## Limitações conhecidas

Este projeto é experimental e possui limitações importantes:

- mecanismos gratuitos de busca podem aplicar rate limit;
- os resultados disponíveis dependem do universo retornado pela metabusca;
- alguns sites bloqueiam automação;
- páginas muito dependentes de JavaScript podem não ser extraídas corretamente;
- conteúdo protegido por login ou paywall não é acessado;
- `robots.txt` pode impedir a coleta;
- a classificação do tipo de fonte é heurística;
- o sinal de nicho não representa tráfego real, autoridade ou audiência;
- a interpretação de contexto ainda pode classificar algumas consultas incorretamente;
- a síntese é extrativa e prioriza fidelidade em vez de fluidez textual.

Para um ambiente de produção, a camada de busca pode ser substituída por uma API de busca mais estável sem alterar o restante da arquitetura.

---

## Possíveis evoluções

Entre as evoluções previstas ou possíveis estão:

- calibrar os pesos utilizando um conjunto de consultas de avaliação;
- melhorar a classificação automática dos tipos de fonte;
- adicionar reranking com CrossEncoder;
- detectar idioma automaticamente;
- extrair metadados estruturados das páginas;
- identificar autoria e data de publicação;
- melhorar a interpretação da intenção de busca;
- aprender com cliques e avaliações dos usuários;
- armazenar histórico de pesquisas;
- disponibilizar o motor como API;
- criar uma interface web independente;
- permitir filtros por tipo de fonte;
- criar versões verticalizadas para áreas específicas.

---

## Possibilidades de uso

A arquitetura pode ser adaptada para diferentes contextos, por exemplo:

- pesquisa técnica;
- pesquisa acadêmica;
- descoberta de conteúdos de nicho;
- levantamento de referências;
- comparação de produtos e serviços;
- pesquisa de notícias;
- documentação técnica;
- busca especializada por área do conhecimento.

Uma evolução particularmente interessante é a criação de buscadores verticais, nos quais a mesma arquitetura é restrita a um determinado universo de fontes e recebe regras próprias de qualidade e relevância.

---

## Estado atual

O projeto encontra-se em desenvolvimento experimental.

A versão atual implementa o pipeline completo de:

```text
contexto
→ descoberta
→ classificação de fonte
→ seleção diversificada
→ pesquisa interna
→ extração
→ ranking semântico
→ deduplicação
→ resultados com links diretos
→ síntese rastreável
```

Os próximos testes serão usados principalmente para calibrar a interpretação de contexto, a classificação das fontes e os pesos do ranking.

---

## Autor

**Rodrigo Terra**

- GitHub: Rodrigo Terra
