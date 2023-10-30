# Código Python para baixar os arquivos da TISS Hospitalar

Código em Python para baixar os arquivos da TISS Hospitalar disponibilizados pela Agencia Nacional de Saúde Sumplementar (ANS) no Portal de Dados Abertos do Governo Federal (https://dados.gov.br/)

Os arquivos da TISS Hospitalar referem-se aos dados das internações hospitalares realizadas no Brasil pelas operadores de plano de saúde privadas reguladas pela ANS.

A página contendo os arquivos em CSV e o dicionário de dados explicativo dos dados denominados "Procedimentos Hospitalares por UF" estão disponíveis no endereço eletrônico https://dados.gov.br/dados/conjuntos-dados/procedimentos-hospitalares-por-uf 


## Notas de Uso

O código a seguir permite ao usuário realizar de forma seguencial e automatizada as seguintes operações: 

(1) O Download dos arquivos compactados (zip) na estrutura de diretórios do potal de dados abertos

(2) A descompactação de todos os arquivos baixados (zip) para o formato CSV

(3) a consolidação de todos os arquivos baixados e descompactados em um único CSV 

### Importar as bibliotecas necessárias:

```python
import requests
from bs4 import BeautifulSoup
import zipfile
import os
import pandas as pd
```
### Definir a URL de base, os anos e os Estados (UF) para download:
```python
BASE_URL = 'https://dadosabertos.ans.gov.br/FTP/PDA/TISS/HOSPITALAR/'
YEARS = ['2020', '2021', '2022']
STATES = ['AC', 'AL', 'AM', 'AP', 'BA', 'CE', 'DF', 'ES', 'GO', 'MA', 'MG', 'MS', 'MT', 'PA', 'PB', 'PE', 'PI', 'PR', 'RJ', 'RN', 'RO', 'RR', 'RS', 'SC', 'SE', 'SP', 'TO']
```

### Criação das funções:
```python
def get_links_from_page(url):
    page = requests.get(url)
    soup = BeautifulSoup(page.content, 'html.parser')
    return [link.get('href') for link in soup.find_all('a')]

def download_file(url, filename):
    response = requests.get(url, stream=True)
    with open(filename, 'wb') as f:
        for chunk in response.iter_content(chunk_size=8192):
            f.write(chunk)
```

### Baixar os arquivos: 
```python
all_files = []
for year in YEARS:
    for state in STATES:
        folder_url = BASE_URL + year + '/' + state + '/'
        local_dir = os.path.join(year, state)
        
        # Verificar e criar o diretório local se ele não existir
        if not os.path.exists(local_dir):
            os.makedirs(local_dir)
        
        links = get_links_from_page(folder_url)
        for link in links:
            if link.endswith('HOSP_CONS.zip'):
                zip_name = os.path.join(local_dir, link)
                full_link = folder_url + link
                download_file(full_link, zip_name)

                with zipfile.ZipFile(zip_name, 'r') as zip_ref:
                    zip_ref.extractall(local_dir)
                    csv_file = zip_name.replace('.zip', '.csv')
                    all_files.append(csv_file)
```
### Apendar todos os arquivos baixados em um único arquivo

```python
dfs = [pd.read_csv(file) for file in all_files]
final_df = pd.concat(dfs, ignore_index=True)
```

### Criar arquivo csv com todos os dados:
```python
final_df.to_csv('combined_data.csv', index=False)
```

## Contato 
Para suporte, elogios e reclamações, envie-me um e-mail para lepanitz@gmail.com
