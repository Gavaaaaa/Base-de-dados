# Base de dados e treino da IA — CRAI

Resultados do trabalho de base de dados e treino/retreino dos modelos da CRAI
(Retention OS para SaaS B2B brasileiro). O código-fonte vive em
[`Gavaaaaa/Crai`](https://github.com/Gavaaaaa/Crai); este repositório guarda
**as evidências, os parâmetros medidos e o patch** que produziram os números
abaixo, de forma que qualquer pessoa consiga reproduzir e auditar.

> **O que isto NÃO é:** não é treino com dado real de churn observado. Não
> existe base pública com o rótulo "esta cobrança falhada foi recuperada" nem
> "este cliente cancelou a assinatura". O que existe aqui: features calibradas
> em dado real de outras bases (declaradas uma a uma), o mecanismo de
> treino/retreino funcionando de ponta a ponta, a mitigação explícita da
> circularidade e uma checagem fora do domínio com a queda de performance
> registrada como saiu. Detalhe completo em [`README_treino.md`](README_treino.md).

## O que o professor pediu, e onde está a prova

| Pedido | Onde está |
|---|---|
| A IA treina | `evidencia/log_rodada_baixa.txt` e `log_rodada_alta.txt`: 4 modelos treinados do zero, artefatos salvos e recarregados via `load()` |
| O retreino responde à entrada de mais dados | `README_treino.md` §3 (duas rodadas por modelo, lado a lado) e §3.5 (curva de volume, 4 volumes × 3 seeds) |
| Parte das features vem de dado real | `calibracao.json → proveniencia_features` e `README_treino.md` §2 (status de cada feature: ancorada / proxy fraco / sintética sem doador) |
| O modelo não decora a fórmula do gerador | `README_treino.md` §4.1 (ruído/exceções em cada gerador) e §4.2 (checagem fora do domínio) |
| Tudo reproduzível | `README_treino.md` §8 + `patch_crai/` |

## Resumo dos números

Fonte dos parâmetros: `sintetico_calibrado` (Olist + BACEN + E-Commerce Customer Churn).

| Modelo | Volume (rodada 1 → 2) | Métrica principal (rodada 1 → 2) | Recarrega via `load()` |
|---|---|---|---|
| FailureClassifier (XGBoost + RF + SHAP) | 3.000 → 6.000 transações | AUC-ROC 0,6695 → 0,6669 (recall @0,25: 0,911 → 0,919; recall operacional 1,0) | sim |
| AnomalyDetector (autoencoder) | 5.500 → 11.000 clientes | ROC-AUC 0,9789 → 0,9810 (recall @p95: 0,899 → 0,933) | sim |
| PaydayInference (LSTM + Prophet) | 600 → 1.200 clientes × 180 dias | ROC-AUC diário 0,9418 → 0,9519 (MAE da janela 0,68 dia vs 4,4–4,6 da heurística) | sim |
| risk_scorer voluntário (candidato, não ativo) | 2.000 → 4.000 eventos | AUC 0,713 → 0,752 (teto das regras: 0,741 → 0,758) | sim |

Curva de volume do classificador (3 seeds por ponto): AUC 0,626 (n=1.000) → 0,681 (3.000) → 0,686 (6.000) → 0,696 (12.000), com o desvio-padrão caindo de 0,037 para 0,005. É isso que prova que o mecanismo responde a mais dado; dois pontos sozinhos ficam dentro do erro do holdout.

Checagem fora do domínio (só inferência, rótulo real `Churn` do dataset E-Commerce):
AnomalyDetector ROC-AUC 0,986 em domínio → **0,506** fora; regras do risk_scorer **0,405**; candidato **0,430**. A queda é o resultado esperado e está explicada no `README_treino.md` §4.2 — o doador tem direção invertida em dias/satisfação e os modelos só veem 4 das 12 features.

## Ponto que precisa ficar claro na apresentação

A AUC do classificador (0,67) fica **abaixo do piso 0,70** que o próprio repositório fixa como gate. Duas causas, as duas previstas e documentadas no `DATA_CARD.md` do projeto: o piso só aparece perto de 15.000 linhas, e a calibração do valor da fatura na Olist (mediana ~R$ 100) apaga o termo de valor do modelo causal do rótulo. Um parâmetro medido e declarado vale mais que um inventado que parecia certo — mas o número tem que ser dito, não escondido. Ver `README_treino.md` §4.3.

## Conteúdo deste repositório

```
README.md                         este arquivo
README_treino.md                  o documento completo (gerado a partir dos JSONs)
calibracao.json                   parâmetros MEDIDOS nos doadores reais + proveniência feature a feature
evidencia/
  rodada_baixa.json               métricas completas da rodada 1 (todos os modelos)
  rodada_alta.json                métricas completas da rodada 2
  fora_do_dominio.json            checagem fora do domínio + curva de volume
  log_rodada_*.txt                saída integral do train_all em cada rodada
  log_fora_do_dominio.txt         saída integral do sanity check
  meta_rodada_baixa/ meta_rodada_alta/   meta.json de cada artefato (fonte, volume, proveniência, versões)
  PROVENIENCIA.json               hashes SHA-256, contagens, nulos e describe() das fontes reais
  pytest_baseline_antes.txt       suíte ANTES do trabalho: 980 passed, 5 skipped
  pytest_final_clone_limpo.txt    suíte DEPOIS (clone limpo): 1039 passed, 5 skipped
  pytest_final_com_modelos.txt    suíte DEPOIS com os modelos em models/: 2 falhas esperadas (ver abaixo)
modelos_treinados/
  models_sintetico_calibrado.tar.gz   os artefatos da rodada 2 (.joblib/.pt/.json), para carregar sem retreinar
patch_crai/
  crai_treino_modificados.patch   diff dos 7 arquivos alterados em Gavaaaaa/Crai
  arquivos_novos/                 os arquivos novos, já nos caminhos certos (app/...)
  APLICAR.md                      como aplicar no clone do Crai
```

## Fontes de dado real (bruto fica fora do git)

| Fonte | Papel | Licença | Identidade |
|---|---|---|---|
| **Olist** — Brazilian E-Commerce Public Dataset (300 transações estratificadas de 103.877) | valor da fatura (MLE lognormal μ=4,5824 σ=0,8883), hora / dia da semana / dia do mês | CC BY-NC-SA 4.0 (acadêmico) | SHA-256 dos 3 CSVs em `PROVENIENCIA.json` |
| **BACEN SGS 21084** — inadimplência PF | peso de `insufficient_funds` nos códigos de erro (hipótese declarada no DATA_CARD §3) | ODbL | média 3,92 / último 5,81 (valores do DATA_CARD; a API estava fora de alcance na execução, e isso está gravado) |
| **E-Commerce Customer Churn** (Kaggle, ankitverma2010; 5.630 clientes, 20 colunas) | `days_since_last(_login)`, `features_used_30d`, `avg_session_min`, `tickets_30d`, `nps_last` | não declarada no Kaggle — uso acadêmico apenas | SHA-256 do xlsx `db70f1e34bc53268…` |

O rótulo `Churn` do E-Commerce **não entra em nenhum parâmetro nem treino** — só na checagem fora do domínio.

## Duas falhas de teste esperadas com os modelos em `models/`

`tests/test_metricas_declaradas.py` do Crai exige que a AUC do `train_metrics.json` presente em `app/models/` esteja em [0,70; 0,92] **e** seja citada na linha §4.6 do `app/README.md`. Com os artefatos desta rodada (AUC 0,6669) esses dois testes reprovam. É o gate de honestidade do próprio projeto funcionando: o `README.md` principal não foi alterado neste trabalho, e a linha §4.6 fica para quem mantém o README decidir. Num clone limpo (`models/` está no `.gitignore`) os testes pulam e a suíte fica 100% verde (1039 passed).

## Como reproduzir do zero

```
git clone https://github.com/Gavaaaaa/Crai && cd Crai
# aplicar o patch (ver patch_crai/APLICAR.md)
cd app
pip install -r requirements.txt            # versões exatas: sklearn 1.5.2, xgboost 2.1.1, torch 2.13.0, prophet 1.4.0
python -m crai.scripts.preparar_amostra_real
python -m crai.scripts.train_all --fonte sintetico_calibrado --classifier-samples 3000 --anomaly-samples 5500 --payday-customers 600 --voluntario-samples 2000 --saida-json rodada_baixa.json
python -m crai.scripts.train_all --fonte sintetico_calibrado --classifier-samples 6000 --anomaly-samples 11000 --payday-customers 1200 --voluntario-samples 4000 --saida-json rodada_alta.json
python -m crai.scripts.sanity_check_fora_do_dominio --saida fora_do_dominio.json
pytest tests/ -q
```

Tempo total das duas rodadas em CPU: ~3,5 minutos.
