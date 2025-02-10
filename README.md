# Cvm-financial-Scraper
pegar dados fundamentalistas da bolsa na CVM
import requests
from bs4 import BeautifulSoup
import pandas as pd
import os

def acessar_cvm():
    url = "https://www.gov.br/cvm/pt-br/assuntos/mercado"
    headers = {"User-Agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/91.0.4472.124 Safari/537.36"}
    response = requests.get(url, headers=headers)
    if response.status_code == 200:
        print("Acesso ao site da CVM bem-sucedido.")
        return response.text
    else:
        print("Erro ao acessar o site da CVM.")
        return None

def buscar_empresa(codigo_empresa):
    url_pesquisa = f"https://www.gov.br/cvm/pt-br/assuntos/mercado/companhias-abertas/dados-cadastrais?search={codigo_empresa}"
    headers = {"User-Agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/91.0.4472.124 Safari/537.36"}
    response = requests.get(url_pesquisa, headers=headers)
    if response.status_code == 200:
        soup = BeautifulSoup(response.text, 'html.parser')
        links = soup.find_all('a', href=True)
        print("Links encontrados:", [link['href'] for link in links])  # Debugging
        for link in links:
            if "balancos" in link['href'].lower():
                return link['href']
    return None

def extrair_dados(url_demonst):
    headers = {"User-Agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/91.0.4472.124 Safari/537.36"}
    response = requests.get(url_demonst, headers=headers)
    if response.status_code == 200:
        soup = BeautifulSoup(response.text, 'html.parser')
        tabelas = soup.find_all('table')
        if tabelas:
            df = pd.read_html(str(tabelas[0]))[0]
            return df
    return None

def salvar_dados(df, codigo_empresa, formato="csv"):
    if formato == "csv":
        df.to_csv(f"{codigo_empresa}_dados.csv", index=False)
    elif formato == "excel":
        df.to_excel(f"{codigo_empresa}_dados.xlsx", index=False)
    print(f"Dados salvos como {codigo_empresa}_dados.{formato}")

def main():
    print("Bem-vindo ao Extrator de Demonstrações Financeiras da CVM")
    codigo_empresa = input("Digite o código da empresa (ex: B3SA3): ")
    url_demonst = buscar_empresa(codigo_empresa)
    if url_demonst:
        print("Extraindo dados de:", url_demonst)
        df = extrair_dados(url_demonst)
        if df is not None:
            print(df.head())
            opcao = input("Deseja salvar os dados? (csv/excel/não): ").strip().lower()
            if opcao in ["csv", "excel"]:
                salvar_dados(df, codigo_empresa, opcao)
        else:
            print("Não foi possível extrair os dados.")
    else:
        print("Empresa não encontrada ou dados indisponíveis.")

if __name__ == "__main__":
    main()
