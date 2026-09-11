# Como aplicar no clone de `Gavaaaaa/Crai`

Base: commit `1370224` (10/09/2026, "Renomeia crai/ para app/"). Nada foi
commitado nem enviado — o patch é para você revisar e decidir.

## 1. Arquivos MODIFICADOS (7) — `crai_treino_modificados.patch`

```
app/crai/ml/anomaly_detector.py        train(fonte=), meta.json com fonte/volume/proveniência/versões,
                                       load() confere versões, snapshot segue a fonte do artefato
app/crai/ml/failure_classifier.py      train(fonte=, seed=), failure_classifier_meta.json, load() confere versões
app/crai/ml/payday_inference.py        train(fonte=), meta.json ampliado, load() confere versões
app/crai/ml/synthetic_data.py          fonte= nos 3 geradores + anti-circularidade + generate_voluntary_dataset()
app/crai/scripts/preparar_amostra_real.py  Fonte E (E-Commerce Churn), calibração, models/calibracao.json,
                                       proveniência feature a feature, fallback do BACEN
app/crai/scripts/train_all.py          --fonte, --voluntario-samples, --ativar-voluntario, --saida-json
app/requirements.txt                   torch==2.13.0, prophet==1.4.0 (eram faixas >=)
```

Aplicar, da raiz do clone (a pasta que contém `.git`):

```
git apply --check patch_crai/crai_treino_modificados.patch   # só confere
git apply         patch_crai/crai_treino_modificados.patch
```

## 2. Arquivos NOVOS (9 + evidências) — `arquivos_novos/`

Copie a pasta `arquivos_novos/app/` por cima do `app/` do clone (os caminhos já
são os finais):

```
app/README_treino.md
app/crai/ml/calibracao.py
app/crai/ml/voluntary_risk.py
app/crai/scripts/gerar_readme_treino.py
app/crai/scripts/sanity_check_fora_do_dominio.py
app/models/calibracao.json                 (único arquivo de models/ que vai para o git — exceção já existente no .gitignore)
app/tests/test_synthetic_data.py           (33 testes)
app/tests/test_train_fonte.py              (21 testes)
app/tests/test_readme_treino.py            (5 testes)
app/docs/evidencia/treino/*                (JSONs e logs das rodadas, PROVENIENCIA.json)
```

## 3. Conferir

```
cd app
pip install -r requirements.txt
pytest tests/ -q          # clone limpo: 1039 passed, 5 skipped
```

Se você descompactar `modelos_treinados/models_sintetico_calibrado.tar.gz` em
`app/models/`, dois testes de `tests/test_metricas_declaradas.py` vão reprovar
(AUC 0,6669 fora do gate [0,70; 0,92] e não citada na linha §4.6 do
`app/README.md`). É esperado e está explicado no `README_treino.md` §4.3; o
`README.md` principal não foi tocado.

## 4. O que NÃO mudou de comportamento

- `train()` sem o parâmetro `fonte` (ou com `fonte="sintetico"`) gera exatamente
  o mesmo dataset e o mesmo modelo de antes — `tests/test_synthetic_data.py::
  TestDefaultNaoMudou` trava o hash dos três geradores medido antes da alteração.
- O agente involuntário, o agente voluntário e a API não foram tocados. O
  candidato do risk_scorer voluntário é gravado como `voluntary_risk_candidato.
  joblib`, que `carregar_modelo()` não lê; só `--ativar-voluntario` (ou
  `VoluntaryRiskModel.ativar()`) promove o arquivo, e isso é decisão sua.
