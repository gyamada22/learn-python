# Playwright com Python

## Objetivo do curso

Este curso ensina Playwright com Python para automações semelhantes às encontradas em ambientes corporativos:

- acessar sistemas com login e senha;
- fechar pop-ups, banners de cookies e modais;
- preencher formulários;
- selecionar datas;
- trabalhar com dropdowns e campos de autocomplete;
- cadastrar e alterar registros;
- navegar por abas internas e novas janelas;
- fazer upload e download de arquivos;
- extrair tabelas para `pandas`;
- validar arquivos baixados;
- gerar logs, screenshots e evidências de erro.

## Ambientes utilizados

### OrangeHRM Demo

Usado como sistema corporativo principal:

```text
URL: https://opensource-demo.orangehrmlive.com/

Usuário: Admin
Senha: admin123
```

Principais atividades:

- login;
- cadastro de funcionários;
- pesquisa de registros;
- abertura de cadastros;
- edição de informações;
- navegação por abas;
- preenchimento de datas e dropdowns.

### Expand Testing

Usado como laboratório técnico:

```text
URL: https://practice.expandtesting.com/
```

Principais atividades:

- pop-ups;
- alertas JavaScript;
- upload;
- download;
- novas janelas;
- elementos dinâmicos;
- tratamento de erros.

> Os ambientes são públicos. Não utilize dados pessoais ou corporativos reais. Cadastre somente informações fictícias.

---

# Preparação do projeto

## Criar a pasta

No PowerShell:

```powershell
New-Item -ItemType Directory -Path "curso-playwright-dados"
Set-Location "curso-playwright-dados"
```

Comandos abreviados equivalentes:

```powershell
mkdir "curso-playwright-dados"
cd "curso-playwright-dados"
```

- `mkdir` é uma forma abreviada usada para criar diretórios.
- `cd` é um alias de `Set-Location`.

## Criar o ambiente virtual

```powershell
python -m venv .venv
```

Ativar:

```powershell
.\.venv\Scripts\Activate.ps1
```

Quando estiver ativo, o terminal deverá mostrar:

```text
(.venv)
```

## Instalar as bibliotecas

```powershell
python -m pip install --upgrade pip
pip install playwright pandas openpyxl python-dotenv
playwright install chromium
```

## Criar as pastas do projeto

```powershell
mkdir aulas
mkdir dados
mkdir downloads
mkdir evidencias
mkdir logs
```

Estrutura inicial:

```text
curso-playwright-dados/
│
├── .venv/
├── aulas/
├── dados/
├── downloads/
├── evidencias/
├── logs/
├── .env
├── .gitignore
└── requirements.txt
```

## Criar o `requirements.txt`

```text
playwright
pandas
openpyxl
python-dotenv
```

## Criar o `.env`

```env
ORANGE_USER=Admin
ORANGE_PASSWORD=admin123
```

## Criar o `.gitignore`

```gitignore
.venv/
__pycache__/
.env
downloads/
evidencias/
logs/
*.pyc
```

## Regra importante

No início do curso usaremos a API síncrona:

```python
from playwright.sync_api import sync_playwright
```

A API síncrona é mais fácil de entender e atende bem a robôs corporativos executados em sequência.

---

# Aula 1 — Navegador, login e validação de acesso

## Objetivos

Nesta aula você aprenderá a:

- iniciar o Playwright;
- abrir o Chromium;
- criar uma sessão;
- acessar uma URL;
- preencher usuário e senha;
- clicar no botão de login;
- validar se o login funcionou;
- tratar uma credencial inválida;
- fechar corretamente o navegador.

## Estrutura principal do Playwright

```text
Playwright
└── Browser
    └── BrowserContext
        └── Page
            └── Locator
```

### Browser

É o navegador aberto pelo Playwright.

### BrowserContext

É uma sessão isolada do navegador, semelhante a uma janela anônima.

Cada contexto possui seus próprios:

- cookies;
- sessão;
- armazenamento local;
- login.

### Page

Representa uma aba do navegador.

### Locator

Representa a forma usada para encontrar um elemento.

Exemplos:

```python
page.get_by_placeholder("Username")
page.get_by_role("button", name="Login")
page.get_by_text("Dashboard")
```

## Criar `aulas/aula01_login.py`

```python
import os
import re

from dotenv import load_dotenv
from playwright.sync_api import (
    Page,
    expect,
    sync_playwright,
)


URL_ORANGE = "https://opensource-demo.orangehrmlive.com/"


def fazer_login(
    page: Page,
    usuario: str,
    senha: str,
) -> None:
    page.goto(URL_ORANGE)

    page.get_by_placeholder("Username").fill(usuario)
    page.get_by_placeholder("Password").fill(senha)

    page.get_by_role(
        "button",
        name="Login",
    ).click()


def executar() -> None:
    load_dotenv()

    usuario = os.getenv("ORANGE_USER")
    senha = os.getenv("ORANGE_PASSWORD")

    if not usuario or not senha:
        raise ValueError(
            "As variáveis ORANGE_USER e ORANGE_PASSWORD "
            "não foram encontradas no arquivo .env."
        )

    with sync_playwright() as playwright:
        browser = playwright.chromium.launch(
            headless=False,
            slow_mo=300,
        )

        context = browser.new_context(
            viewport={
                "width": 1440,
                "height": 900,
            }
        )

        page = context.new_page()

        page.set_default_timeout(15_000)
        page.set_default_navigation_timeout(30_000)

        try:
            fazer_login(
                page=page,
                usuario=usuario,
                senha=senha,
            )

            expect(page).to_have_url(
                re.compile(r".*/dashboard/.*")
            )

            expect(
                page.get_by_text(
                    "Dashboard",
                    exact=True,
                )
            ).to_be_visible()

            print("Login realizado com sucesso.")
            print(f"URL atual: {page.url}")
            print(f"Título da página: {page.title()}")

        finally:
            context.close()
            browser.close()


if __name__ == "__main__":
    executar()
```

## Executar

```powershell
python .\aulas\aula01_login.py
```

## Entendendo as ações

### Preencher campos

```python
page.get_by_placeholder("Username").fill(usuario)
```

`fill()`:

1. localiza o campo;
2. aguarda o campo ficar disponível;
3. limpa o conteúdo anterior;
4. preenche o novo conteúdo.

### Clicar

```python
page.get_by_role(
    "button",
    name="Login",
).click()
```

O Playwright aguarda automaticamente o botão ficar:

- visível;
- habilitado;
- estável;
- disponível para receber o clique.

### Validar a URL

```python
expect(page).to_have_url(
    re.compile(r".*/dashboard/.*")
)
```

A validação confirma que o sistema redirecionou para o dashboard.

## Testando credenciais inválidas

Adicione temporariamente esta função:

```python
def testar_login_invalido(page: Page) -> None:
    fazer_login(
        page=page,
        usuario="usuario_invalido",
        senha="senha_invalida",
    )

    mensagem = page.get_by_text(
        "Invalid credentials",
        exact=True,
    )

    expect(mensagem).to_be_visible()

    print("O sistema recusou corretamente as credenciais.")
```

Para executá-la, substitua a chamada de login correto por:

```python
testar_login_invalido(page)
```

## Locator recomendado

Ordem preferencial:

```text
1. get_by_role()
2. get_by_label()
3. get_by_placeholder()
4. get_by_text()
5. get_by_test_id()
6. locator() com CSS
7. XPath somente quando necessário
```

Exemplo recomendado:

```python
page.get_by_role("button", name="Login")
```

Exemplo mais frágil:

```python
page.locator(
    "body > div:nth-child(1) > div > div > button"
)
```

## Exercício

