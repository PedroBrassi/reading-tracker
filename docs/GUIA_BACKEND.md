# Guia de desenvolvimento do backend — Reading Tracker

Responsáveis: **Pedro Luccas e Raissa Peret**. Atualização: 27/10/2026.

Este guia organiza o trabalho do backend com base no README e no roteiro de desenvolvimento do repositório. É um plano de implementação: não declara funcionalidades prontas. O enunciado oficial prevalece; conferir nele as listas completas e os detalhes das validações antes de programar.

## 1. Objetivo e entregas

Construir uma API REST que gerencie autores, livros e leituras, valide os dados e preserve as alterações após reiniciar a aplicação.

- **Entrega 1 — 20/11/2026, 23h59 de Brasília:** o frontend funciona com localStorage. Pedro e Raissa podem estudar e preparar o backend, combinar o contrato de dados e apoiar testes e documentação, preservando essa versão funcional.
- **Entrega 2 — 11/12/2026, 23h59 de Brasília:** API, banco persistente e frontend integrado, com execução documentada, exemplos reais e vídeo da equipe.

Frontend: Aurelio Junior, Ana Flavia Freiria e Otávio de Oliveira. A integração, os testes de aceitação e a documentação são compartilhados.

## 2. O que Pedro e Raissa precisam entregar

- Projeto Java 21 ou superior com Spring Boot 4.x e Maven Wrapper versionado.
- API para autores, livros, leituras, temas, países e restauração dos dados.
- Regras de negócio e validações executadas no servidor, inclusive quando a requisição não passa pelo navegador.
- Banco persistente; H2 em arquivo é o caminho recomendado na documentação do projeto.
- Importação do seed oficial apenas no primeiro início com banco vazio e restauração sob demanda.
- Respostas JSON e erros que o frontend consiga consumir.
- Aplicação completa servida pelo Spring Boot, com frontend em src/main/resources/static.
- Testes relevantes e instruções verificadas para executar a aplicação.

Login, Docker, gráficos e outras extensões opcionais não são prioridade antes do obrigatório.

## 3. Preparação e organização do código

Criar o projeto com Spring Web, Spring Data JPA, Validation e o driver H2. Usar o suporte de testes permitido pelo enunciado. Registrar no README as versões efetivamente utilizadas.

Estrutura sugerida, adaptável ao pacote escolhido pela equipe:

```text
reading-tracker/
├── pom.xml
├── mvnw
├── mvnw.cmd
├── .mvn/wrapper/
├── src/main/java/<pacote>/
│   ├── ReadingTrackerApplication.java
│   ├── controller/
│   ├── service/
│   ├── repository/
│   ├── model/
│   └── config/
├── src/main/resources/
│   ├── application.properties
│   ├── seed.json
│   └── static/          # frontend integrado na entrega 2
└── src/test/java/<pacote>/
```

| Camada | Responsabilidade |
|---|---|
| controller | Receber HTTP/JSON, chamar o serviço e devolver status e resposta |
| service | Validar regras, coordenar operações e delimitar transações |
| repository | Consultar e persistir os dados |
| model | Representar entidades e relacionamentos |
| config | Configurar a aplicação e coordenar a carga inicial |

Manter regras de negócio nos serviços. Não renderizar páginas com Thymeleaf/JSP. Coordenar pom.xml, configuração e entidades compartilhadas antes de alterações simultâneas.

## 4. Modelo de dados: validar com o seed antes de implementar

| Recurso | Papel e relações |
|---|---|
| Country | Lista fixa de países; usada na nacionalidade dos autores |
| Author | Autor que pode estar associado a vários livros |
| Theme | Tema que pode estar associado a vários livros |
| Book | Obra com zero ou mais autores, zero ou mais temas e várias leituras |
| Reading | Uma ocasião de leitura ou intenção de ler, vinculada a um livro |

Uma releitura cria outro registro Reading para o mesmo Book. Não duplicar o livro nem sobrescrever a leitura anterior.

O seed oficial define os campos e referências que devem orientar o contrato JSON. Não inventar os nomes de propriedades restantes nem as listas de valores controlados. Confirmar com o frontend os tipos, datas, IDs, campos opcionais, null e arrays vazios. Evitar respostas com referências circulares entre livro e leitura.

## 5. Contrato de API previsto

