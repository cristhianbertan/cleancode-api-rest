# Entrega 01: Dossiê do projeto de refatoração (AgriPecu)

## 1. Integrantes e repositório

| Integrante | GitHub |
|---|---|
| Alexandre Sartor Teixeira | https://github.com/AlexandreSartor |
| Carine Ghisi Cadorin | https://github.com/Polarzinha008 |
| Cristhian Cardoso Bertan | https://github.com/cristhianbertan |
| Guilherme Zapelini da Rosa Schwan | https://github.com/wguii |

**Repositório público:** https://github.com/cristhianbertan/cleancode-api-rest

> Observação: o sistema foi escrito originalmente em outra disciplina por Cristhian Bertan e dois colegas que não fazem parte deste grupo (eles aparecem em `SobreController`). Neste trabalho, o grupo apenas refatora o código, sem acrescentar funcionalidades novas.

## 2. Descrição do software

**AgriPecu** é uma API REST que consulta o clima atual de uma cidade no OpenWeather e devolve **recomendações operacionais para agricultura e pecuária**, como colheita, plantio, aragem, pulverização, pastoreio, manejo, reprodução e produção de leite. Cada consulta fica gravada no banco e pode ser vista depois como histórico.

**Domínio:** agronegócio, mais precisamente apoio à decisão no campo com base em dados meteorológicos.

**Principais funcionalidades**

| Endpoint | Função |
|---|---|
| `POST /clima` | Recebe cidade, tipo (agricultura/pecuária) e atividade, e devolve as condições atuais e uma recomendação |
| `POST /avaliar` | Diz se o clima da cidade está favorável ou desfavorável para trabalhar no campo |
| `GET /historico` | Lista todas as consultas já feitas |
| `GET/POST/PUT/DELETE /usuarios` | CRUD de usuários |
| `GET/POST/PUT/DELETE /consultas`, `/consultas/cidade/{c}`, `/consultas/usuario/{id}` | CRUD e filtros das consultas, com cache Caffeine de 60 s |
| `GET /sobre` | Autores do projeto |

**Stack:** Java 17, Spring Boot 3.4.5 (Web, Data JPA, Cache), Hibernate, banco H2 em memória, Caffeine 3.1.8, Jackson, `RestTemplate` para a API do OpenWeather, Maven (com o wrapper `mvnw`) e JUnit 5.

## 3. Contagem de linhas (cloc)

Comando: `npx cloc@2.6.0 --exclude-dir=.idea,.mvn,target .` (cloc 1.86)

```
github.com/AlDanial/cloc v 1.86  T=0.10 s (231.2 files/s, 15826.8 lines/s)
-------------------------------------------------------------------------------
Language                     files          blank        comment           code
-------------------------------------------------------------------------------
Java                            20            186              0            736
Bourne Shell                     1             26             48            185
Markdown                         1             51              0            178
DOS Batch                        1             15              0            134
Maven                            1              9              0             75
-------------------------------------------------------------------------------
SUM:                            24            287             48           1308
-------------------------------------------------------------------------------
```

Só o código de produção (`npx cloc@2.6.0 src/main/java`): **19 arquivos Java, 727 linhas de código**. Os testes (`src/test`) têm 1 arquivo com 9 linhas. Os scripts `mvnw` e `mvnw.cmd` (Bourne Shell e DOS Batch) foram gerados pelo Maven e não entram no escopo.

Arquivos com mais linhas: `ClimaService.java` (152), `ConsultaClima.java` (92), `ConsultaClimaService.java` (57), `HistoricoClima.java` (56), `ConsultaClimaController.java` (51).

## 4. Baseline medido

Todas as medições foram feitas no commit `c31c468`, sem nenhuma alteração no `pom.xml`. Os plugins foram chamados direto pela linha de comando.

### 4.1 Cobertura de testes (JaCoCo 0.8.12)

Comando: `./mvnw org.jacoco:jacoco-maven-plugin:0.8.12:prepare-agent test org.jacoco:jacoco-maven-plugin:0.8.12:report`

Testes: **1 teste** (`AgriPecuApplicationTests.contextLoads`), que passou (1 run, 0 failures).

| Métrica | Cobertas / Total | % |
|---|---|---|
| Linhas | 24 / 292 | **8,2%** |
| Branches | 0 / 73 | **0,0%** |
| Instruções | 73 / 1062 | 6,9% |
| Métodos | 14 / 115 | 12,2% |