Altere o script para imprimir:

- usuário utilizado;
- URL antes do login;
- URL depois do login;
- título da página;
- confirmação de que o menu `PIM` está visível.

Validação sugerida:

```python
expect(
    page.get_by_role(
        "link",
        name="PIM",
    )
).to_be_visible()
```

## Resultado esperado

```text
Login realizado com sucesso.
URL atual: .../dashboard/index
Título da página: OrangeHRM
```

---

# Aula 2 — Pop-ups, banners, modais e alertas

## Objetivos

Nesta aula você aprenderá a:

- fechar um pop-up opcional;
- aceitar um banner de cookies;
- fechar uma janela modal;
- aceitar ou cancelar alertas JavaScript;
- preencher um `prompt`;
- continuar a automação quando o pop-up não aparecer.

## Diferença entre os elementos

### Modal HTML

É um componente dentro da própria página.

Pode ser localizado normalmente:

```python
page.get_by_text("Close").click()
```

### Banner de cookies

Também faz parte da página.

Exemplo:

```python
page.get_by_role(
    "button",
    name="I Accept",
).click()
```

### Alerta JavaScript

É uma janela controlada pelo navegador.

Não pode ser tratada com um locator comum.

É necessário usar o evento:

```python
page.once("dialog", ...)
```

## Criar `aulas/aula02_popups.py`

```python
from playwright.sync_api import (
    Locator,
    Page,
    TimeoutError as PlaywrightTimeoutError,
    sync_playwright,
)


URL_COOKIES = "https://practice.expandtesting.com/cookie-alert"
URL_MODAL = "https://practice.expandtesting.com/entry-ad"
URL_DIALOGOS = "https://practice.expandtesting.com/js-dialogs"


def clicar_se_aparecer(
    locator: Locator,
    timeout: int = 3_000,
) -> bool:
    try:
        locator.wait_for(
            state="visible",
            timeout=timeout,
        )

        locator.click()

        return True

    except PlaywrightTimeoutError:
        return False


def aceitar_cookies(page: Page) -> None:
    page.goto(URL_COOKIES)

    aceitou = clicar_se_aparecer(
        page.get_by_role(
            "button",
            name="I Accept",
        )
    )

    if aceitou:
        print("Banner de cookies aceito.")
    else:
        print("O banner de cookies não apareceu.")


def fechar_modal(page: Page) -> None:
    page.goto(URL_MODAL)

    fechou = clicar_se_aparecer(
        page.get_by_text(
            "Close",
            exact=True,
        )
    )

    if fechou:
        print("Modal fechado.")
    else:
        print("O modal não apareceu.")


def testar_alert(page: Page) -> None:
    page.goto(URL_DIALOGOS)

    def tratar_alerta(dialog) -> None:
        print(f"Tipo: {dialog.type}")
        print(f"Mensagem: {dialog.message}")

        dialog.accept()

    page.once(
        "dialog",
        tratar_alerta,
    )

    page.get_by_role(
        "button",
        name="Js Alert",
    ).click()

    print("Alert aceito.")


def testar_confirm(page: Page) -> None:
    page.goto(URL_DIALOGOS)

    def cancelar_confirmacao(dialog) -> None:
        print(f"Mensagem: {dialog.message}")

        dialog.dismiss()

    page.once(
        "dialog",
        cancelar_confirmacao,
    )

    page.get_by_role(
        "button",
        name="Js Confirm",
    ).click()

    print("Confirmação cancelada.")


def testar_prompt(page: Page) -> None:
    page.goto(URL_DIALOGOS)

    def preencher_prompt(dialog) -> None:
        print(f"Mensagem: {dialog.message}")

        dialog.accept(
            "Automação Playwright"
        )

    page.once(
        "dialog",
        preencher_prompt,
    )

    page.get_by_role(
        "button",
        name="Js Prompt",
    ).click()

    print("Prompt preenchido.")


def executar() -> None:
    with sync_playwright() as playwright:
        browser = playwright.chromium.launch(
            headless=False,
            slow_mo=400,
        )

        context = browser.new_context()
        page = context.new_page()

        try:
            aceitar_cookies(page)
            fechar_modal(page)
            testar_alert(page)
            testar_confirm(page)
            testar_prompt(page)

        finally:
            context.close()
            browser.close()


if __name__ == "__main__":
    executar()
```

## Executar

```powershell
python .\aulas\aula02_popups.py
```

## Pop-up obrigatório versus opcional

### Obrigatório

Se o pop-up sempre deve aparecer:

```python
page.get_by_text("Close").click()
```

Se ele não aparecer, o robô deve falhar.

### Opcional

Se ele pode aparecer ou não:

```python
clicar_se_aparecer(
    page.get_by_text("Close")
)
```

O robô continuará mesmo que o elemento não exista.

## Não use desta forma

```python
if page.get_by_text("Close").count() > 0:
    page.get_by_text("Close").click()
```

`count()` verifica apenas o que existe naquele instante. Se o modal aparecer alguns milissegundos depois, poderá não ser encontrado.

Prefira aguardar por um curto período usando `wait_for()`.

## Handler automático para elementos bloqueadores

Em versões atuais do Playwright, também é possível registrar um handler:

```python
popup = page.get_by_text(
    "Close",
    exact=True,
)

page.add_locator_handler(
    popup,
    lambda elemento: elemento.click(),
)
```

Quando o elemento bloquear uma ação, o Playwright tentará executar o handler.

Para começar, a função `clicar_se_aparecer()` é mais fácil de compreender e controlar.

## Exercício

Crie uma função chamada:

```python
fechar_elementos_opcionais(page)
```

Ela deve tentar fechar:

- banner de cookies;
- modal;
- botão com texto `Close`;
- botão com texto `Dismiss`;
- botão com texto `Not now`.

Cada tentativa deve retornar `True` ou `False`.

---

# Aula 3 — Cadastro de funcionário e upload de imagem

## Objetivos

Nesta aula você aprenderá a:

- navegar pelo menu de um sistema;
- abrir uma tela de cadastro;
- gerar dados únicos;
- preencher vários campos;
- fazer upload de uma imagem;
- salvar o registro;
- validar se o cadastro foi concluído;
- salvar os dados criados em JSON.

## Por que gerar dados únicos?

O OrangeHRM Demo é público. Um nome como:

```text
João Silva
```

pode já existir.

Usaremos um identificador baseado na data e hora:

```text
Aluno PW20260708103045
```

Isso reduz conflitos durante os exercícios.

## Preparar uma imagem

Coloque uma imagem fictícia na pasta:

```text
dados/foto_teste.png
```

Não utilize fotografia real de funcionário.

O upload será opcional. Se a imagem não existir, o cadastro continuará sem ela.

## Criar `aulas/aula03_cadastro_funcionario.py`