Esta tabela acompanha o README. Os exemplos completos de JSON ainda precisam ser acordados com o frontend a partir do seed e depois verificados com a API real.

| Métodos | Caminho | Operação |
|---|---|---|
| GET, POST | /api/authors | Listar e criar autores |
| GET, PUT, DELETE | /api/authors/{id} | Consultar, atualizar e excluir autor |
| GET, POST | /api/books | Listar e criar livros |
| GET, PUT, DELETE | /api/books/{id} | Consultar, atualizar e excluir livro |
| GET, POST | /api/books/{id}/readings | Listar leituras do livro e adicionar uma leitura |
| GET | /api/readings | Listar todas as leituras |
| PUT, DELETE | /api/readings/{id} | Atualizar e excluir leitura |
| GET, POST | /api/themes | Listar e criar temas |
| GET | /api/countries | Listar países fixos |
| POST | /api/reset | Restaurar o seed |

Combinar expressamente o corpo do cadastro de livro com primeira leitura opcional. A operação deve salvar os dois ou nenhum, em uma transação. Ao excluir livro, remover suas leituras na mesma operação, sem excluir autores ou temas compartilhados. Ao excluir autor associado a livro, bloquear e informar os títulos relacionados.

### Status e erros

- 200: consulta ou atualização com resposta.
- 201: criação bem-sucedida.
- 204: operação bem-sucedida sem corpo, como exclusão.
- 400: dados inválidos.
- 404: recurso não encontrado.
- 409: conflito de negócio, como exclusão bloqueada por vínculo.

Documentar a escolha efetiva para cada operação, incluindo reset. Respostas de erro devem conter status, message e errors. Cada erro de campo contém field e message; para 404 e 409, errors fica vazio.

Exemplo ilustrativo de erro, ainda não produzido por uma implementação:

```json
{
  "status": 400,
  "message": "Validation failed",
  "errors": [
    { "field": "yearRead", "message": "Year read is required for Read status." }
  ]
}
```

Combinar os caminhos de field para erros de objetos e listas. A interface deve conseguir mostrar cada mensagem no campo correspondente. Mensagens visíveis ao usuário devem estar em inglês dos EUA.

## 6. Regras que o servidor precisa validar

| Área | Verificações |
|---|---|
| Autores | Nome obrigatório e único sem diferenciar maiúsculas/minúsculas; país opcional da lista fixa; nascimento não futuro; falecimento posterior ao nascimento; exclusão bloqueada quando houver livros vinculados |
| Livros | Título obrigatório; autores e temas existentes; ano opcional, não futuro e diferente de zero; aceitar anos negativos; validar listas controladas; impedir temas repetidos |
| Temas | Nome único sem diferenciar maiúsculas/minúsculas; conferir demais exigências no enunciado |
| Leituras | Livro e status válidos; nova leitura para cada releitura; validar referências e campos abaixo |
| Países | Somente consulta; não permitir criação, edição ou exclusão pela aplicação |

| Status | yearRead | grade | priority |
|---|---|---|---|
| Want to read | Vazio | Vazia | Opcional: Low, Medium ou High |
| Reading | Vazio | Vazia | Vazia |
| Paused | Vazio | Vazia | Vazia |
| Read | Obrigatório | Opcional | Vazia |
| Abandoned | Opcional | Opcional | Vazia |

Quando preenchido, yearRead não pode ser futuro nem anterior à publicação do livro. Grade é de 0 a 10, em intervalos de 0,5. Preservar recommendedBy e notes nas mudanças de status.

O frontend pede confirmação antes de apagar valores incompatíveis e antes das exclusões/restauração exigidas. O backend valida o estado final recebido e não persiste combinações inválidas. Combinar o envio dos valores limpos antes de implementar esse fluxo.

## 7. Persistência, seed e reset

Configuração de desenvolvimento sugerida no roteiro existente:

```properties
spring.datasource.url=jdbc:h2:file:./data/readingtracker
spring.jpa.hibernate.ddl-auto=update
```

Ignorar no Git a pasta de banco gerada na raiz (/data/) e a saída de build (/target/). Não ignorar o seed oficial nem frontend/data/ por engano. Não versionar credenciais ou dados locais do banco.

Procedimento de carga:

