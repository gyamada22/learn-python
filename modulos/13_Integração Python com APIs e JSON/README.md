# 🐍 Módulo 13 — Integração Python com APIs e JSON  
## ✅ Status: CONCLUÍDO
- **Aulas:** 7/7  
- **Duração Total:** ~3h37min  
- **Progresso:** 100%

---

## 🎯 Objetivo
Aprender a integrar Python com APIs externas, consumir e manipular dados em JSON, criar e gerenciar REST APIs, automatizar requisições e entender conceitos de comunicação entre sistemas, preparando para projetos práticos e integrações em aplicações reais.

---

## 📌 Conteúdos Abordados

### 🔹 Conceitos de API
- API: interface que permite comunicação padronizada entre sistemas  
- Facilita integração, reutilização e segurança  
- Exemplo em Python: `qwe = request.get('LINK-API'); qwe = qwe.json()`  
- JSON: formato de texto padronizado, em Python normalmente convertido para dicionário  
- REST API: tipo de API que utiliza protocolo HTTP e métodos (GET, POST, PUT, DELETE)  

---

### 🔹 Métodos HTTP e CRUD
- **GET:** pegar informações  
- **POST:** criar informações  
- **PUT:** atualizar informações  
- **DELETE:** deletar informações  

- **Principais conceitos REST:**
  - Recursos → tudo é tratado como recurso (`/usuarios`, `/produtos`)  
  - URLs claras → representam recursos, não ações  
  - Métodos HTTP → definem ação  
  - Stateless → cada requisição é independente  
  - Resposta padronizada → status code + dados  

- **Códigos de status HTTP:**
  - 200 → sucesso  
  - 404 → recurso não encontrado  
  - 500 → erro interno do servidor  

- **CRUD:** ações básicas em bancos de dados  
  - CREATE, READ, UPDATE, DELETE  

---

### 🔹 Python e Integrações de API
- Realização de requisições HTTP com Python  
- Consumo de APIs de cotação de moeda  
- Exercícios práticos para manipulação de respostas JSON  

---

### 🔹 APIs com Autenticação
- Uso de login/API key para acesso  
- Exemplo: envio de SMS via Python + Twilio  

---

### 🔹 Automação de Códigos
- Rodar scripts com computador desligado  
- Agendamento diário, semanal ou mensal  
- Preparação para tarefas automatizadas com Python  

---

### 🔹 Criação de REST API com Flask
- Estrutura mínima de uma API com Flask:
```python 
from flask import Flask

app = Flask(name)

@app.route('/')
def home():
return "Hello World!"

app.run()
```

- Desenvolvimento de endpoints para manipulação de recursos  
- Criação de APIs locais para testes e projetos  

---

## 🎓 Estrutura das Aulas

Aula | Tema | Duração
---- | ---- | -------
1 | Leia antes de começar | 00:15
2 | Python e Integrações de API | 07:13
3 | API de Cotação de Moeda | 12:37
4 | Mais exercício de API com Cotação de Moeda | 11:13
5 | API com Login - Envio de SMS com Python e Twilio | 11:46
6 | Rodar códigos com computador desligado, agendar diariamente, semanalmente, mensalmente | 24:32
7 | Criação de REST API com Python | 2:20:46

---

## 🚀 Resultado
- Domínio do consumo de APIs externas com Python  
- Capacidade de manipular dados em JSON e integrá-los a sistemas  
- Entendimento completo de REST API e métodos HTTP  
- Criação de APIs próprias com Flask  
- Automação de scripts e requisições para aplicações reais