O pouco que aparece como coberto vem só da inicialização do contexto Spring (construtores e beans). Nenhuma regra de negócio é testada. `ClimaService` tem 3 de 81 linhas cobertas e 0 de 63 branches.

### 4.2 Avisos de linter

**Checkstyle** (maven-checkstyle-plugin 3.6.0, Checkstyle 9.3, `google_checks.xml`): **623 avisos**

| Regra | Qtd. |
|---|---|
| Indentation | 523 |
| PackageName | 19 |
| MissingJavadocType | 19 |
| MissingJavadocMethod | 19 |
| CustomImportOrder | 19 |
| LineLength | 13 |
| AvoidStarImport | 5 |
| FileTabCharacter | 3 |
| EmptyLineSeparator | 2 |
| WhitespaceAfter | 1 |

O Google Style pede indentação de 2 espaços e o projeto usa 4, então 523 desses avisos são só de indentação. Sem eles, sobram **100 avisos**. Na Entrega 02 vamos adotar uma configuração própria (Google com 4 espaços) e repetir a medição com ela.

**PMD** (maven-pmd-plugin 3.26.0, ruleset `quickstart`): **27 violações**

| Regra | Qtd. | Onde |
|---|---|---|
| PackageCase | 19 | todos os arquivos (`com.AgriPecu.AgriPecu`) |
| UseLocaleWithCaseConversions | 4 | `ClimaService.java:58, 70, 72, 114` |
| UnnecessaryImport | 2 | `ClimaService.java:3`, `SobreController.java:4` |
| UseUtilityClass | 1 | `AgriPecuApplication.java:9` |
| UncommentedEmptyConstructor | 1 | `Usuario.java:18` |

Com o ruleset padrão do plugin, o PMD acusa apenas 1 violação (`UnnecessaryImport`).

### 4.3 Complexidade (Lizard 1.26.1)

Comando: `uvx lizard src/main/java -l java`

| Total NLOC | Funções | CCN médio | Avisos (CCN > 15) |
|---|---|---|---|
| 727 | 101 | 1,5 | **2** |

| Função | NLOC | CCN | Linhas |
|---|---|---|---|
| `ClimaService::gerarRecomendacaoAgricultura` | 39 | **18** | 69–111 |
| `ClimaService::gerarRecomendacaoPecuaria` | 38 | **18** | 113–153 |

`ClimaService` tem CCN médio de 8,0 por função, contra cerca de 1,0 a 1,4 nas outras classes. A complexidade do sistema se concentra quase toda nessa classe.

## 5. Versão original

- **Branch de trabalho:** `main` (no repositório antigo a branch era `master`)
- **Commit:** `c31c468ce9b59c63cdc73a0c175765f9e02794a3` ("Ajustes no HttpResponse do UsuárioController", 27/06/2025)
- **Tag e branch a criar:** `v0-original` e `original`

```bash
git tag -a v0-original c31c468 -m "Versão original antes da refatoração"
git branch original c31c468
git push origin v0-original original
```

## 6. Catálogo inicial de smells

As categorias seguem a taxonomia de Mäntylä para os smells de Fowler: Bloaters, Object-Orientation Abusers, Change Preventers, Dispensables e Couplers.