1. Obter e inspecionar o seed oficial; confirmar se o acesso aos materiais pendentes foi resolvido.
2. Guardar a cópia do backend em src/main/resources/seed.json.
3. Importar apenas quando o banco estiver vazio; reiniciar não pode duplicar nem sobrescrever dados.
4. Importar países, temas, autores, livros e leituras, respeitando as referências.
5. Deixar o banco gerar IDs; manter mapas entre os IDs do seed e os registros persistidos para montar os vínculos.
6. Executar a carga em transação para evitar dados parcialmente importados.
7. Reutilizar a rotina no reset, removendo registros em ordem que respeite os vínculos e restaurando o conjunto completo.

Após reset, o frontend deve recarregar os dados retornados pelo servidor; não pressupor que IDs anteriores continuarão válidos. Testar falhas de importação para garantir que a transação não deixe uma base incompleta.

## 8. O que combinar e receber do frontend

| Combinar com Aurelio, Ana e Otávio | Resultado esperado |
|---|---|
| Seed e modelo JSON | Mesmos campos, tipos, valores controlados e referências |
| Funções de store.js | Lista de operações, parâmetros e retornos esperados |
| Endpoints e exemplos | Requisições e respostas acordadas para autores, livros e leituras |
| Cadastro combinado | Formato para livro com primeira leitura opcional e tratamento de erros |
| Validação | Mensagens por campo e comportamento para 400, 404 e 409 |
| Confirmações | Cancelamento preserva dados; confirmação envia a operação correta |
| Histórico | Dados para título, autores, status, ano e nota; tratamento de ausências e ordenação |
| Reset e IDs | Recarregamento dos dados após restauração |
| Integração | Páginas funcionais, estados de carregamento/erro e chamadas por store.js |

Pedro e Raissa entregam os endpoints e exemplos ao trio; o trio adapta store.js para fetch com URLs relativas e apresenta os erros do servidor. Ambos verificam o fluxo completo. A interface não precisa estar pronta para começar o backend: o seed e o contrato acordado permitem trabalhar em paralelo.

Na entrega 2, mover o frontend para src/main/resources/static, manter uma única cópia ativa e executar em http://localhost:8080. Retirar localStorage como fonte dos dados da aplicação. Coordenar essa mudança após preservar a entrega 1 na tag correspondente.

## 9. Ordem prática de implementação

1. Conferir seed e regras; acordar o contrato com o frontend.
2. Criar projeto, wrapper, configuração e estrutura de pacotes; verificar inicialização.
3. Implementar entidades, relações, persistência e carga inicial transacional.
4. Construir um fluxo vertical de autores: controller → service → repository → banco, com validação e erros.
5. Implementar temas, países e livros, incluindo relacionamentos e cadastro combinado.
6. Implementar leituras e todas as regras de status, ano, nota e prioridade.
7. Concluir exclusões, reset e testes de integridade.
8. Integrar store.js e as páginas; verificar todos os FR-01 a FR-07.
9. Executar a partir de cópia limpa, revisar documentação e participar do vídeo.

Não marcar uma etapa como concluída só porque os arquivos existem: demonstrar seu comportamento.

## 10. TODO por integrante e fluxo Git

A divisão abaixo organiza as tarefas de Pedro e Raissa. Todos os itens começam pendentes; marcar somente após implementar, verificar e integrar a tarefa. O responsável implementa e o outro integrante revisa o pull request. Os dois devem compreender o backend completo.

### Pedro — infraestrutura, cadastros de apoio e carga de dados