```python
import json
import os
import re
from datetime import datetime
from pathlib import Path

from dotenv import load_dotenv
from playwright.sync_api import (
    Page,
    expect,
    sync_playwright,
)


URL_ORANGE = "https://opensource-demo.orangehrmlive.com/"

RAIZ_PROJETO = Path(__file__).resolve().parent.parent

CAMINHO_FOTO = (
    RAIZ_PROJETO
    / "dados"
    / "foto_teste.png"
)

CAMINHO_REGISTRO = (
    RAIZ_PROJETO
    / "dados"
    / "ultimo_funcionario.json"
)


def fazer_login(
    page: Page,
    usuario: str,
    senha: str,
) -> None:
    page.goto(URL_ORANGE)

    page.get_by_placeholder("Username").fill(usuario)
    page.get_by_placeholder("Password").fill(senha)

    page.get_by_role(
        "button",
        name="Login",
    ).click()

    expect(page).to_have_url(
        re.compile(r".*/dashboard/.*")
    )


def gerar_dados_funcionario() -> dict[str, str]:
    identificador = datetime.now().strftime(
        "%Y%m%d%H%M%S"
    )

    return {
        "primeiro_nome": "Aluno",
        "nome_meio": "Dados",
        "sobrenome": f"PW{identificador}",
        "id_funcionario": identificador[-8:],
    }


def abrir_cadastro(page: Page) -> None:
    page.get_by_role(
        "link",
        name="PIM",
    ).click()

    page.get_by_text(
        "Add Employee",
        exact=True,
    ).click()

    expect(
        page.get_by_text(
            "Add Employee",
            exact=True,
        )
    ).to_be_visible()


def preencher_dados(
    page: Page,
    dados: dict[str, str],
) -> None:
    page.get_by_placeholder(
        "First Name"
    ).fill(
        dados["primeiro_nome"]
    )

    page.get_by_placeholder(
        "Middle Name"
    ).fill(
        dados["nome_meio"]
    )

    page.get_by_placeholder(
        "Last Name"
    ).fill(
        dados["sobrenome"]
    )

    grupo_id = page.locator(
        ".oxd-input-group"
    ).filter(
        has_text="Employee Id"
    )

    campo_id = grupo_id.locator("input")

    campo_id.fill(
        dados["id_funcionario"]
    )


def enviar_foto(page: Page) -> None:
    if not CAMINHO_FOTO.exists():
        print(
            "Foto não encontrada. "
            "O cadastro continuará sem imagem."
        )
        return

    campo_upload = page.locator(
        "input[type='file']"
    )

    campo_upload.set_input_files(
        CAMINHO_FOTO
    )

    print(
        f"Foto selecionada: {CAMINHO_FOTO.name}"
    )


def salvar_funcionario(page: Page) -> None:
    page.get_by_role(
        "button",
        name="Save",
    ).click()

    expect(
        page.get_by_text(
            "Personal Details",
            exact=True,
        )
    ).to_be_visible(
        timeout=20_000
    )


def salvar_dados_em_json(
    dados: dict[str, str],
) -> None:
    CAMINHO_REGISTRO.parent.mkdir(
        parents=True,
        exist_ok=True,
    )

    CAMINHO_REGISTRO.write_text(
        json.dumps(
            dados,
            ensure_ascii=False,
            indent=4,
        ),
        encoding="utf-8",
    )


def executar() -> None:
    load_dotenv()

    usuario = os.getenv("ORANGE_USER")
    senha = os.getenv("ORANGE_PASSWORD")

    if not usuario or not senha:
        raise ValueError(
            "Credenciais não encontradas no .env."
        )

    dados = gerar_dados_funcionario()

    nome_completo = " ".join(
        [
            dados["primeiro_nome"],
            dados["nome_meio"],
            dados["sobrenome"],
        ]
    )

    print(f"Funcionário: {nome_completo}")
    print(f"ID: {dados['id_funcionario']}")

    with sync_playwright() as playwright:
        browser = playwright.chromium.launch(
            headless=False,
            slow_mo=300,
        )

        context = browser.new_context()
        page = context.new_page()

        page.set_default_timeout(15_000)

        try:
            fazer_login(
                page=page,
                usuario=usuario,
                senha=senha,
            )

            abrir_cadastro(page)

            preencher_dados(
                page=page,
                dados=dados,
            )

            enviar_foto(page)

            salvar_funcionario(page)

            salvar_dados_em_json(dados)

            print("Funcionário cadastrado.")
            print(
                f"Dados salvos em: {CAMINHO_REGISTRO}"
            )

        finally:
            context.close()
            browser.close()


if __name__ == "__main__":
    executar()
```

## Executar

```powershell
python .\aulas\aula03_cadastro_funcionario.py
```

## Upload de arquivo

O método principal é:

```python
locator.set_input_files(caminho)
```

Exemplo:

```python
page.locator(
    "input[type='file']"
).set_input_files(
    r"C:\Arquivos\foto.png"
)
```

O Playwright não precisa abrir a janela do Windows para selecionar o arquivo. Ele envia o caminho diretamente para o campo HTML.

## Por que salvar em JSON?

O próximo script precisará localizar o mesmo funcionário.

Exemplo do arquivo gerado:

```json
{
    "primeiro_nome": "Aluno",
    "nome_meio": "Dados",
    "sobrenome": "PW20260708103045",
    "id_funcionario": "08103045"
}
```

Assim, uma aula pode reutilizar os dados da aula anterior.

## Exercício

Adicione os campos abaixo ao dicionário:

```python
{
    "data_cadastro": "...",
    "origem": "curso_playwright",
    "status": "cadastrado"
}
```

A data deve ser salva em formato ISO:

```text
2026-07-08T10:30:45
```

Use:

```python
datetime.now().isoformat(
    timespec="seconds"
)
```

---

# Aula 4 — Pesquisa, abas internas, datas, dropdowns e autocomplete

## Objetivos

Nesta aula você aprenderá a:

- pesquisar um registro cadastrado;
- usar um campo de autocomplete;
- abrir o registro correto;
- navegar por abas internas;
- preencher datas;
- trabalhar com dropdown customizado;
- salvar alterações;
- validar mensagens de sucesso.

## Abas internas versus abas do navegador

No OrangeHRM, itens como:

```text
Personal Details
Contact Details
Job
Salary
Report-to
```

são abas internas do sistema.

Elas não criam outra `Page`.

Continuamos utilizando:

```python
page
```

## Carregar o funcionário da aula anterior

O script lerá:

```text
dados/ultimo_funcionario.json
```

## Criar `aulas/aula04_editar_funcionario.py`

