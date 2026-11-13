# Face-X_v2.2

# Guia de Configuração e Execução do QML no Python

Este documento explica quais dependências são necessárias para rodar a interface do **FACE-X** (`Screen01.qml`) no Python, onde colocar os arquivos e como abrir só o front-end para conferir se a tela está funcionando.

---

## 1. Dependências do Python

Para o Python renderizar a interface em QML, é necessário instalar os bindings do **Qt**. A interface usa o módulo `Qt5Compat.GraphicalEffects` (blur do fundo), então a recomendação é o **PySide6** completo.

```bash
pip install PySide6
```

> **Nota:** use Python 3.8 ou superior e, se usar ambiente virtual (venv), ative-o antes de instalar.
> Se aparecer o erro `module "Qt5Compat.GraphicalEffects" is not installed`, reinstale com `pip install --force-reinstall PySide6`.

---

## 2. Estrutura de Arquivos e Destino

Não é preciso copiar arquivos de configuração do Qt Design Studio (`.qmlproject`, pastas de metadados etc.). Mova apenas o arquivo de interface (`Screen01.qml`) e os recursos que ele usa (imagens de logo e ícones de tema).

### Caminho de destino

```
Face-X-main/
└── central/
    ├── interface.py      // Script Python que inicializa a aplicação
    ├── Screen01.qml      // Arquivo principal do QML
    ├── Logo_dark.png     // Logo do tema escuro (opcional)
    ├── logo_claro.png    // Logo do tema claro (opcional)
    ├── claro.png         // Ícone do botão de tema (opcional)
    └── escuro.png        // Ícone do botão de tema (opcional)
```

Imagens ausentes não quebram a tela: o botão de tema mostra o texto `CLARO`/`ESCURO` no lugar do ícone.

---

## 3. Script para visualizar o front-end

O item raiz do `Screen01.qml` é um `Item` (não uma `Window`), por isso o script usa `QQuickView`, que cria a janela e ajusta o QML ao tamanho dela. Com `QQmlApplicationEngine.load()` a janela não apareceria.

Crie `central/interface.py`:

```python
import sys
from pathlib import Path

from PySide6.QtCore import QMetaObject, QUrl, Q_ARG
from PySide6.QtGui import QGuiApplication
from PySide6.QtQuick import QQuickView

# True = preenche a tela com dados de exemplo só para conferir o visual
DADOS_DE_TESTE = True

app = QGuiApplication(sys.argv)

view = QQuickView()
view.setTitle("FACE-X")
view.setResizeMode(QQuickView.ResizeMode.SizeRootObjectToView)
view.resize(1280, 800)
view.setSource(QUrl.fromLocalFile(str(Path(__file__).parent / "Screen01.qml")))

if view.status() == QQuickView.Status.Error:
    sys.exit(-1)

if DADOS_DE_TESTE:
    root = view.rootObject()

    QMetaObject.invokeMethod(root, "definirOcorrencias", Q_ARG("QVariant", [
        {
            "idOcorrencia": "ID 37", "nome": "Gustavo Henrique", "sim": "58.76%",
            "cam": "CAM-001", "idade": "19 anos", "situacao": "SUSPEITO",
            "crime": "Roubo", "local": "Faculdade", "status": "PENDING",
            "dataHora": "2026-10-29 19:23:24",
        },
    ]))
    QMetaObject.invokeMethod(root, "definirCadastrados", Q_ARG("QVariant", [
        {"idCad": "1", "nome": "Henrique Keller", "idade": "47",
         "situacao": "PROCURADO", "cadastro": "ATIVO",
         "descricao": "Mandado de prisão em aberto"},
    ]))
    QMetaObject.invokeMethod(root, "definirHistorico", Q_ARG("QVariant", [
        {"idOcorrencia": "33", "dataHora": "16/10/2026 18:58:07",
         "pessoa": "Henrique Keller", "camera": "CAM-001",
         "similaridade": "97.90%", "status": "CONFIRMED"},
    ]))

view.show()
sys.exit(app.exec())
```

Execute a partir da pasta `central`:

```bash
cd Face-X-main/central
python interface.py
```

Com `DADOS_DE_TESTE = False` a tela abre vazia (`-` nos campos e `Nenhuma ocorrência`), que é o estado real antes de o back-end enviar dados.

---

## 4. Estado atual

- A interface está pronta para integração com Python/PostgreSQL: os dados de exemplo (mock) foram removidos do QML e os modelos começam vazios.
- O Python preenche a tela chamando funções do item raiz: `definirOcorrencias(lista)`, `adicionarOcorrencia(obj)`, `definirCadastrados(lista)` e `definirHistorico(lista)`. Os campos esperados estão comentados no próprio QML.
- **Pendente:** avisar o back-end quando o operador confirmar/descartar uma ocorrência e quando mexer nos cadastros. Hoje essas ações só atualizam a memória do QML.
- A imagem de fundo é carregada de uma URL fixa no QML, então precisa de internet para aparecer.