- [ ] Criar a base Spring Boot com Java 21+, dependências, Maven Wrapper e estrutura de pacotes; verificar inicialização e testes.
- [ ] Configurar H2 em arquivo e .gitignore para banco local e saída de build, preservando os arquivos de seed.
- [ ] Combinar com Raissa e o frontend o modelo de dados, as relações e o contrato JSON antes de implementar as entidades.
- [ ] Implementar Country, Author e Theme, com seus repositories e relacionamentos acordados com Book.
- [ ] Definir com Raissa o formato compartilhado de erros e implementar a infraestrutura para respostas 400, 404 e 409.
- [ ] Implementar GET /api/countries, sem operações de alteração de países.
- [ ] Implementar a API de autores: listagem, consulta, criação, atualização e exclusão, com validações de nome, nacionalidade e datas.
- [ ] Bloquear exclusão de autor associado a livros e informar os títulos relacionados, após integrar os relacionamentos de Book.
- [ ] Implementar GET e POST /api/themes, incluindo validação de nomes duplicados sem diferenciar maiúsculas/minúsculas.
- [ ] Implementar a importação transacional do seed: países, temas, autores, livros e leituras, mapeando os IDs originais para os gerados pelo banco.
- [ ] Implementar POST /api/reset reutilizando o importador, respeitando os vínculos e sem depender de IDs fixos.
- [ ] Testar persistência após reinício, ausência de reimportação automática, restauração e rollback da carga quando houver falha.
- [ ] Testar os endpoints e regras de autores, temas e países, incluindo erros e exclusão bloqueada.
- [ ] Documentar execução, configuração, seed/reset e exemplos reais da API de autores.
- [ ] Revisar os pull requests de Raissa e verificar o impacto em persistência, relações e importação.

### Raissa — livros, leituras e regras de negócio

- [ ] Revisar a base preparada por Pedro e verificar se consegue executar o projeto e os testes localmente.
- [ ] Combinar com Pedro e o frontend os campos de Book e Reading, referências, valores controlados e cadastro combinado.
- [ ] Implementar Book e Reading, com seus repositories e relações com Author e Theme, coordenando alterações nos dois lados das associações.
- [ ] Implementar a API de livros: listagem, consulta, criação, atualização e exclusão.
- [ ] Validar título obrigatório, ano de publicação, autores e temas existentes, temas repetidos e valores controlados do livro.
- [ ] Implementar o cadastro de livro com primeira leitura opcional em uma transação: salvar ambos ou nenhum.
- [ ] Implementar GET e POST /api/books/{id}/readings, GET /api/readings e PUT/DELETE /api/readings/{id}.
- [ ] Implementar as regras de Want to read, Reading, Paused, Read e Abandoned, validando yearRead, grade e priority.
- [ ] Preservar recommendedBy e notes nas mudanças de status e criar outro registro para cada releitura.
- [ ] Implementar exclusão de livro com suas leituras em uma transação, preservando autores e temas compartilhados.
- [ ] Aplicar o formato de erros acordado com Pedro, incluindo mensagens por campo em operações compostas.
- [ ] Testar livros e leituras com dados válidos e inválidos, referências inexistentes, anos negativos e futuros e todos os status.
- [ ] Testar rollback do cadastro combinado, exclusão em cascata e preservação do histórico de releituras.
- [ ] Documentar exemplos reais de requisição/resposta de livros e leituras, incluindo erros e cadastro combinado.
- [ ] Revisar os pull requests de Pedro e verificar o impacto das APIs de apoio, do seed e do reset em livros e leituras.

### Pedro e Raissa — coordenação e conclusão

- [ ] Obter o seed oficial e conferir os requisitos completos no enunciado.
- [ ] Acordar com Aurelio, Ana e Otávio os endpoints, exemplos JSON, funções de store.js, mensagens por campo e fluxo de confirmações.
- [ ] Registrar no Trello o responsável, o revisor e as dependências de cada tarefa desta lista.
- [ ] Integrar as entidades antes de concluir o importador; Pedro coordena a carga, Raissa verifica os vínculos e regras de Book e Reading.
- [ ] Entregar ao frontend exemplos verificados dos endpoints para permitir a adaptação de store.js.
- [ ] Integrar com o trio de frontend as páginas em static e as chamadas fetch, mantendo uma única cópia ativa da interface na entrega 2.
- [ ] Validar o fluxo autor → livro → leitura, histórico, erros, confirmações, reset e persistência após reinício.
- [ ] Executar a aplicação em cópia limpa, rodar os testes e revisar todos os critérios da seção 11.
- [ ] Atualizar README, declaração de IA, checklist e exemplos; participar do vídeo e da preparação da entrega 2 com a equipe.

### Ordem e dependências

Primeiro integrar a base Spring Boot. Depois Pedro implementa os cadastros de apoio e a estrutura de erros enquanto Raissa implementa livros e leituras sobre o modelo acordado. A importação completa depende de todas as entidades; o bloqueio de exclusão de autor depende da associação com Book. O cadastro combinado deve reutilizar as regras de leitura implementadas por Raissa. Reset depende do importador concluído.

