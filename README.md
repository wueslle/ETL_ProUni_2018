# Projeto 1 - README

## 📋 Sobre o Projeto
Este projeto integra uma API com um banco de dados para realizar extração, armazenamento e análise de dados. Inclui autenticação em um endpoint de API, criação de tabelas no banco de dados, inserção em massa de dados e execução de queries estratégicas para análise.

## 🔧 Tecnologias Utilizadas
- Python 3.8+
- psycopg2
- pandas
- requests
- python-dotenv

## 🚀 Como Executar

### Pré-requisitos
- Instale as dependências:
```bash
pip install -r requirements.txt
```
- Configure suas credenciais da API e do banco de dados utilizando variáveis de ambiente.

### Configuração do Token e Banco de Dados
1. Crie um arquivo `.env` na raiz do projeto.
2. Adicione o token da API e as credenciais do banco no formato:
```env
# Token da API
API_TOKEN=seu_token_aqui

# Configuração do banco de dados
DB_NAME=nome_do_db
DB_USER=usuario
DB_PASSWORD=senha
DB_HOST=endereço_do_host
DB_PORT=porta
```
3. No código, utilize a biblioteca `python-dotenv` para carregar as variáveis:
```python
from dotenv import load_dotenv
import os

load_dotenv()

# Configuração do Token
API_TOKEN = os.getenv("API_TOKEN")

# Configuração do Banco de Dados
database_config = {
    'dbname': os.getenv("DB_NAME"),
    'user': os.getenv("DB_USER"),
    'password': os.getenv("DB_PASSWORD"),
    'host': os.getenv("DB_HOST"),
    'port': os.getenv("DB_PORT"),
}
```

### Configuração do Banco de Dados
- Crie as tabelas necessárias executando as queries DDL do notebook ou utilizando scripts SQL:
```sql
CREATE TABLE IF NOT EXISTS curso_prouni (
            UF VARCHAR,
            Cidade VARCHAR,
            Universidade VARCHAR,
            Curso VARCHAR,
            TURNO VARCHAR,
            MENSALIDADE FLOAT,
            BOLSA_INTEGRAL_AMPLA FLOAT,
            NOTA_INTEGRAL_AMPLA FLOAT,
            BOLSA_INTEGRAL_COTA FLOAT,
            NOTA_INTEGRAL_COTA FLOAT )
```

### Execução
- Execute o notebook `Projeto_1_Final.ipynb` para:
  - Testar a conexão com a API
  - Criar as tabelas no banco de dados
  - Inserir os dados extraídos no banco
  - Realizar queries estratégicas

## 📊 Estrutura do Banco de Dados
### Tabela Principal
**curso_prouni**

| Coluna                   | Tipo     | Descrição                                |
| ------------------------ | -------- | ---------------------------------------- |
| UF                       | VARCHAR  | Unidade Federativa (estado)              |
| Cidade                   | VARCHAR  | Cidade onde está localizada a universidade |
| Universidade             | VARCHAR  | Nome da universidade                     |
| Curso                    | VARCHAR  | Nome do curso                            |
| TURNO                    | VARCHAR  | Turno do curso (ex.: manhã, tarde, noite, à distância) |
| MENSALIDADE              | FLOAT    | Valor da mensalidade do curso            |
| BOLSA_INTEGRAL_AMPLA     | FLOAT    | Quantidade de bolsas integrais ampla concorrência |
| NOTA_INTEGRAL_AMPLA      | FLOAT    | Nota de corte para bolsa integral ampla    |
| BOLSA_INTEGRAL_COTA      | FLOAT    | Quantidade de bolsas integrais para cotas |
| NOTA_INTEGRAL_COTA       | FLOAT    | Nota de corte para bolsa integral de cotas |

## 📈 Explicação do Código
### Função `processar_paginas`
```python
def processar_paginas(): #Função principal - percorre as páginas e chama a função dados_pagina, se houver dados, chama a função insert_sql
    paginas = 42
    for page in range(1, paginas + 1):
        print(f"Processando página {page} de {paginas}...")
        dados = dados_pagina(page)
        if dados:
            insert_sql(dados)
        else:
            print(f"Nenhum dado encontrado na página {page}.")
```

Esta função principal é responsável por iterar pelas páginas da API e processar os dados disponíveis em cada uma delas:
1. Define o número total de páginas a serem processadas (`42`).
2. Em cada iteração, chama a função `dados_pagina` para obter os dados da API.
3. Caso existam dados, insere no banco usando a função `insert_sql`.

### Função `dados_pagina`
```python
def dados_pagina(page): # Função responsável por acessar os dados do endpoint por página e retornar os valores para a função processar_paginas
    url = f"{url_base}?page={page}"
    response = requests.get(url, headers= {"Authorization": f"Token {token}", "User-Agent": "python-requests/brasilio-client-0.1.0"})
    if response.status_code == 200:
        return response.json()["results"]
    else:
        print(f"Erro ao buscar página {page}: {response.status_code}")
        return []
```

Realiza a requisição HTTP para uma página específica da API:
- Constrói a URL com base no número da página.
- Envia a requisição usando `requests.get`, incluindo o token no cabeçalho de autenticação.
- Verifica se a resposta foi bem-sucedida (`status_code == 200`).
- Retorna os dados no formato JSON ou uma lista vazia em caso de erro.

### Função `insert_sql`
```python
def insert_sql(data): # Função responsável por inserir os dados recebidos pela função dados_pagina no banco de dados através de bulk insert
    with psycopg2.connect(**database_config) as conn:
        with conn.cursor() as cur:
            query = """
            INSERT INTO curso_prouni (UF, Cidade, Universidade, Curso, TURNO, MENSALIDADE, BOLSA_INTEGRAL_AMPLA, NOTA_INTEGRAL_AMPLA, BOLSA_INTEGRAL_COTA, NOTA_INTEGRAL_COTA)
            VALUES %s
            """
            records = [
                (
                    item["uf_busca"],
                    item["cidade_busca"],
                    item["universidade_nome"],
                    item["nome"],
                    item["turno"],
                    item["mensalidade"],
                    item["bolsa_integral_ampla"],
                    item["nota_integral_ampla"],
                    item["bolsa_integral_cotas"],
                    item["nota_integral_cotas"],
                )
                for item in data
            ]
            extras.execute_values(cur, query, records)
            conn.commit()
```

Insere os dados recebidos no banco de dados usando bulk insert:
1. Conecta ao banco usando as credenciais definidas em `database_config`.
2. Percorre os dados e cria uma lista de tuplas formatados para inserção na tabela `curso_prouni`.
3. Usa `psycopg2.extras.execute_values` para executar um comando SQL de inserção.


## 📝 Notas
- Os dados da API não mudam, uma vez que são dados de 2018
- Pelo volume grande de dados (41447 linhas) foi escolhido o método de bulk insert
- o dashboard pode ser encontrado no link: [Análise dos cursos do prouni 2018](https://public.tableau.com/app/profile/wueslle.thibes8447/viz/Livro1_17347475273590/Painel1)