```python
import json
import os
import re
from pathlib import Path

from dotenv import load_dotenv
from playwright.sync_api import (
    Locator,
    Page,
    expect,
    sync_playwright,
)


URL_ORANGE = "https://opensource-demo.orangehrmlive.com/"

RAIZ_PROJETO = Path(__file__).resolve().parent.parent

CAMINHO_REGISTRO = (
    RAIZ_PROJETO
    / "dados"
    / "ultimo_funcionario.json"
)


def carregar_funcionario() -> dict[str, str]:
    if not CAMINHO_REGISTRO.exists():
        raise FileNotFoundError(
            "Execute primeiro a Aula 3 para gerar "
            "o arquivo ultimo_funcionario.json."
        )

    return json.loads(
        CAMINHO_REGISTRO.read_text(
            encoding="utf-8"
        )
    )


def fazer_login(
    page: Page,
    usuario: str,
    senha: str,
) -> None:
    page.goto(URL_ORANGE)

    page.get_by_placeholder("Username").fill(usuario)
    page.get_by_placeholder("Password").fill(senha)

    page.get_by_role(
        "button",
        name="Login",
    ).click()

    expect(page).to_have_url(
        re.compile(r".*/dashboard/.*")
    )


def localizar_grupo(
    page: Page,
    texto_rotulo: str,
) -> Locator:
    return page.locator(
        ".oxd-input-group"
    ).filter(
        has_text=re.compile(
            texto_rotulo,
            re.IGNORECASE,
        )
    )


def pesquisar_por_id(
    page: Page,
    id_funcionario: str,
) -> None:
    page.get_by_role(
        "link",
        name="PIM",
    ).click()

    page.get_by_text(
        "Employee List",
        exact=True,
    ).click()

    grupo_id = localizar_grupo(
        page,
        "Employee Id",
    )

    grupo_id.locator("input").fill(
        id_funcionario
    )

    page.get_by_role(
        "button",
        name="Search",
    ).click()


def abrir_resultado(
    page: Page,
    id_funcionario: str,
) -> None:
    linha = page.locator(
        ".oxd-table-card"
    ).filter(
        has_text=id_funcionario
    ).first

    expect(linha).to_be_visible(
        timeout=15_000
    )

    linha.get_by_text(
        id_funcionario,
        exact=True,
    ).click()

    expect(
        page.get_by_text(
            "Personal Details",
            exact=True,
        )
    ).to_be_visible()


def preencher_data_nascimento(page: Page) -> None:
    grupo_data = localizar_grupo(
        page,
        "Date of Birth",
    )

    campo_data = grupo_data.locator("input")

    campo_data.fill("1995-06-15")


def selecionar_dropdown_customizado(
    page: Page,
    rotulo: str,
    opcao: str,
) -> None:
    grupo = localizar_grupo(
        page,
        rotulo,
    )

    grupo.locator(
        ".oxd-select-text"
    ).click()

    item = page.get_by_role(
        "option",
        name=opcao,
    )

    expect(item).to_be_visible()

    item.click()


def salvar_detalhes_pessoais(page: Page) -> None:
    botoes_salvar = page.get_by_role(
        "button",
        name="Save",
    )

    botoes_salvar.first.click()

    mensagem = page.get_by_text(
        "Successfully Updated",
        exact=True,
    )

    expect(mensagem).to_be_visible(
        timeout=10_000
    )


def abrir_aba_job(page: Page) -> None:
    page.get_by_role(
        "link",
        name="Job",
    ).click()

    expect(
        page.get_by_text(
            "Job Details",
            exact=True,
        )
    ).to_be_visible()


def executar() -> None:
    load_dotenv()

    usuario = os.getenv("ORANGE_USER")
    senha = os.getenv("ORANGE_PASSWORD")

    if not usuario or not senha:
        raise ValueError(
            "Credenciais não encontradas."
        )

    dados = carregar_funcionario()

    with sync_playwright() as playwright:
        browser = playwright.chromium.launch(
            headless=False,
            slow_mo=300,
        )

        context = browser.new_context()
        page = context.new_page()

        page.set_default_timeout(15_000)

        try:
            fazer_login(
                page=page,
                usuario=usuario,
                senha=senha,
            )

            pesquisar_por_id(
                page=page,
                id_funcionario=dados[
                    "id_funcionario"
                ],
            )

            abrir_resultado(
                page=page,
                id_funcionario=dados[
                    "id_funcionario"
                ],
            )

            preencher_data_nascimento(page)

            selecionar_dropdown_customizado(
                page=page,
                rotulo="Nationality",
                opcao="Brazilian",
            )

            salvar_detalhes_pessoais(page)

            abrir_aba_job(page)

            print(
                "Registro localizado e atualizado."
            )

        finally:
            context.close()
            browser.close()


if __name__ == "__main__":
    executar()
```

## Datas

Campos HTML de data normalmente aceitam:

```python
campo.fill("2026-07-08")
```

Formato recomendado:

```text
AAAA-MM-DD
```

Exemplos:

```text
2026-01-31
2026-06-15
2026-12-01
```

Quando um sistema utiliza um calendário personalizado, existem duas estratégias.

### Estratégia 1 — preencher diretamente

```python
campo.fill("2026-07-08")
```

É a opção mais simples e estável.

### Estratégia 2 — clicar no calendário

```python
campo.click()
page.get_by_text("8", exact=True).click()
```

Essa abordagem pode ser mais frágil quando existem vários números iguais na tela.

Prefira preencher o campo diretamente quando o sistema permitir.

## Dropdown HTML tradicional

Quando o HTML utiliza `<select>`:

```python
page.get_by_label(
    "Unidade"
).select_option(
    label="Nefroclínicas BH"
)
```

## Dropdown customizado

Sistemas modernos frequentemente usam componentes feitos com `div`.

Nesse caso:

```python
grupo.locator(
    ".oxd-select-text"
).click()

page.get_by_role(
    "option",
    name="Brazilian",
).click()
```

## Autocomplete

O padrão de um autocomplete é:

```python
campo.fill("Aluno Dados")

opcao = page.get_by_role(
    "option"
).filter(
    has_text="Aluno Dados"
).first

expect(opcao).to_be_visible()

opcao.click()
```

Não basta preencher o texto. Em muitos sistemas é obrigatório selecionar uma opção da lista.

## Exercício

Após abrir o funcionário:

1. entre na aba `Contact Details`;
2. localize o campo `City`;
3. preencha `Belo Horizonte`;
4. preencha `State/Province` com `Minas Gerais`;
5. salve;
6. valide a mensagem de sucesso.

---

# Aula 5 — Tabelas, filtros, pandas e exportação

## Objetivos

Nesta aula você aprenderá a:

- pesquisar registros em uma tabela;
- contar resultados;
- identificar a linha correta;
- ler cabeçalhos e células;
- transformar dados em lista;
- criar um `DataFrame`;
- salvar CSV e Excel;
- validar a quantidade de registros.

## Contexto de dados

Em uma automação corporativa, normalmente não basta clicar.

Também precisamos:

- capturar o resultado;
- conferir se a pesquisa retornou dados;
- armazenar as informações;
- comparar com outra fonte;
- gerar um arquivo estruturado.

## Criar `aulas/aula05_tabelas_pandas.py`