Acordar os contratos e integrar pequenas partes regularmente para evitar que uma pessoa espere toda a implementação da outra. Coordenar alterações em pom.xml, configurações, entidades relacionadas e seed antes de editar os mesmos arquivos.

### Branches e revisão

| Tarefa | Responsável | Revisor | Branch sugerida |
|---|---|---|---|
| Base Spring Boot e wrapper | Pedro | Raissa | feat/backend-setup |
| Formato compartilhado de erros | Pedro | Raissa | feat/api-errors |
| Países e temas | Pedro | Raissa | feat/countries-themes-api |
| API de autores | Pedro | Raissa | feat/authors-api |
| Entidades e API de livros | Raissa | Pedro | feat/books-api |
| Leituras e cadastro combinado | Raissa | Pedro | feat/readings-api |
| Importação do seed | Pedro | Raissa | feat/backend-seed |
| Restauração | Pedro | Raissa | feat/reset-api |

Atualizar main local antes de criar uma branch por tarefa. Cada integrante faz commits próprios e coerentes. Abrir pull request para main e pedir revisão ao outro integrante antes do merge; evitar commits diretos em main. Após integrar dependências, criar as próximas branches a partir da main atualizada. Não criar branches permanentes por pessoa, frontend ou backend.

A integração com o frontend é uma tarefa conjunta: combinar um responsável pela branch com o trio antes de começar. Atualizar os exemplos da documentação sempre que o contrato mudar.

## 11. Testes e critérios de conclusão

- [ ] Criar, consultar, editar e excluir autores, livros e leituras.
- [ ] Rejeitar dados inválidos enviados diretamente à API.
- [ ] Validar nomes duplicados com diferenças de maiúsculas/minúsculas.
- [ ] Bloquear exclusão de autor associado e identificar os livros.
- [ ] Salvar livro e primeira leitura juntos; uma falha não deixa metade da operação salva.
- [ ] Excluir livro e suas leituras sem apagar autores ou temas compartilhados.
- [ ] Testar todos os status e combinações de yearRead, grade e priority.
- [ ] Confirmar que releitura mantém o histórico anterior.
- [ ] Verificar 400, 404 e 409, incluindo mensagens por campo.
- [ ] Reiniciar e confirmar persistência, sem duplicação do seed.
- [ ] Restaurar o seed e recarregar dados sem depender de IDs anteriores.
- [ ] Confirmar rollback quando a carga ou uma operação composta falha.
- [ ] Testar integração, histórico, confirmações e exibição de erros no navegador.
- [ ] Executar em cópia limpa conforme o README e rodar os testes automatizados relevantes.

Dar prioridade a testes de regras, relacionamentos, transações e persistência, além da verificação dos fluxos pela interface.

## 12. Execução e documentação da entrega

Comandos planejados para executar na raiz, depois de criar e verificar o projeto:

```powershell
java -version
.\mvnw.cmd test
.\mvnw.cmd spring-boot:run
```

Abrir http://localhost:8080 quando o frontend estiver integrado. O primeiro build pode precisar baixar dependências. Não declarar esses comandos como testados antes de executá-los no projeto implementado.

Antes da entrega 2:

- [ ] Registrar versão exata do Java e comando de execução verificado.
- [ ] Atualizar o status real de FR-01 a FR-07 no README.
- [ ] Documentar endpoints e exemplos reais de requisição/resposta de autores, livros e leituras, com IDs retornados pela API.
- [ ] Documentar erros e comportamento do seed/reset.
- [ ] Atualizar a declaração de IA, incluindo este guia e qualquer código ou depuração assistidos.
- [ ] Participar do vídeo de até 15 minutos: demonstrar a aplicação e a persistência, depois explicar a implementação.
- [ ] Conferir o acesso ao vídeo e colocar seu link no README.
- [ ] Preparar com a equipe o commit/tag delivery-2 e a submissão dentro do prazo; não criar a tag final antecipadamente.

## Referências do projeto

- [README](../README.md)
- [Roteiro geral](ROTEIRO_DESENVOLVIMENTO.md)
- [Trello](https://trello.com/b/BSpxAKlv)

Este guia foi preparado com auxílio de ChatGPT/Codex para planejamento e documentação do backend. Não foi implementado código da aplicação nesta atualização.

