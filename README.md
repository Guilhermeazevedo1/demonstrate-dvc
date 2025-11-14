# 🚀 Demonstração de DVC + MLflow: MLOps Simples

Este guia aborda o uso do DVC para versionar dados e pipelines de Machine Learning, utilizando uma pasta local como remote storage para facilitar o aprendizado.

## 💾 DVC: Versionamento de Dados e Setup

### 1. Clonagem e Setup Inicial

1. Clone este repositório em um local fora de pastas com sincronização em nuvem (como OneDrive) para evitar problemas de cache do DVC.

```bash
git clone <URL_DO_REPOSITORIO>
cd demonstrate-dvc
```

2. Crie uma pasta local para servir como Storage Remoto do DVC (onde os dados grandes serão guardados). Exemplo:

```bash
mkdir ~/Documentos/storage_dvc_remote
```

3. Defina essa nova pasta como o remote padrão do DVC:

```bash
dvc remote add -d local_remote ~/Documentos/storage_dvc_remote
```

(Ajuste o caminho conforme o local que você criou.)

### 2. Geração e Versionamento dos Dados

Como os dados de treino (`data/train`) e validação (`data/val`) não existem inicialmente, precisamos gerá-los e, então, adicioná-los ao controle do DVC.

1. Gere os dados executando o script de pré-processamento (ele salvará as imagens nas pastas `data/`):

```bash
python src/preprocess.py
```

2. Adicione os dados ao controle do DVC:

```bash
dvc add data/train
dvc add data/val
```

3. Confirme os metadados (`.dvc` files) no Git:

```bash
git add data/train.dvc data/val.dvc
git commit -m "Adiciona dados de treino e val ao DVC"
```

4. Envie o conteúdo real dos dados para o Storage Remoto do DVC:

```bash
dvc push
```

5. Teste o checkout (Opcional: apague as pastas e puxe novamente para confirmar):

```bash
# Apague as pastas de dados para simular um novo ambiente
rm -rf data/train data/val
# Restaure os dados do storage remoto
dvc pull
```

## ⚙️ Pipeline DVC

Seu pipeline está definido no arquivo `dvc.yaml` e se concentra na etapa de Treinamento (`train`).

### Stage `train`

| Parâmetro | Valor | Função |
|-----------|-------|--------|
| `cmd` | `python src/train.py ...` | Comando a ser executado. |
| `deps` | `src/train.py`, `data/train/`, `data/val/` | Arquivos/pastas que, se alterados, forçam a reexecução do stage. |
| `outs` | `models/model.pkl` | Artefato de saída (modelo) a ser versionado. |
| `metrics` | `metrics/metrics.json` | Arquivo de métricas a ser rastreado. |

Execute o seguinte comando para que o DVC seja responsável por rodar e rastrear toda a pipeline:

```bash
dvc repro
```

Após o `dvc repro`, o modelo e as métricas serão salvos. Não se esqueça de confirmar as mudanças no Git:

```bash
git add dvc.yaml models/model.pkl.dvc metrics/metrics.json
git commit -m "Executa pipeline e salva novo modelo/métricas"
git push
dvc push
```

## 📊 MLflow: Rastreamento de Experimentos

O script `src/train.py` já contém o código para rastrear parâmetros, métricas e o modelo usando o MLflow, definindo o experimento como `"demo-mlops"`.

Para visualizar o histórico de treinamento e as execuções do MLflow:

1. Execute a interface web do MLflow no terminal:

```bash
mlflow ui
```

2. Abra seu navegador em `http://localhost:5000` (ou a URL indicada) para ver as métricas e artefatos de cada execução de `dvc repro`.