```python
import os
import re
from datetime import datetime
from pathlib import Path

import pandas as pd
from dotenv import load_dotenv
from playwright.sync_api import (
    Page,
    expect,
    sync_playwright,
)


URL_ORANGE = "https://opensource-demo.orangehrmlive.com/"

RAIZ_PROJETO = Path(__file__).resolve().parent.parent

PASTA_DADOS = RAIZ_PROJETO / "dados"


def fazer_login(
    page: Page,
    usuario: str,
    senha: str,
) -> None:
    page.goto(URL_ORANGE)

    page.get_by_placeholder("Username").fill(usuario)
    page.get_by_placeholder("Password").fill(senha)

    page.get_by_role(
        "button",
        name="Login",
    ).click()

    expect(page).to_have_url(
        re.compile(r".*/dashboard/.*")
    )


def abrir_lista_funcionarios(page: Page) -> None:
    page.get_by_role(
        "link",
        name="PIM",
    ).click()

    page.get_by_text(
        "Employee List",
        exact=True,
    ).click()

    expect(
        page.get_by_text(
            "Employee Information",
            exact=True,
        )
    ).to_be_visible()


def aguardar_tabela(page: Page) -> None:
    corpo = page.locator(
        ".oxd-table-body"
    )

    expect(corpo).to_be_visible()


def extrair_cabecalhos(page: Page) -> list[str]:
    textos = page.locator(
        ".oxd-table-header "
        ".oxd-table-cell"
    ).all_inner_texts()

    cabecalhos = []

    for indice, texto in enumerate(
        textos,
        start=1,
    ):
        texto_limpo = texto.strip()

        if not texto_limpo:
            texto_limpo = f"coluna_{indice}"

        cabecalhos.append(texto_limpo)

    return cabecalhos


def extrair_linhas(
    page: Page,
) -> list[list[str]]:
    linhas = page.locator(
        ".oxd-table-card"
    )

    quantidade = linhas.count()

    registros = []

    for indice in range(quantidade):
        linha = linhas.nth(indice)

        celulas = linha.locator(
            ".oxd-table-cell"
        ).all_inner_texts()

        valores = [
            valor.strip()
            for valor in celulas
        ]

        registros.append(valores)

    return registros


def ajustar_colunas(
    cabecalhos: list[str],
    registros: list[list[str]],
) -> list[str]:
    maior_quantidade = max(
        [len(cabecalhos)]
        + [len(linha) for linha in registros],
        default=0,
    )

    colunas = cabecalhos.copy()

    while len(colunas) < maior_quantidade:
        colunas.append(
            f"coluna_{len(colunas) + 1}"
        )

    return colunas


def ajustar_linhas(
    registros: list[list[str]],
    quantidade_colunas: int,
) -> list[list[str]]:
    resultado = []

    for linha in registros:
        linha_ajustada = linha.copy()

        while (
            len(linha_ajustada)
            < quantidade_colunas
        ):
            linha_ajustada.append("")

        resultado.append(
            linha_ajustada[
                :quantidade_colunas
            ]
        )

    return resultado


def salvar_resultado(
    dataframe: pd.DataFrame,
) -> None:
    PASTA_DADOS.mkdir(
        parents=True,
        exist_ok=True,
    )

    momento = datetime.now().strftime(
        "%Y%m%d_%H%M%S"
    )

    caminho_csv = (
        PASTA_DADOS
        / f"funcionarios_{momento}.csv"
    )

    caminho_excel = (
        PASTA_DADOS
        / f"funcionarios_{momento}.xlsx"
    )

    dataframe.to_csv(
        caminho_csv,
        index=False,
        encoding="utf-8-sig",
    )

    dataframe.to_excel(
        caminho_excel,
        index=False,
    )

    print(f"CSV: {caminho_csv}")
    print(f"Excel: {caminho_excel}")


def executar() -> None:
    load_dotenv()

    usuario = os.getenv("ORANGE_USER")
    senha = os.getenv("ORANGE_PASSWORD")

    if not usuario or not senha:
        raise ValueError(
            "Credenciais não encontradas."
        )

    with sync_playwright() as playwright:
        browser = playwright.chromium.launch(
            headless=False,
            slow_mo=200,
        )

        context = browser.new_context()
        page = context.new_page()

        page.set_default_timeout(15_000)

        try:
            fazer_login(
                page=page,
                usuario=usuario,
                senha=senha,
            )

            abrir_lista_funcionarios(page)
            aguardar_tabela(page)

            cabecalhos = extrair_cabecalhos(
                page
            )

            registros = extrair_linhas(
                page
            )

            colunas = ajustar_colunas(
                cabecalhos=cabecalhos,
                registros=registros,
            )

            linhas_ajustadas = ajustar_linhas(
                registros=registros,
                quantidade_colunas=len(
                    colunas
                ),
            )

            df = pd.DataFrame(
                linhas_ajustadas,
                columns=colunas,
            )

            print(df.head())
            print(
                f"Quantidade extraída: {len(df)}"
            )

            if df.empty:
                raise ValueError(
                    "Nenhum registro foi extraído."
                )

            salvar_resultado(df)

        finally:
            context.close()
            browser.close()


if __name__ == "__main__":
    executar()
```

## Executar

```powershell
python .\aulas\aula05_tabelas_pandas.py
```

## Encontrar uma linha específica

Exemplo:

```python
linha = page.locator(
    ".oxd-table-card"
).filter(
    has_text="08103045"
).first
```

Validar:

```python
expect(linha).to_be_visible()
```

Clicar no botão de edição da linha:

```python
linha.get_by_role(
    "button"
).last.click()
```

Quando houver vários botões, o ideal é inspecionar seus atributos ou usar um locator mais específico.

## Limitação desta aula

O script extrai apenas os registros visíveis na página atual.

Uma tabela com paginação pode possuir mais registros.

Estratégias posteriores:

```text
1. aumentar a quantidade de linhas exibidas;
2. percorrer cada página;
3. capturar os dados de uma API interna;
4. utilizar o botão oficial de exportação, quando existir.
```

## Exercício

Adicione ao script:

- remoção de linhas completamente vazias;
- remoção de espaços duplicados;
- coluna `data_extracao`;
- coluna `origem`;
- validação de duplicidade.

Exemplo:

```python
df["data_extracao"] = datetime.now()
df["origem"] = "OrangeHRM"

duplicados = df.duplicated().sum()

print(f"Duplicados: {duplicados}")
```

---

# Aula 6 — Upload, download e validação de arquivos

## Objetivos

Nesta aula você aprenderá a:

- criar um arquivo de teste com pandas;
- fazer upload;
- validar se o upload funcionou;
- capturar um download;
- definir a pasta de destino;
- manter o arquivo depois de fechar o navegador;
- validar extensão e tamanho;
- ler o conteúdo baixado.

## Regra do download

O evento deve ser preparado antes do clique:

```python
with page.expect_download() as download_info:
    botao.click()

download = download_info.value
```

Depois:

```python
download.save_as(caminho_final)
```

Sem `save_as()`, o arquivo poderá permanecer apenas em uma pasta temporária.

## Criar `aulas/aula06_arquivos.py`

```python
import json
from pathlib import Path

import pandas as pd
from playwright.sync_api import (
    Page,
    expect,
    sync_playwright,
)


URL_UPLOAD = "https://practice.expandtesting.com/upload"
URL_DOWNLOAD = "https://practice.expandtesting.com/download"

RAIZ_PROJETO = Path(__file__).resolve().parent.parent

PASTA_DADOS = RAIZ_PROJETO / "dados"
PASTA_DOWNLOADS = RAIZ_PROJETO / "downloads"

ARQUIVO_UPLOAD = (
    PASTA_DADOS
    / "colaboradores_teste.csv"
)


def criar_csv_teste() -> None:
    PASTA_DADOS.mkdir(
        parents=True,
        exist_ok=True,
    )

    dados = [
        {
            "id": 1,
            "nome": "Pessoa Teste 1",
            "unidade": "NCBHZ",
            "status": "Ativo",
        },
        {
            "id": 2,
            "nome": "Pessoa Teste 2",
            "unidade": "NCBSB",
            "status": "Ativo",
        },
        {
            "id": 3,
            "nome": "Pessoa Teste 3",
            "unidade": "NCIPA",
            "status": "Inativo",
        },
    ]

    df = pd.DataFrame(dados)

    df.to_csv(
        ARQUIVO_UPLOAD,
        index=False,
        encoding="utf-8-sig",
    )

    print(
        f"Arquivo criado: {ARQUIVO_UPLOAD}"
    )


def fazer_upload(page: Page) -> None:
    page.goto(URL_UPLOAD)

    campo_arquivo = page.locator(
        "input[type='file']"
    )

    campo_arquivo.set_input_files(
        ARQUIVO_UPLOAD
    )

    page.get_by_role(
        "button",
        name="Upload",
    ).click()

    expect(
        page.get_by_text(
            ARQUIVO_UPLOAD.name,
            exact=False,
        )
    ).to_be_visible(
        timeout=10_000
    )

    print("Upload concluído.")


def baixar_json(page: Page) -> Path:
    PASTA_DOWNLOADS.mkdir(
        parents=True,
        exist_ok=True,
    )

    page.goto(URL_DOWNLOAD)

    link = page.get_by_role(
        "link",
        name="some-file.json",
        exact=True,
    )

    with page.expect_download() as download_info:
        link.click()

    download = download_info.value

    caminho_final = (
        PASTA_DOWNLOADS
        / download.suggested_filename
    )

    download.save_as(
        caminho_final
    )

    erro_download = download.failure()

    if erro_download:
        raise RuntimeError(
            f"Falha no download: {erro_download}"
        )

    print(
        f"Download salvo em: {caminho_final}"
    )

    return caminho_final


def validar_arquivo(caminho: Path) -> None:
    if not caminho.exists():
        raise FileNotFoundError(
            f"Arquivo não encontrado: {caminho}"
        )

    tamanho = caminho.stat().st_size

    if tamanho == 0:
        raise ValueError(
            "O arquivo foi baixado, "
            "mas está vazio."
        )

    print(f"Extensão: {caminho.suffix}")
    print(f"Tamanho: {tamanho} bytes")

    if caminho.suffix.lower() == ".json":
        conteudo = json.loads(
            caminho.read_text(
                encoding="utf-8"
            )
        )

        print("Conteúdo JSON:")
        print(conteudo)

    elif caminho.suffix.lower() == ".csv":
        df = pd.read_csv(caminho)

        print(df.head())
        print(
            f"Linhas no CSV: {len(df)}"
        )

    elif caminho.suffix.lower() in {
        ".xlsx",
        ".xls",
    }:
        df = pd.read_excel(caminho)

        print(df.head())
        print(
            f"Linhas no Excel: {len(df)}"
        )

    else:
        print(
            "Não existe uma validação "
            "específica para essa extensão."
        )


def executar() -> None:
    criar_csv_teste()

    with sync_playwright() as playwright:
        browser = playwright.chromium.launch(
            headless=False,
            slow_mo=300,
        )

        context = browser.new_context()
        page = context.new_page()

        try:
            fazer_upload(page)

            arquivo_baixado = baixar_json(
                page
            )

            validar_arquivo(
                arquivo_baixado
            )

        finally:
            context.close()
            browser.close()


if __name__ == "__main__":
    executar()
```

