## Desafio: Tradutor de artigos com AzureIA
### Passo a passo:
#### Criando e provisionando
1. Criar um grupo de recursos:
![image](https://github.com/user-attachments/assets/7a893c3a-5028-4882-8740-6f6ca3b26381)
![image](https://github.com/user-attachments/assets/b1cba918-c582-4f48-80b7-541ee65d19ff)
2. Acessar o grupo de recursos criado: 
![image](https://github.com/user-attachments/assets/255ab1a6-1c1f-491d-a9c2-f33fae9adff5)
3. Criar uma instancia do serviço Azure OpenAI dentro do grupo de recursos:
![image](https://github.com/user-attachments/assets/dd3f5bc0-b81a-4ec4-9bb2-73ad845ce325)
4. Pesquisar Azure OpenAI no marketplace e criar o recurso abaixo:
![image](https://github.com/user-attachments/assets/05fbc6fa-8b62-4ca9-a660-58dfe6122834)
![image](https://github.com/user-attachments/assets/36436f8f-da17-4937-9774-79d25662bedf)
5. Implantar o modelo GPT-4 mini selecionando o modo básico e opção standard.
6. Anotar as chave API key 1 e API key 2 e Azure OpenAI Endpoint
![image](https://github.com/user-attachments/assets/d82e9169-ae8a-4b5a-bc43-4166d09b9fbb)
7. Do lado esquerdo da tela, em playgrounds, selecionar chat
![image](https://github.com/user-attachments/assets/198d615e-396d-4120-817c-cec9edb239df)
8. Atualize as configurações e faça um teste.
![image](https://github.com/user-attachments/assets/4ebcfd4b-f2c0-4e18-92f8-1bf9046d40cd)
#### Projeto: Tradução de artigos
1. Instalar pacotes:
![image](https://github.com/user-attachments/assets/ea39d301-c2d2-48b6-a64f-bb5ca43ce2cd)
2. Importar as bibliotecas
![image](https://github.com/user-attachments/assets/eb1810e6-4f2f-46cc-9709-d9eb18cde7d3)
3.  Código .py
```python
from bs4 import BeautifulSoup
import requests
import os
from dotenv import load_dotenv

# Carregar variáveis do arquivo .env
load_dotenv()
azureai_endpoint = os.getenv("AZURE_ENDPOINT")
API_KEY = os.getenv("AZURE_OPENAI_KEY")

# URL do artigo
url = "https://dev.to/eric_dequ/steve-jobs-the-visionary-who-blended-spirituality-and-technology-3ppi"

# Função para extrair o texto de uma página web
def extract_text(url):
    response = requests.get(url)
    if response.status_code == 200:
        soup = BeautifulSoup(response.text, 'html.parser')

        # Remove scripts e estilos para obter apenas texto
        for script in soup(["script", "style"]):
            script.decompose()

        text = soup.get_text(" ", strip=True)
        return text
    else:
        print("Falha ao buscar a URL. Código de status:", response.status_code)
        return None

# Testando a função
if __name__ == "__main__":
    extracted_text = extract_text(url)
    if extracted_text:
        print("Texto extraído com sucesso:\n")
        print(extracted_text[:500])  # Exibe os primeiros 500 caracteres do texto extraído
    else:
        print("Nenhum texto foi extraído.")
```
3. Saída teste do código anterior:
   > Texto extraído com sucesso:
Steve Jobs The Visionary Who Blended Spirituality and Technology - DEV Community Skip to content Navigation menu Search Powered by Search Algolia Search Log in Create account DEV Community Close Add reaction Like Unicorn Exploding Head Raised Hands Fire Jump to Comments Save More... Copy link Copy link Copied to Clipboard Share to X Share to LinkedIn Share to Facebook Share to Mastodon Report Abuse Eric Dequevedo Posted on Jun 28 • Originally published at rics-notebook.com Steve Jobs The Visiona

4. Código com a função de tradução de texto:
   ```python
from bs4 import BeautifulSoup
import requests
import os
from dotenv import load_dotenv

# Carregar variáveis do arquivo .env
load_dotenv()
azureai_endpoint = os.getenv("AZURE_ENDPOINT")
API_KEY = os.getenv("AZURE_OPENAI_KEY")


# Função para extrair texto de uma página web
def extract_text(url):
    """
    Extrai o texto limpo de uma página web, removendo scripts e estilos.
    """
    try:
        response = requests.get(url)
        response.raise_for_status()  # Lança exceção para erros HTTP
    except requests.RequestException as e:
        print(f"Erro ao acessar a URL: {e}")
        return None

    soup = BeautifulSoup(response.text, 'html.parser')

    # Remove scripts e estilos
    for script in soup(["script", "style"]):
        script.decompose()

    text = soup.get_text(" ", strip=True)
    return text


# Função para traduzir o texto usando Azure OpenAI
def translate_article(text, lang):
    """
    Traduz um texto para o idioma especificado utilizando o Azure OpenAI.
    """
    headers = {
        "Content-Type": "application/json",
        "api-key": API_KEY,
    }

    # Payload para a requisição
    payload = {
        "messages": [
            {
                "role": "system",
                "content": "Você atua como tradutor de textos."
            },
            {
                "role": "user",
                "content": f"Traduza o seguinte texto para o idioma {lang}: {text}. Responda apenas com a tradução no formato markdown."
            }
        ],
        "temperature": 0.7,
        "top_p": 0.95,
        "max_tokens": 900
    }

    # Envia a requisição para o endpoint configurado
    try:
        response = requests.post(azureai_endpoint, headers=headers, json=payload)
        response.raise_for_status()  # Lança exceção para erros HTTP
        result = response.json()
    except requests.RequestException as e:
        print(f"Erro na requisição ao Azure OpenAI: {e}")
        return None
    except KeyError:
        print("Erro ao processar a resposta da API. Verifique o payload e o endpoint.")
        return None

    # Retorna o conteúdo traduzido
    return result.get('choices', [{}])[0].get('message', {}).get('content', "Tradução não encontrada.")


# Testando as funções
if __name__ == "__main__":
    # URL de teste
    url = "https://dev.to/eric_dequ/steve-jobs-the-visionary-who-blended-spirituality-and-technology-3ppi"
    language = "português"  # Idioma para tradução

    # Extração de texto
    print("Extraindo texto do artigo...")
    article_text = extract_text(url)
    if article_text:
        print("Texto extraído com sucesso!")
        print(article_text[:500])  # Exibe os primeiros 500 caracteres para confirmação

        # Tradução de texto
        print("\nTraduzindo texto para o idioma solicitado...")
        translated_text = translate_article(article_text, language)
        if translated_text:
            print("Texto traduzido com sucesso!")
            print(translated_text)
        else:
            print("Falha na tradução.")
    else:
        print("Nenhum texto foi extraído.")
```
  
    

