# Blue Blood — labs no Databricks

Notebooks dos labs da disciplina **Tomada de Decisão Baseada em Dados para Gestores de Segurança Pública** (CAES-PMERJ / FGV), 2026.

A **Blue Blood** é uma força policial fictícia cuja área coincide com o estado do Rio de Janeiro: Comando de Área = RISP, batalhão = AISP, companhia = CISP. Organização, pessoas e números internos são fictícios; os dados de criminalidade são reais e públicos (ISP-RJ).

## Os labs

| Notebook | Aula | O que o grupo faz |
| --- | --- | --- |
| `lab-1.ipynb` | 1 — 03/11, on-line | Forma o Alto Comando, nomeia a decisão, define critérios com pesos (AHP), indicador e limiar |
| `lab-2.ipynb` | 2 — 05/11, presencial | NLP em narrativas de boletins, anonimização, dashboard do ISP em número e taxa, checagem de notícias com `ai_decide` |
| `lab-3.ipynb` | 3 — 11/11, assíncrona | Ensaio do arco sobre uma notícia: critérios antes dos dados, série do ISPDados, posição com risco de errar |
| `lab-4.ipynb` | 4 — 12/11, laboratório | Miniprojeto: o arco completo no sub-cenário do grupo (Bronze, Silver, Gold, NLP, notícias, decisão e política pública) |

As células marcadas como prontas só se executam; o grupo preenche as marcadas com `TODO`. A última célula de cada notebook confere o que falta.

## O espaço `blue-blood` no Databricks

O lab 1 cria um espaço persistente, que os labs seguintes reabrem e completam:

| Onde | O que guarda |
| --- | --- |
| Catálogo `blue-blood` | Tudo da Blue Blood, de uma aula para a outra |
| Zona `bronze` | O dado como chegou: séries do ISP, narrativas, registro de ingestão |
| Zona `silver` | Dado limpo, tipado e checado |
| Zona `gold` | Indicadores e todas as decisões e parâmetros do grupo, em tabelas com o prefixo do lab (`lab1_`, `lab2_`...) |
| Volume `bronze.arquivos` | Os arquivos baixados |
| Volume `bronze.isp_microdados` | Restrito: a base do ISP Microdados que cada aluno baixa (lab 4) |

Os labs são escritos em SQL. Python aparece só onde o SQL não alcança: baixar arquivos para o volume, agrupar textos com k-means, desenhar gráficos e seguir em frente quando o `ai_decide` não existe no workspace.

## Como usar

1. Crie uma conta no [Databricks Free Edition](https://www.databricks.com/learn/free-edition).
2. Baixe o notebook da aula e importe em **Workspace → (seu usuário) → Import → File**.
3. Conecte ao compute *Serverless* e execute as células na ordem. A primeira célula cria (ou reabre) o catálogo `blue-blood`. Se `CREATE CATALOG` der erro, crie o catálogo pela tela: **Catalog → + → Create catalog**, nome `blue-blood`, armazenamento *Default Storage*.

Os notebooks baixam os dados públicos na hora. O Free Edition restringe o acesso à internet: se o download falhar, o notebook grava no volume uma amostra real embutida (séries do ISPDados das CISPs da RISP 1 e das CISPs 33 e 64, jan/2024 a set/2026; 12 narrativas do corpus KDMiLe) e avisa.

## Dados e licenças

| Fonte | Uso | Condição |
| --- | --- | --- |
| [Corpus KDMiLe 2026](https://github.com/murilogoes/bo-anonymizer-ptbr): 996 narrativas de roubo descaracterizadas | NLP e anonimização | CC BY 4.0; citar ALMEIDA, M. G. de; BOTEGA, L. C.; LIMA, P. M. de P. *Local Anonymization of Free-Text Robbery Police Reports in Brazilian Portuguese*. KDMiLe, SBC, 2026 |
| [ISPDados](https://www.ispdados.rj.gov.br/): séries mensais por CISP e população | Evidência e plano B | Download livre |
| [ISP Microdados](https://ispconecta.rj.gov.br/microdados/) | Base do miniprojeto (lab 4) | Cada aluno baixa com as próprias credenciais e o Termo de Responsabilidade |

**A base do ISP Microdados não entra neste repositório.** A [Portaria ISP nº 127/2022](http://arquivo.proderj.rj.gov.br/isp_imagens/Uploads/PortariaISP127_Fornecimento_Microdados.pdf) proíbe comercializar ou repassar a base a terceiros. O arquivo fica só no workspace de quem baixou, no volume restrito `bronze.isp_microdados`, e o `.gitignore` bloqueia planilhas e arquivos de dados.

## Julgamentos com IA

Os labs 2 e 4 usam a função SQL [`ai_decide`](https://docs.databricks.com/aws/en/sql/language-manual/functions/ai_decide) do Databricks para julgamentos tipados (sim ou não com probabilidade), no padrão do [TypeSafe System One](https://docs.typesafe.ai/concepts/how-to-build-with-system-one) visto em aula. Não é preciso criar chave de API. A função está em Beta e só existe em algumas regiões: se o seu workspace não a tiver, a célula avisa e segue com resultados de exemplo.