## Executar

```powershell
python .\aulas\aula06_arquivos.py
```

## Nome sugerido versus nome final

Nome sugerido pelo site:

```python
download.suggested_filename
```

Nome personalizado:

```python
caminho_final = (
    PASTA_DOWNLOADS
    / "resultado_processamento.json"
)
```

Salvar:

```python
download.save_as(caminho_final)
```

## Downloads iniciados depois de filtros

Em sistemas corporativos, o fluxo costuma ser:

```python
page.get_by_label(
    "Data inicial"
).fill(
    "2026-06-01"
)

page.get_by_label(
    "Data final"
).fill(
    "2026-06-30"
)

page.get_by_role(
    "button",
    name="Processar",
).click()

expect(
    page.get_by_text("Concluído")
).to_be_visible()

with page.expect_download() as download_info:
    page.get_by_role(
        "button",
        name="Exportar",
    ).click()

download = download_info.value
download.save_as(caminho_final)
```

A ordem é importante:

```text
1. preencher filtros;
2. processar;
3. esperar a conclusão;
4. iniciar a captura do download;
5. clicar em exportar;
6. salvar o arquivo;
7. validar o conteúdo.
```

## Exercício

Altere o script para baixar:

```text
some-file.txt
```

Depois:

1. leia o arquivo com `read_text()`;
2. confirme que não está vazio;
3. salve uma cópia com outro nome;
4. registre o tamanho em um CSV de controle.

---

# Aula 7 — Novas abas, janelas e troca de contexto

## Objetivos

Nesta aula você aprenderá a:

- identificar quando um clique abre outra aba;
- capturar a nova `Page`;
- aguardar seu carregamento;
- executar ações na página correta;
- voltar à página principal;
- fechar somente a aba secundária;
- diferenciar abas internas de abas do navegador.

## Quando usar `expect_popup()`

Use quando um clique abrir:

- nova aba;
- nova janela;
- popup do navegador.

Padrão:

```python
with page.expect_popup() as popup_info:
    elemento.click()

nova_pagina = popup_info.value
```

## Criar `aulas/aula07_abas_janelas.py`

```python
from playwright.sync_api import (
    Page,
    expect,
    sync_playwright,
)


URL_JANELAS = "https://practice.expandtesting.com/windows"


def abrir_nova_janela(
    pagina_principal: Page,
) -> Page:
    with pagina_principal.expect_popup() as popup_info:
        pagina_principal.get_by_role(
            "link",
            name="Click Here",
        ).click()

    nova_pagina = popup_info.value

    nova_pagina.wait_for_load_state(
        "domcontentloaded"
    )

    return nova_pagina


def exibir_paginas(context) -> None:
    print(
        f"Quantidade de páginas: "
        f"{len(context.pages)}"
    )

    for indice, pagina in enumerate(
        context.pages,
        start=1,
    ):
        print(
            f"Página {indice}: {pagina.url}"
        )


def executar() -> None:
    with sync_playwright() as playwright:
        browser = playwright.chromium.launch(
            headless=False,
            slow_mo=500,
        )

        context = browser.new_context()

        pagina_principal = context.new_page()

        try:
            pagina_principal.goto(
                URL_JANELAS
            )

            expect(
                pagina_principal.get_by_role(
                    "heading",
                    name="Opening a new window",
                )
            ).to_be_visible()

            nova_pagina = abrir_nova_janela(
                pagina_principal
            )

            print(
                "URL da página principal:"
            )
            print(pagina_principal.url)

            print("URL da nova página:")
            print(nova_pagina.url)

            print("Título da nova página:")
            print(nova_pagina.title())

            exibir_paginas(context)

            nova_pagina.bring_to_front()

            expect(
                nova_pagina.locator("body")
            ).to_be_visible()

            print(
                "Texto da nova página:"
            )
            print(
                nova_pagina.locator(
                    "body"
                ).inner_text()
            )

            nova_pagina.close()

            pagina_principal.bring_to_front()

            print(
                "A aba secundária foi fechada."
            )

            print(
                f"Abas restantes: "
                f"{len(context.pages)}"
            )

        finally:
            context.close()
            browser.close()


if __name__ == "__main__":
    executar()
```

## Executar

```powershell
python .\aulas\aula07_abas_janelas.py
```

## Página principal e página secundária

```python
pagina_principal = context.new_page()
```

Depois do popup:

```python
nova_pagina = popup_info.value
```

As duas variáveis representam páginas diferentes.

Ação na página principal:

```python
pagina_principal.get_by_text(
    "Relatórios"
).click()
```

Ação na nova página:

```python
nova_pagina.get_by_text(
    "Exportar"
).click()
```

## Listar todas as páginas

```python
for pagina in context.pages:
    print(pagina.url)
```

## Alternar visualmente

```python
nova_pagina.bring_to_front()
```

Isso coloca a página na frente quando o navegador está visível.

Não é obrigatório para o Playwright interagir com ela, mas ajuda durante a depuração.

## Fechar apenas uma aba

```python
nova_pagina.close()
```

Não use:

```python
browser.close()
```

se ainda precisar continuar na página principal.

## Caso real: abrir relatório e baixar

```python
with page.expect_popup() as popup_info:
    page.get_by_role(
        "button",
        name="Abrir relatório",
    ).click()

pagina_relatorio = popup_info.value

pagina_relatorio.wait_for_load_state(
    "domcontentloaded"
)

pagina_relatorio.get_by_label(
    "Data inicial"
).fill(
    "2026-06-01"
)

pagina_relatorio.get_by_label(
    "Data final"
).fill(
    "2026-06-30"
)

with pagina_relatorio.expect_download() as download_info:
    pagina_relatorio.get_by_role(
        "button",
        name="Exportar",
    ).click()

download = download_info.value

download.save_as(
    r"C:\Relatorios\resultado.xlsx"
)

pagina_relatorio.close()
```

## Abas internas do sistema

Quando um sistema possui:

```text
Dados pessoais
Contato
Cargo
Salário
Documentos
```

mas a URL continua na mesma página ou apenas muda de rota, normalmente são abas internas.

Nesse caso:

```python
page.get_by_role(
    "tab",
    name="Contato",
).click()
```

ou:

