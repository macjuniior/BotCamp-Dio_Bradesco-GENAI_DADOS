# Desafio Microsoft Application Platform
---
## Aprendizado
### Documentação do Sistema: Cadastro e Listagem de Produtos
Este documento descreve a arquitetura e o funcionamento do sistema de gerenciamento de produtos desenvolvido com Streamlit, Azure Blob Storage e SQL Server.
--


### 🏗️ Arquitetura do Sistema
O sistema utiliza uma abordagem híbrida para o armazenamento de dados, otimizando o desempenho e a escalabilidade:

Arquivos de Mídia (Imagens): São enviados para o Azure Blob Storage. O armazenamento em nuvem gera uma URL única para cada arquivo.

Dados Estruturados (Banco de Dados): O SQL Server armazena os metadados do produto (nome, preço, descrição) e a URL da imagem gerada pelo Azure.

📝 Componentes Principais
1. Segurança e Configuração (.env)
O projeto utiliza a biblioteca python-dotenv para carregar credenciais sensíveis. Isso garante que chaves de conexão e senhas do banco de dados não fiquem expostas no código-fonte, seguindo as melhores práticas de segurança.

2. Upload para a Nuvem (upload_blob)
A função de upload gerencia a transferência de arquivos para o Azure:

Identificação Única: Utiliza uuid.uuid4() para garantir que cada imagem tenha um nome exclusivo, evitando que arquivos com nomes iguais se sobrescrevam.

Acesso Remoto: Retorna a URL pública necessária para renderizar a imagem no navegador do usuário final.

3. Persistência de Dados (insert_product)
A inserção no SQL Server foi desenvolvida com foco em estabilidade:

Consultas Parametrizadas: O uso de %s protege o sistema contra ataques de SQL Injection e evita erros de sintaxe causados por caracteres especiais (como aspas em nomes de produtos).

Gerenciamento de Conexão: Garante a abertura e o fechamento correto do túnel de comunicação com o banco de dados.

4. Interface de Usuário e Listagem (list_produtcs_screen)
A exibição utiliza o sistema de grid do Streamlit:

Layout Responsivo: Os produtos são organizados em colunas (st.columns(3)), criando uma visualização de "vitrine".

Renderização Customizada: O uso de st.markdown com HTML/CSS permite o controle do tamanho das imagens (object-fit: cover) para que a interface permaneça uniforme, independente do tamanho original da foto enviada.

---
### ❌ Variação: Erros Encontrados e Soluções
Esta seção detalha os problemas técnicos enfrentados durante a implementação e como foram corrigidos.

1. Erro de Sintaxe SQL (Código 102)
O que apareceu: Incorrect syntax near 'f0598ff0'...

Causa: O código estava tentando enviar o objeto do arquivo (o "binário" da imagem) diretamente para a query SQL. O SQL não entende o objeto e gera um erro de sintaxe ao tentar converter caracteres aleatórios.

Solução: Alterado o fluxo para primeiro realizar o upload_blob(file), obter a URL (string) resultante e só então realizar o INSERT no banco.

2. Nome de Coluna Inválido (Código 207)
O que apareceu: Invalid column name 'image_url'

Causa: Houve uma divergência entre o nome da coluna definida na tabela do SQL Server e o nome utilizado no comando INSERT dentro do Python (ex: o banco esperava imagem_url e o código enviou image_url).

Solução: Sincronização dos nomes das colunas na função insert_product para corresponderem exatamente ao esquema do banco de dados.

3. Falha na Exibição (Imagem Quebrada)
O que apareceu: Um ícone de link quebrado na lista de produtos.

Causa A (Código): O atributo HTML foi escrito incorretamente como <img scr="...">.

Causa B (Azure): O container do Azure estava configurado como "Privado", impedindo o navegador de carregar a imagem via URL.

Solução: Correção do atributo para src (source) e alteração do nível de acesso do container no Azure para "Blob (acesso anônimo)".

📝 Funcionalidades Técnicas
Upload com Identidade Única
Para evitar que imagens com o mesmo nome (ex: produto.jpg) se sobrescrevam, o sistema utiliza:

Python
blob_name = str(uuid.uuid4()) + file.name
Isso gera um identificador único para cada upload realizado.

Consultas Parametrizadas
Para evitar erros de aspas e ataques de SQL Injection, a função de inserção utiliza parâmetros:

Python
query = "INSERT INTO Produtos (...) VALUES (%s, %d, %s, %s)"
cursor.execute(query, (product_name, product_price, product_description, image_url))
Visualização em Grade (Grid)
A listagem utiliza o componente st.columns(3) do Streamlit, permitindo uma visualização moderna em formato de cards, com tratamento de imagem via CSS para garantir que todas tenham o mesmo tamanho na tela.

Dica de Manutenção: Sempre que alterar a estrutura da tabela no SQL Server, lembre-se de atualizar os dicionários e queries nas funções insert_product e list_products.


## Prints
<img width="1489" height="700" alt="image" src="https://github.com/user-attachments/assets/f59a8d4d-cf95-498a-951c-7089dcb9ba06" />
<img width="1039" height="479" alt="image" src="https://github.com/user-attachments/assets/7f0ba9b1-89c6-467f-82d6-1c62b8bdd6ee" />
<img width="1064" height="755" alt="image" src="https://github.com/user-attachments/assets/fdc7995f-df22-41cc-a4ed-ce7a64dccf35" />
<img width="1039" height="479" alt="image" src="https://github.com/user-attachments/assets/b74cb8da-3481-4ca3-8f8a-e1850ad65d32" />
<img width="251" height="454" alt="image" src="https://github.com/user-attachments/assets/11b764c6-37fa-4137-afda-0c66e0cae00b" />
<img width="1360" height="787" alt="image" src="https://github.com/user-attachments/assets/ac88f367-6b99-47ce-82b6-c2f05b8584ea" />
<img width="636" height="344" alt="image" src="https://github.com/user-attachments/assets/263ee878-7aae-49b9-8556-f61af4b5c312" />
<img width="1858" height="977" alt="image" src="https://github.com/user-attachments/assets/61a71138-c474-48a2-a8c7-7747467af63e" />









