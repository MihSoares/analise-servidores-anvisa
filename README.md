# README — Análise de Servidores da ANVISA

## O que este script faz

Conta o número de servidores ativos da **Agência Nacional de Vigilância Sanitária (ANVISA)** a partir do arquivo exportado pelo Portal da Transparência.

Gera dois relatórios em Excel:
- `relatorio_anvisa_por_estado.xlsx` — total de servidores por estado + total Brasil
- `relatorio_anvisa_por_unidade.xlsx` — total de servidores por unidade (UORG) + total Brasil

---

## Arquivo necessário

Baixe o arquivo de servidores em:
> https://www.gov.br/anvisa/pt-br/composicao/diretor-presidente/gerencia-geral-de-gestao-de-pessoas

Ou pelo Portal da Transparência:
> https://portaldatransparencia.gov.br/servidores → filtrar por órgão: **ANVISA**

| Arquivo | Descrição |
|---|---|
| `servidor__1_.csv` | Lista de servidores exportada do portal (separador `;`, encoding UTF-8 BOM) |

Coloque o arquivo na **mesma pasta** que o script.

---

## Como usar

### 1. Instalar dependências

```bash
pip install pandas openpyxl
```

### 2. Executar

```bash
python analise_anvisa.py
```

---

## Como funciona por dentro

### Remoção de duplicatas

O arquivo bruto contém **duas linhas por servidor**: uma com o cargo real e outra com `"Sem informaç"` (registro de função/CCT). O script resolve isso assim:

```
Para cada CPF:
  → prioriza a linha com cargo real (não começa com "Sem informaç")
  → descarta a linha duplicada
```

Resultado: de **1.897 linhas brutas** → **1.441 servidores únicos**.

### Identificação da UF por UORG

O arquivo não tem uma coluna de estado explícita. A UF é extraída do nome da UORG usando a seguinte lógica:

```
Nome da UORG termina com "-XX"?  (ex: PAF-SP, PAF-RS, PAF-MG)
    ↓ SIM → extrai a sigla como UF
    ↓ NÃO → assume DF e mantém o nome original da UORG
```

**Exemplos:**

| UORG | UF atribuída |
|---|---|
| `COORDENACAO ESTADUAL VIG.SANT.DE PAF-SP` | SP |
| `COORDENACAO REGIONAL VIG.SANT.DE PAF-RJ` | RJ |
| `COORD REGIONAL VIG SANIT DE PAF-NE` | — (NE não é UF válida → **DF**) |
| `GERENCIA DE LOGISTICA` | **DF** (sem sufixo de UF) |
| `GABINETE DO DIRETOR PRESIDENTE` | **DF** (sede em Brasília) |
| `PVPAF GUARULHOS` | **DF** (sem sufixo → assume DF) |

> **Atenção:** UORGs como `PVPAF GUARULHOS` ou `PVPAF SANTOS` não têm sufixo `-XX`, portanto são atribuídas ao DF por padrão. Se quiser corrigir manualmente, adicione um dicionário de exceções no script na seção indicada.

---

## Estrutura dos relatórios gerados

### `relatorio_anvisa_por_estado.xlsx`

| UF | Estado | Qtd_Servidores |
|---|---|---|
| BR | TOTAL BRASIL | _total_ |
| DF | Distrito Federal | _n_ |
| SP | São Paulo | _n_ |
| ... | ... | ... |

### `relatorio_anvisa_por_unidade.xlsx`

| Unidade (UORG) | UF | Estado | Qtd_Servidores |
|---|---|---|---|
| TOTAL BRASIL | BR | — | _total_ |
| AGENCIA NACIONAL DE VIGILANCIA SANITARIA | DF | Distrito Federal | _n_ |
| COORDENACAO ESTADUAL VIG.SANT.DE PAF-SP | SP | São Paulo | _n_ |
| ... | ... | ... | ... |

---

## Fonte dos dados

- **Órgão:** ANVISA — Agência Nacional de Vigilância Sanitária
- **Portal:** https://www.gov.br/anvisa e https://portaldatransparencia.gov.br
- **Situação considerada:** Apenas servidores **Ativos**