```python
page.get_by_role(
    "link",
    name="Contact Details",
).click()
```

Não use `expect_popup()` se não houver uma nova aba real.

## Exercício

Modifique o script para:

1. abrir duas novas janelas;
2. guardar cada uma em uma variável;
3. imprimir as URLs;
4. fechar somente a primeira;
5. executar uma ação na segunda;
6. voltar para a principal;
7. confirmar quantas páginas continuam abertas.

---

# Aula 8 — Erros, logs, screenshots, trace e automação robusta

## Objetivos

Nesta aula você aprenderá a:

- tratar `TimeoutError`;
- capturar screenshots quando ocorrer falha;
- criar logs;
- registrar erros do navegador;
- criar trace de execução;
- repetir ações de maneira controlada;
- fechar recursos mesmo quando ocorrer erro;
- gerar um resumo estruturado da execução.

## Tipos comuns de falha

```text
Elemento não encontrado
Elemento demorou para aparecer
Login expirado
Popup bloqueando a tela
Download não iniciado
Arquivo vazio
Registro não encontrado
Site indisponível
Locator encontrou mais de um elemento
Nova aba não abriu
```

## Estrutura de uma automação robusta

```text
1. preparar pastas;
2. iniciar log;
3. iniciar navegador;
4. iniciar trace;
5. executar etapas;
6. validar cada resultado;
7. capturar evidência em caso de erro;
8. salvar resumo;
9. finalizar trace;
10. fechar contexto e navegador.
```

## Criar `aulas/aula08_robustez.py`

```python
import logging
import os
import re
from datetime import datetime
from pathlib import Path
from typing import Callable

import pandas as pd
from dotenv import load_dotenv
from playwright.sync_api import (
    Locator,
    Page,
    TimeoutError as PlaywrightTimeoutError,
    expect,
    sync_playwright,
)


URL_ORANGE = "https://opensource-demo.orangehrmlive.com/"

RAIZ_PROJETO = Path(__file__).resolve().parent.parent

PASTA_LOGS = RAIZ_PROJETO / "logs"
PASTA_EVIDENCIAS = RAIZ_PROJETO / "evidencias"

MOMENTO_EXECUCAO = datetime.now().strftime(
    "%Y%m%d_%H%M%S"
)

CAMINHO_LOG = (
    PASTA_LOGS
    / f"execucao_{MOMENTO_EXECUCAO}.log"
)

CAMINHO_RESULTADO = (
    PASTA_LOGS
    / f"resultado_{MOMENTO_EXECUCAO}.xlsx"
)

CAMINHO_TRACE = (
    PASTA_EVIDENCIAS
    / f"trace_{MOMENTO_EXECUCAO}.zip"
)


def preparar_pastas() -> None:
    PASTA_LOGS.mkdir(
        parents=True,
        exist_ok=True,
    )

    PASTA_EVIDENCIAS.mkdir(
        parents=True,
        exist_ok=True,
    )


def configurar_log() -> None:
    logging.basicConfig(
        level=logging.INFO,
        format=(
            "%(asctime)s | "
            "%(levelname)s | "
            "%(message)s"
        ),
        handlers=[
            logging.FileHandler(
                CAMINHO_LOG,
                encoding="utf-8",
            ),
            logging.StreamHandler(),
        ],
    )


def registrar_resultado(
    resultados: list[dict],
    etapa: str,
    status: str,
    mensagem: str,
) -> None:
    resultados.append(
        {
            "data_hora": datetime.now(),
            "etapa": etapa,
            "status": status,
            "mensagem": mensagem,
        }
    )


def salvar_resultados(
    resultados: list[dict],
) -> None:
    df = pd.DataFrame(resultados)

    df.to_excel(
        CAMINHO_RESULTADO,
        index=False,
    )


def capturar_evidencia(
    page: Page,
    nome: str,
) -> Path:
    caminho = (
        PASTA_EVIDENCIAS
        / f"{MOMENTO_EXECUCAO}_{nome}.png"
    )

    page.screenshot(
        path=caminho,
        full_page=True,
    )

    logging.info(
        "Screenshot salvo em: %s",
        caminho,
    )

    return caminho


def clicar_se_aparecer(
    locator: Locator,
    timeout: int = 2_000,
) -> bool:
    try:
        locator.wait_for(
            state="visible",
            timeout=timeout,
        )

        locator.click()

        return True

    except PlaywrightTimeoutError:
        return False


def tentar_acao(
    descricao: str,
    acao: Callable[[], None],
    tentativas: int = 3,
) -> None:
    ultimo_erro = None

    for numero in range(
        1,
        tentativas + 1,
    ):
        try:
            logging.info(
                "%s — tentativa %s de %s",
                descricao,
                numero,
                tentativas,
            )

            acao()

            logging.info(
                "%s — concluída",
                descricao,
            )

            return

        except PlaywrightTimeoutError as erro:
            ultimo_erro = erro

            logging.warning(
                "%s — timeout na tentativa %s",
                descricao,
                numero,
            )

    raise RuntimeError(
        f"A ação '{descricao}' falhou "
        f"após {tentativas} tentativas."
    ) from ultimo_erro


def registrar_eventos_navegador(
    page: Page,
) -> None:
    page.on(
        "console",
        lambda mensagem: logging.info(
            "Console [%s]: %s",
            mensagem.type,
            mensagem.text,
        ),
    )

    page.on(
        "pageerror",
        lambda erro: logging.error(
            "Erro JavaScript: %s",
            erro,
        ),
    )


def fazer_login(
    page: Page,
    usuario: str,
    senha: str,
) -> None:
    page.goto(
        URL_ORANGE,
        wait_until="domcontentloaded",
    )

    page.get_by_placeholder(
        "Username"
    ).fill(
        usuario
    )

    page.get_by_placeholder(
        "Password"
    ).fill(
        senha
    )

    page.get_by_role(
        "button",
        name="Login",
    ).click()

    expect(page).to_have_url(
        re.compile(r".*/dashboard/.*"),
        timeout=20_000,
    )


def validar_dashboard(page: Page) -> None:
    painel = page.get_by_text(
        "Dashboard",
        exact=True,
    )

    expect(painel).to_be_visible()


def executar() -> None:
    preparar_pastas()
    configurar_log()
    load_dotenv()

    resultados: list[dict] = []

    usuario = os.getenv("ORANGE_USER")
    senha = os.getenv("ORANGE_PASSWORD")

    if not usuario or not senha:
        raise ValueError(
            "Credenciais não encontradas no .env."
        )

    logging.info(
        "Iniciando automação."
    )

    with sync_playwright() as playwright:
        browser = playwright.chromium.launch(
            headless=False,
            slow_mo=200,
        )

        context = browser.new_context(
            viewport={
                "width": 1440,
                "height": 900,
            }
        )

        context.tracing.start(
            screenshots=True,
            snapshots=True,
            sources=True,
        )

        page = context.new_page()

        page.set_default_timeout(15_000)
        page.set_default_navigation_timeout(
            30_000
        )

        registrar_eventos_navegador(page)

        try:
            tentar_acao(
                descricao="Realizar login",
                acao=lambda: fazer_login(
                    page=page,
                    usuario=usuario,
                    senha=senha,
                ),
                tentativas=2,
            )

            registrar_resultado(
                resultados=resultados,
                etapa="Login",
                status="Sucesso",
                mensagem=(
                    "Login realizado corretamente."
                ),
            )

            validar_dashboard(page)

            registrar_resultado(
                resultados=resultados,
                etapa="Dashboard",
                status="Sucesso",
                mensagem=(
                    "Dashboard encontrado."
                ),
            )

            page.get_by_role(
                "link",
                name="PIM",
            ).click()

            expect(
                page.get_by_text(
                    "Employee Information",
                    exact=True,
                )
            ).to_be_visible()

            registrar_resultado(
                resultados=resultados,
                etapa="Acessar PIM",
                status="Sucesso",
                mensagem=(
                    "Tela de funcionários carregada."
                ),
            )

            logging.info(
                "Automação concluída com sucesso."
            )

        except PlaywrightTimeoutError as erro:
            logging.exception(
                "Tempo limite excedido."
            )

            evidencia = capturar_evidencia(
                page,
                "timeout",
            )

            registrar_resultado(
                resultados=resultados,
                etapa="Execução",
                status="Erro",
                mensagem=(
                    f"Timeout: {erro}. "
                    f"Evidência: {evidencia}"
                ),
            )

            raise

        except Exception as erro:
            logging.exception(
                "Erro inesperado."
            )

            evidencia = capturar_evidencia(
                page,
                "erro_geral",
            )

            registrar_resultado(
                resultados=resultados,
                etapa="Execução",
                status="Erro",
                mensagem=(
                    f"{type(erro).__name__}: "
                    f"{erro}. "
                    f"Evidência: {evidencia}"
                ),
            )

            raise

        finally:
            salvar_resultados(
                resultados
            )

            context.tracing.stop(
                path=CAMINHO_TRACE
            )

            context.close()
            browser.close()

            logging.info(
                "Navegador encerrado."
            )

            logging.info(
                "Log: %s",
                CAMINHO_LOG,
            )

            logging.info(
                "Resultado: %s",
                CAMINHO_RESULTADO,
            )

            logging.info(
                "Trace: %s",
                CAMINHO_TRACE,
            )


if __name__ == "__main__":
    executar()
```