| ID | Smell | Categoria | Arquivo:linha | Impacto |
|---|---|---|---|---|
| S01 | Long Method / Complex Conditional | Bloaters | `service/ClimaService.java:69` | **Alto.** `gerarRecomendacaoAgricultura` tem CCN 18 e 43 linhas, com `if/else` encadeado dentro de cada `case`. É difícil de ler e de testar. |
| S02 | Long Method / Complex Conditional | Bloaters | `service/ClimaService.java:113` | **Alto.** `gerarRecomendacaoPecuaria` tem CCN 18 e 41 linhas, com a mesma estrutura do S01. |
| S03 | Magic Numbers | Bloaters | `service/ClimaService.java:74–145, 167` | **Alto.** As faixas de temperatura (10, 15, 18, 20, 25, 28, 30, 32, 35) aparecem soltas e repetidas. Mudar um limite exige procurar o número no código. |
| S04 | Primitive Obsession | Bloaters | `model/ClimaRequest.java:8–10`; `service/ClimaService.java:58, 70, 114` | **Médio.** `tipo` e `atividade` são `String` livres comparadas com literais em `switch`. Um erro de digitação passa sem aviso e cai no `default`. |
| S05 | Switch Statements (switch aninhado por tipo de código) | OO Abusers | `service/ClimaService.java:58, 70, 114` | **Alto.** Há um switch por tipo e, dentro dele, outro por atividade. Para incluir uma atividade é preciso alterar o switch e a classe de serviço, o que viola o princípio aberto/fechado. |
| S06 | Temporary Field | OO Abusers | `model/ConsultaClima.java:30, 39` | **Médio.** A mesma entidade guarda dois conceitos diferentes: recomendação (`tipo`, `atividade`, `recomendacao`) e avaliação (`avaliacao`). O conjunto de campos `null` muda conforme o construtor usado. |
| S07 | Alternative Classes with Different Interfaces | OO Abusers | `service/UsuarioService.java:38`; `service/ConsultaClimaService.java:50`; `controller/ConsultaClimaController.java:42` | **Médio.** Os dois CRUDs indicam "não encontrado" de formas diferentes: um retorna `null`, o outro lança `RuntimeException`, que o controller captura de forma genérica. |
| S08 | Divergent Change / Large Class | Change Preventers | `service/ClimaService.java:14` | **Alto.** A classe monta a URL, chama a API HTTP, faz o parse do JSON, aplica as regras de negócio, grava no banco e ainda serve o histórico. Qualquer mudança passa por ela. |
| S09 | Duplicated Code (chamada e parse do OpenWeather) | Dispensables | `service/ClimaService.java:30–39` e `155–163` | **Alto.** URL, `getForObject`, `readTree` e extração de `temp` e `condicao` estão copiados em dois métodos. |
| S10 | Duplicated Code (tratamento de erro) | Dispensables | `service/ClimaService.java:47–51` e `177–181` | **Médio.** Os blocos `catch (Exception e)` são quase idênticos. O `catch` genérico também esconde bugs: com `tipo == null`, o `NullPointerException` vira a resposta "Erro ao consultar dados climáticos". |
| S11 | Dead Code / Lazy Class | Dispensables | `model/HistoricoClima.java:3`; `model/ClimaResponse.java:19–37` | **Baixo.** `HistoricoClima` não é referenciada em lugar nenhum (0% de cobertura) e os setters de `ClimaResponse` nunca são chamados. |
| S12 | Duplicated Code / Comments | Dispensables | `AgriPecuApplication.java:8` e `config/CacheConfig.java:13`; `config/CacheConfig.java:18, 25–27`; `model/ConsultaClima.java:31, 40` | **Baixo.** `@EnableCaching` aparece duas vezes, e há comentários que só repetem o que o código já diz. |
| S13 | Inappropriate Intimacy (entidades JPA expostas na API) | Couplers | `controller/UsuarioController.java:21, 34, 40`; `controller/ConsultaClimaController.java:20, 32, 38` | **Alto.** O contrato HTTP fica acoplado ao modelo de persistência. `Usuario.senha` é devolvida no JSON de `GET /usuarios`. |
| S14 | Middle Man | Couplers | `service/ClimaService.java:54`; `controller/HistoricoController.java:19` | **Baixo.** `getHistorico()` apenas repassa a chamada para `ConsultaClimaService.getAllConsultas()`. |
| S15 | Dependências concretas e injeção por campo | Couplers | `service/ClimaService.java:19–20, 22`; `@Autowired` em todos os controllers e services | **Alto.** `new RestTemplate()` e `new ObjectMapper()` são criados dentro da classe. Sem injeção pelo construtor, não dá para usar mocks em teste unitário. |
| S16 | Configuração e segredo no código | Couplers (com o ambiente) | `resources/application.properties:1, 12`; `variables.env:1`; `service/ClimaService.java:30, 156` | **Crítico.** A chave do OpenWeather está versionada num repositório público. A URL está fixa no código, e o dialeto do Hibernate é PostgreSQL embora o banco seja H2. |

Fora do catálogo, encontramos também:
- **Bug de cache:** `updateConsulta` e `deleteConsulta` (`ConsultaClimaService.java:36, 54`) removem do cache só a chave `#id`. As entradas `allConsultas`, `byCidade-*` e `byUsuario-*` continuam com dados antigos por até 60 s.
- **Tratamento inconsistente de maiúsculas:** em `avaliarClima`, `condicao.contains("chuva")` (`ClimaService.java:167`) não usa `toLowerCase()`, ao contrário do que acontece em `ClimaService.java:72`.

