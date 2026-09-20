# Base v2 da CRAI — pacote de entrega

Gerado em 20/09/2026. Semente 42.

```
LEIA-ME.md                      este arquivo
RELATORIO_BASE_V2.md            o que foi feito, o que foi medido, o que isso quer dizer
TAREFA_BLOCO_B.md               a especificação para o agente, em D:\CRAIV3\CraiV3

data/v2/
  populacao.parquet             120.000 clientes — a população compartilhada
  classificador.parquet         120.000 cobranças falhadas (grão: cobrança)
  comportamental.parquet        120.000 retratos de cliente (grão: cliente)
  liquidez.parquet            1.800.000 linhas (grão: cliente-dia, 10.000 clientes)
  voluntario.parquet            120.000 eventos de SDK (grão: evento)
  MANIFESTO.json                sha256, contagens, verificações, versões de biblioteca

codigo/
  populacao.py                  → app/crai/ml/populacao.py
  visoes.py                     → app/crai/ml/visoes.py
  gerar_bases_v2.py             → app/crai/scripts/gerar_bases_v2.py

amostras/
  *_amostra_500.csv             as 500 primeiras linhas de cada tabela, em CSV,
                                para abrir no Excel sem instalar nada
```

## Por que o código vem junto com a base

A base sozinha é um arquivo que ninguém pode conferir. Com os três `.py`, ela é
**reproduzível**: qualquer pessoa — você, o professor, a banca — roda

```
python -m crai.scripts.gerar_bases_v2 --seed 42 --out data/v2/
```

e obtém os mesmos cinco arquivos, com os mesmos sha256 do `MANIFESTO.json`. Foi
testado: duas gerações completas produziram hashes idênticos.

É isso que separa "base sintética" de "números que apareceram". A pergunta
"como vocês sabem que essa base é essa base?" tem resposta em uma linha de
comando.

## Onde commitar

A base é dado gerado, e `app/data/` está no `.gitignore` do repositório do
sistema — foi decisão do Bloco A, e ela está certa: o repositório de código não
carrega artefato de dado.

Duas opções, e a escolha é sua:

1. **`Gavaaaaa/Base-de-dados`** (o repositório de evidência). É onde o plano
   previa que a base ficasse. A favor: mantém o repositório do sistema limpo, e
   o manifesto com os sha256 amarra uma coisa na outra.
2. **`Gavaaaaa/CraiV3`**, abrindo exceção no `.gitignore` para `app/data/v2/`.
   A favor: uma coisa a menos para a banca procurar. Contra: contraria uma
   decisão que já foi executada, e 14,66 MB de parquet no repositório de código
   é o tipo de coisa que um avaliador técnico comenta.

Minha recomendação é a **1**, com o `MANIFESTO.json` também commitado dentro do
`CraiV3` (são 4 KB de texto): o repositório do sistema passa a declarar
exatamente qual base treinou os modelos, com hash, sem carregar os dados.

## O que ler primeiro

`RELATORIO_BASE_V2.md`, seção 5 — "O que isto quer dizer, em uma página". O
resto do relatório é a evidência por trás daquela página.