## Executar

```powershell
python .\aulas\aula08_robustez.py
```

## Abrir o Trace Viewer

Depois da execução:

```powershell
playwright show-trace ".\evidencias\trace_XXXXXXXX_XXXXXX.zip"
```

Substitua pelo nome real do arquivo.

O trace permite analisar:

- cada ação executada;
- locator utilizado;
- screenshot antes e depois;
- HTML da página;
- tempo gasto;
- requisições;
- mensagens de erro.

## Quando repetir uma ação

Uma repetição é aceitável para ações como:

```text
abrir página;
consultar registro;
aguardar processamento;
ler tabela;
baixar relatório sem alterar dados.
```

Tenha cuidado ao repetir:

```text
cadastrar pessoa;
excluir registro;
efetuar pagamento;
enviar formulário;
confirmar lançamento.
```

Essas ações podem gerar duplicidade.

Antes de repetir um cadastro, verifique se o registro já foi criado.

## Não use repetição para esconder um locator ruim

Ruim:

```python
for _ in range(10):
    try:
        page.locator(
            "div:nth-child(5)"
        ).click()
        break
    except Exception:
        pass
```

Melhor:

```python
page.get_by_role(
    "button",
    name="Salvar",
).click()
```

## Screenshot somente em erro

```python
except Exception:
    page.screenshot(
        path="erro.png",
        full_page=True,
    )
    raise
```

Isso evita criar milhares de imagens em execuções normais.

## Resultado final da execução

O script gera:

```text
logs/
├── execucao_20260708_103045.log
└── resultado_20260708_103045.xlsx

evidencias/
├── trace_20260708_103045.zip
└── 20260708_103045_erro_geral.png
```

## Exercício final da Aula 8

Crie um script que:

1. leia `dados/ultimo_funcionario.json`;
2. faça login;
3. acesse o PIM;
4. pesquise o funcionário;
5. registre se ele foi encontrado;
6. tire screenshot somente se não for encontrado;
7. salve o resultado em Excel;
8. gere um trace;
9. encerre o navegador no `finally`.

---

# Resumo das oito aulas

## Aula 1

```text
Navegador
Contexto
Página
Locators
Login
Validação
```

## Aula 2

```text
Cookies
Modal
Popup opcional
Alert
Confirm
Prompt
```

## Aula 3

```text
Cadastro
Dados únicos
Formulários
Upload
JSON de controle
```

## Aula 4

```text
Pesquisa
Autocomplete
Abas internas
Datas
Dropdowns
Alteração de registro
```

## Aula 5

```text
Tabelas
Filtros
Extração
Pandas
CSV
Excel
```

## Aula 6

```text
Upload
Download
Pasta de destino
Validação de arquivo
Leitura com pandas
```

## Aula 7

```text
Nova aba
Nova janela
Popup
Troca de página
Fechamento de abas
```

## Aula 8

```text
Timeout
Logs
Screenshots
Trace
Tratamento de erros
Tentativas controladas
Resultado estruturado
```

---

# Boas práticas aprendidas

## Prefira locators semânticos

```python
page.get_by_role(
    "button",
    name="Salvar",
)
```

Em vez de:

```python
page.locator(
    "div:nth-child(8) > button"
)
```

## Evite `time.sleep()`

Evite:

```python
import time

time.sleep(10)
```

Prefira esperar uma condição real:

```python
expect(
    page.get_by_text("Concluído")
).to_be_visible()
```

## Valide toda ação importante

Depois de salvar:

```python
expect(
    page.get_by_text(
        "Successfully Saved"
    )
).to_be_visible()
```

Depois de baixar:

```python
if not caminho.exists():
    raise FileNotFoundError(
        "Arquivo não foi encontrado."
    )
```

Depois de extrair:

```python
if df.empty:
    raise ValueError(
        "Nenhum registro foi extraído."
    )
```

## Não use dados reais em ambientes públicos

Utilize:

```text
Pessoa Teste
Usuário Automação
Empresa Demonstração
```

Não utilize:

```text
CPF real
CNPJ real
Senha corporativa
Nome completo de funcionário
Dados médicos
Dados financeiros
```

## Use o Codegen para descobrir locators

OrangeHRM:

```powershell
playwright codegen --target python "https://opensource-demo.orangehrmlive.com/"
```

Expand Testing:

```powershell
playwright codegen --target python "https://practice.expandtesting.com/"
```

O Codegen ajuda a descobrir elementos, mas o código gerado deve ser revisado antes de entrar no robô final.

---

# Projeto prático após as oito aulas

## Automação de cadastro a partir de Excel

Fluxo:

```text
1. Ler uma planilha com pandas
2. Validar colunas obrigatórias
3. Fazer login
4. Fechar pop-ups
5. Cadastrar cada pessoa
6. Preencher nome, ID e data
7. Selecionar opções em dropdowns
8. Salvar
9. Confirmar a mensagem de sucesso
10. Pesquisar o registro cadastrado
11. Gerar log por pessoa
12. Tirar screenshot das falhas
13. Exportar resultado para Excel
14. Encerrar a sessão
```

Exemplo de entrada:

| id | primeiro_nome | sobrenome | data_nascimento | nacionalidade |
|---:|---|---|---|---|
| 1001 | Pessoa | Teste 1 | 1995-06-15 | Brazilian |
| 1002 | Pessoa | Teste 2 | 1990-03-20 | Brazilian |
| 1003 | Pessoa | Teste 3 | 1988-11-05 | Brazilian |

Exemplo de resultado:

| id | nome | status | mensagem | data_hora |
|---:|---|---|---|---|
| 1001 | Pessoa Teste 1 | Sucesso | Cadastro realizado | 2026-07-08 10:30 |
| 1002 | Pessoa Teste 2 | Erro | ID já existente | 2026-07-08 10:31 |
| 1003 | Pessoa Teste 3 | Sucesso | Cadastro realizado | 2026-07-08 10:32 |