## 7. Plano de refatoração

### 7.1 Técnicas do catálogo (Fowler, *Refactoring*, 2ª ed.)

| Smell | Técnica |
|---|---|
| S01, S02 | Extract Function, Decompose Conditional, Replace Nested Conditional with Guard Clauses |
| S03 | Replace Magic Literal (Symbolic Constant), com as faixas num objeto de valor `FaixaTemperatura` |
| S04 | Replace Primitive with Object, com os enums `TipoProducao` e `Atividade` |
| S05 | Replace Conditional with Polymorphism, com Strategy: uma `RegraRecomendacao` por atividade e um mapa `Atividade → regra` |
| S06 | Extract Class e Introduce Parameter Object, separando recomendação e avaliação ou criando um `TipoConsulta` explícito |
| S07 | Replace Error Code with Exception (`RecursoNaoEncontradoException`) com `@RestControllerAdvice` |
| S08 | Extract Class (`OpenWeatherClient`, `RecomendacaoService`, `AvaliacaoService`) e Move Function |
| S09 | Extract Function / Extract Class (`OpenWeatherClient.buscarClimaAtual(cidade) → CondicaoClimatica`) |
| S10 | Extract Function para o tratamento de erro, com captura das exceções específicas (`RestClientException`, `JsonProcessingException`) |
| S11, S12 | Remove Dead Code, retirada de comentários redundantes e da anotação duplicada |
| S13 | Introduce Parameter Object e criação de DTOs `record` de request e response, com mapeamento na fronteira e sem `senha` na saída |
| S14 | Remove Middle Man |
| S15 | Injeção por construtor, com `RestTemplate` e `ObjectMapper` como beans |
| S16 | Mover configuração para `application.properties` com variável de ambiente (`${OPENWEATHER_API_KEY}`) e corrigir o dialeto |

### 7.2 Ordem de execução e PRs previstos

O primeiro passo é montar uma rede de segurança: testes de caracterização que registram o comportamento atual, incluindo os textos exatos das recomendações. Só depois começam as refatorações de maior risco. Cada PR deve manter todos os testes verdes e não pode baixar a cobertura.

| PR | Título | Conteúdo | Smells |
|---|---|---|---|
| PR 0 | Infra de qualidade | Checkstyle e PMD no `pom.xml` sem bloquear o build no início, Spotless, JaCoCo com relatório, GitHub Actions e hook de pre-commit | |
| PR 1 | Remover segredo da configuração | Chave lida de variável de ambiente, `variables.env` no `.gitignore`, `.env.example`, URL e dialeto ajustados no `application.properties`. A chave antiga deve ser revogada antes. | S16 |
| PR 2 | Testes de caracterização | Testes de `ClimaService` com `MockRestServiceServer` ligado ao `RestTemplate` interno via `ReflectionTestUtils`, sem mexer no código de produção, cobrindo todos os ramos de agricultura, pecuária e avaliação. Testes `@WebMvcTest` dos controllers e testes dos services com Mockito. | (rede de segurança) |
| PR 3 | Injeção por construtor | `@Autowired` em campo trocado por construtor, `RestTemplate` e `ObjectMapper` como beans, e testes simplificados | S15 |
| PR 4 | Extrair `OpenWeatherClient` | Remove a duplicação da chamada e do parse e trata exceções específicas | S09, S10, S08 (parte) |
| PR 5 | Constantes e enums | Faixas de temperatura nomeadas, enums `TipoProducao` e `Atividade` | S03, S04 |
| PR 6 | Strategy para recomendações | Uma classe de regra por atividade, sem os switches aninhados, e CCN de cada método abaixo de 5 | S01, S02, S05, S08 |
| PR 7 | DTOs e tratamento de erros | DTOs `record`, `@RestControllerAdvice`, `RecursoNaoEncontradoException` e senha fora das respostas | S07, S13 |
| PR 8 | Limpeza | Remove `HistoricoClima`, setters não usados, `@EnableCaching` duplicado, comentários redundantes e o middle man do histórico. Também corrige o `@CacheEvict` e separa a consulta da avaliação. | S06, S11, S12, S14 |
| PR 9 (opcional) | Renomear pacote | `com.AgriPecu.AgriPecu` passa a ser `com.agripecu`, o que zera os 19 avisos de `PackageCase`/`PackageName` | nomenclatura |

