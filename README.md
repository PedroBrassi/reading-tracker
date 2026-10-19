# Reading Tracker

Aplicação web para gerenciar autores, livros e leituras, desenvolvida para o Trabalho #2 da disciplina **Programação Web (DCE701), UNIFAL-MG, 2026/2**.

O projeto será construído em duas etapas: frontend em HTML, CSS e JavaScript puros com armazenamento no navegador; depois, integração com uma API REST em Java e Spring Boot e persistência em banco de dados.

> **Situação inicial:** este README descreve o planejamento. As funcionalidades e os comandos da etapa backend ainda precisam ser implementados e verificados. Atualizar o documento conforme o desenvolvimento; não entregar com pendências de documentação.

## Equipe

| Integrante | Matrícula |
|---|---|
| Aurelio Junior | A informar |
| Ana Flavia Freiri | A informar |
| Raissa Nunes | A informar |
| Pedro Brassi | A informar |

A composição da equipe deve ser mantida entre as duas entregas. Cada integrante deve contribuir com commits e compreender o código apresentado.

## Repositório e planejamento

- [Repositório público](https://github.com/PedroBrassi/reading-tracker)
- [Quadro de tarefas no Trello](https://trello.com/b/BSpxAKlv)
- [Roteiro de desenvolvimento](docs/ROTEIRO_DESENVOLVIMENTO.md)
- [Enunciados no Moodle](https://campusvirtual.unifal-mg.edu.br/moodle/course/section.php?id=30335)

## Entregas

| Etapa | Prazo no horário de Brasília | Escopo | Valor |
|---|---|---|---|
| Entrega 1 | **20/11/2026 às 23h59** | Frontend completo, FR-01 a FR-07 e localStorage | 5,0 |
| Entrega 2 | **11/12/2026 às 23h59** | API REST, persistência e frontend integrado | 5,0 |

## Tecnologias

| Camada | Tecnologia prevista |
|---|---|
| Frontend | HTML, CSS e JavaScript puros; módulos ES; Fetch API e JSON |
| Dados da entrega 1 | localStorage com importação inicial de seed.json |
| Backend da entrega 2 | Java 21 ou superior e Spring Boot 4.x |
| Build do backend | Maven com wrapper (escolha proposta para o projeto) |
| Persistência | Spring Data JPA e H2 em arquivo (caminho recomendado pelo enunciado) |

A interface e os valores controlados serão em **inglês dos Estados Unidos**. Os nomes de autores e títulos de livros permanecem no idioma original.

Não usar React, Vue, Angular, Svelte, jQuery, Bootstrap, Tailwind, TypeScript, Vite, Webpack ou pacotes npm/build tools para construir o frontend. Não usar Thymeleaf/JSP para renderização no servidor. Bibliotecas adicionais dependem das permissões do enunciado ou de aprovação registrada do professor.

## Estrutura prevista para a entrega 1

```text
reading-tracker/
├── README.md
├── .gitignore
├── docs/
│   └── ROTEIRO_DESENVOLVIMENTO.md
└── frontend/
    ├── index.html
    ├── authors.html
    ├── books.html
    ├── readings.html
    ├── css/styles.css
    ├── js/
    │   ├── store.js
    │   ├── validation.js
    │   ├── ui.js
    │   ├── authors.js
    │   ├── books.js
    │   └── readings.js
    ├── data/seed.json
    └── assets/images/
```

`store.js` concentra todas as operações de dados em funções assíncronas. Na entrega 2, será adaptado para chamar a API, e o frontend será movido para `src/main/resources/static`. O seed usado pelo backend ficará em `src/main/resources/seed.json`.

## Como preparar e executar a entrega 1

As instruções abaixo são o procedimento planejado e devem ser testadas quando a interface estiver implementada.

1. Clonar o repositório, se ainda não existir uma cópia local:

   ```powershell
   git clone git@github.com:PedroBrassi/reading-tracker.git
   cd reading-tracker
   code .
   ```

2. Obter o `seed.json` oficial e colocá-lo em `frontend/data/seed.json`.
3. No VS Code, usar a extensão **Live Server** e abrir `frontend/index.html` com **Open with Live Server**.
4. Acessar a URL HTTP indicada pela extensão. Não abrir a página somente por `file://`, pois o carregamento do seed usa `fetch`.
5. No primeiro acesso, carregar o seed; nos acessos seguintes, manter as alterações no armazenamento do navegador.

Na verificação de 19/10/2026, os links de `seed.json`, `How_Books_Read_Currently_Tracked.pdf` e `Students_Seed_Data_Readme.pdf` estavam indisponíveis no Moodle. Confirmar novamente e obter os arquivos com o professor se necessário. Dados fictícios de teste não substituem o seed obrigatório na entrega.

## Como executar a entrega 2

**Planejado, ainda não verificado.** Após implementar o backend e versionar o Maven Wrapper:

1. Ter Java 21 ou superior instalado e conferir `java -version`.
2. Na raiz do repositório, executar:

   ```powershell
   .\mvnw.cmd spring-boot:run
   ```

3. Abrir `http://localhost:8080`.

Registrar aqui, antes da entrega, a versão exata do Java utilizada. O banco H2 deve estar em arquivo, preservar dados após reinícios e ter sua pasta gerada ignorada pelo Git. Não será necessário instalar Maven separadamente quando o wrapper estiver completo.

## Requisitos obrigatórios

Atualizar cada célula para **Implementado**, **Parcialmente implementado** ou **Não implementado**, conforme verificação real. O status inicial abaixo não pressupõe funcionalidades prontas.

| ID | Requisito | Entrega 1 | Entrega 2 |
|---|---|---|---|
| FR-01 | CRUD de autores e bloqueio de exclusão de autor vinculado | Não implementado | Não implementado |
| FR-02 | CRUD de livros, seleção de autores e temas e criação de tema | Não implementado | Não implementado |
| FR-03 | CRUD de leituras, cadastro combinado e exclusão em cascata confirmada | Não implementado | Não implementado |
| FR-04 | Fluxo de status e compatibilidade de ano, nota e prioridade | Não implementado | Não implementado |
| FR-05 | Histórico ordenável e tratamento de valores ausentes | Não implementado | Não implementado |
| FR-06 | Validações da seção 4 e mensagens junto aos campos | Não implementado | Não implementado |
| FR-07 | Seed no primeiro uso, preservação de alterações e Reset data | Não implementado | Não implementado |

Os exercícios FR-08 a FR-18 são opcionais e não pontuados; nenhum está declarado como implementado neste momento.

## API da entrega 2

**Contrato previsto, ainda não implementado.** Atualizar a tabela se os caminhos forem alterados.

| Métodos | Caminho | Finalidade |
|---|---|---|
| GET, POST | /api/authors | Listar e criar autores |
| GET, PUT, DELETE | /api/authors/{id} | Consultar, atualizar e excluir autor |
| GET, POST | /api/books | Listar e criar livros |
| GET, PUT, DELETE | /api/books/{id} | Consultar, atualizar e excluir livro |
| GET, POST | /api/books/{id}/readings | Listar e adicionar leituras de um livro |
| GET | /api/readings | Listar todas as leituras |
| PUT, DELETE | /api/readings/{id} | Atualizar e excluir leitura |
| GET, POST | /api/themes | Listar e criar temas |
| GET | /api/countries | Listar os países fixos |
| POST | /api/reset | Restaurar o seed |

**Pendente para a entrega 2:** acrescentar exemplos reais de requisição e resposta para autores, livros e leituras. Registrar códigos HTTP, erros de validação e exemplos com os IDs efetivamente retornados pela API.

## Uso de inteligência artificial

| Ferramenta | Finalidade já utilizada | Partes afetadas |
|---|---|---|
| ChatGPT / Codex | Explicação do enunciado; planejamento da estrutura, das etapas e do Trello; preparação da documentação inicial | README.md e roteiro de desenvolvimento |

Até a preparação desta documentação, não foi produzido código da aplicação nesta conversa. A equipe deverá atualizar esta seção com qualquer uso posterior de IA, incluindo explicações, depuração e geração de código, identificando as partes afetadas. Todos devem compreender o que entregarem.

## Submissão e vídeo

Enviar para **pagliares@bcc.unifal-mg.edu.br**, com todos os demais integrantes em cópia:

- Entrega 1: assunto `Reading Tracker – Delivery 1 – [nomes dos estudantes]`; incluir GitHub e autoavaliação de 0 a 5.
- Entrega 2: assunto `Reading Tracker – Delivery 2 – [nomes dos estudantes]`; incluir GitHub, autoavaliação de 0 a 5 e vídeo de até 15 minutos acessível sem login.

**Vídeo da entrega 2:** link pendente. Adicionar aqui antes do envio. Demonstrar primeiro a aplicação e a persistência após reinício; depois explicar as principais partes do código.

Marcar e enviar ao GitHub as tags `delivery-1` e `delivery-2` nos commits a serem avaliados, dentro dos respectivos prazos. Não criar as tags finais antecipadamente. Fazer commits descritivos durante todo o trabalho.

## Referência

Enunciado **Reading Tracker Web Application**, atualizado em **16/10/2026**, disponibilizado no Moodle da disciplina. O roteiro de desenvolvimento diferencia exigências oficiais de sugestões de organização.