**Metas para o fim do projeto:** cobertura de linhas **≥ 50%** (hoje 8,2%) e de branches **≥ 50%** (hoje 0%), **0 funções com CCN > 10** (hoje 2 com CCN 18), **0 violações** de PMD quickstart e Checkstyle na configuração do projeto, e o CI bloqueando PRs que piorem esses números.

## 8. Ferramentas

| Finalidade | Ferramenta |
|---|---|
| Linter | Checkstyle (`maven-checkstyle-plugin`, Google Style ajustado para 4 espaços) e PMD (`maven-pmd-plugin`, ruleset quickstart) |
| Formatador | Spotless (`spotless-maven-plugin`) com `google-java-format` no estilo AOSP (4 espaços) |
| Framework de testes | JUnit 5, Mockito, AssertJ, Spring Boot Test (`@WebMvcTest`, `MockMvc`) e `MockRestServiceServer` para simular o OpenWeather |
| Cobertura | JaCoCo (`jacoco-maven-plugin`), com `jacoco:check` exigindo mínimo de 50% de linhas no final |
| Complexidade | Lizard (`uvx lizard src/main/java -l java -C 10`) |
| Contagem de linhas | cloc |
| CI | GitHub Actions: `./mvnw verify` (testes, JaCoCo, Checkstyle, PMD, `spotless:check`) a cada push e PR, com o relatório de cobertura salvo como artefato |
| Hooks | Hook de pre-commit versionado em `.githooks/pre-commit` (`git config core.hooksPath .githooks`), que roda `spotless:apply` e `checkstyle:check` antes do commit |
| Modelos de IA | Claude Code (Claude Opus 5.5), usado para levantar smells, medir o baseline e sugerir testes e refatorações. Todo código gerado é revisado por um integrante antes do merge, e o uso fica registrado na descrição de cada PR. |

## 9. Riscos, limitações e o que não será refatorado

**Riscos**
- **Não existe rede de segurança hoje:** há um único teste, de smoke. Mitigação: o PR 2 vem antes de qualquer refatoração estrutural, e os testes de caracterização registram os textos exatos das recomendações.
- **Dependência de API externa:** o OpenWeather depende de rede e de chave e muda com o tempo. Mitigação: nenhum teste chama a API real, todos usam `MockRestServiceServer`.
- **Mudança acidental de comportamento** ao trocar os switches por Strategy: valores de fronteira (como 18, 30 e 35 °C) podem trocar de faixa. Mitigação: testes parametrizados nos limites de cada faixa.
- **Contrato da API:** os DTOs do PR 7 mudam o JSON de `/usuarios` (a senha deixa de aparecer). É uma mudança intencional e vai ser documentada no PR.
- **Chave vazada:** apagar o arquivo não remove a chave do histórico do git. A ação certa é revogar a chave no painel do OpenWeather.
- **Conflitos de merge:** vários PRs mexem em `ClimaService`. Mitigação: PRs pequenos, feitos em sequência, com rebase frequente.

**Limitações do baseline**
- No Checkstyle, 523 dos 623 avisos são só de indentação (2 contra 4 espaços), então o número cheio exagera o problema.
- O ruleset quickstart do PMD é enxuto e não pega smells de design como God Class. Esses smells foram levantados manualmente no catálogo da seção 6.
- O cloc não conta como comentário os comentários no fim da linha (`// ...`), por isso aparece "comment = 0".

**Fora do escopo**
- Funcionalidades novas (previsão de vários dias, novas atividades, autenticação e autorização).
- Hash de senha e segurança de usuários, além de tirar a senha das respostas.
- Troca do banco H2 por PostgreSQL ou criação de migrações (Flyway/Liquibase).
- Troca do `RestTemplate` por `WebClient`/`RestClient`.
- Reescrita do histórico do git para apagar a chave antiga, já que ela será revogada.
- Arquivos gerados ou de IDE: `mvnw`, `mvnw.cmd`, `.mvn/`, `.idea/` e os metadados vazios do `pom.xml`.
- Atualização de versões do Spring Boot e de outras dependências.
