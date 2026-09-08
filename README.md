# Checkpoint 4 — Bug Hunt StreamFIAP


**Turma** 2CCPG

| Integrante | RM | 
|---|---|
|Bruno Otávio da Cruz Carvalho | RM562354| |
| Letícia Gabrielle Andrade Temóteo | RM563985| |
| João Vitor Santana Silva Ribeiro| RM564693| |
|Rafael Quatter Dalla Costa | RM562052| |
|Rafael Louzã Lopes | RM564963| |



## Parte 1 — Bugs encontrados

| # | Sintoma observado (o que fiz/vi) | Causa raiz (arquivo e linha aproximada) | Correção aplicada | Conceito da disciplina |
|---|---|---|---|---|
| bug01 | Um filme de estreia com preço normal de R$ 14,90 retornou R$ 17,88 na promoção, em vez de R$ 11,92. | `Filme.java`, método `aplicarPromocao`: o preço era multiplicado por `1.2`, aumentando 20%. | Alterado o multiplicador para `0.8`, aplicando corretamente 20% de desconto. | Interface, sobrescrita de método e regra de negócio. |
| bug02 | Ao criar a série Dark, título e categoria ficaram nulos; duração e classificação ficaram zeradas. | `Serie.java`, construtor: apenas `numeroTemporadas` era atribuído e o construtor da classe pai não era chamado. | Adicionada a chamada `super(...)` e o parâmetro de disponibilidade foi preservado no cadastro. | Herança, construtores e inicialização de objetos. |
| bug03 | Uma série com 5 temporadas retornou R$ 9,90, em vez de R$ 24,50. | `Serie.java`: `calcularPrecoAluguel(double)` sobrecarregava o método, enquanto o aluguel chamava a versão sem argumentos herdada. | Removido o parâmetro e adicionada `@Override`, mantendo R$ 4,90 por temporada. | Polimorfismo, sobrescrita e sobrecarga de métodos. |
| bug04 | Um documentário retornou preço de aluguel de R$ 9,90, embora o contrato determine gratuidade. | `Documentario.java`: a classe não sobrescrevia o cálculo e herdava o preço padrão de `Conteudo`. | Sobrescrito `calcularPrecoAluguel()` para retornar `0.0`. | Herança, polimorfismo e sobrescrita de método. |
| bug05 | Ao cadastrar um usuário, o ID não era gerado automaticamente. | `Usuario.java`: o campo `id` possuía `@Id`, mas não tinha estratégia de geração configurada. | Adicionada `@GeneratedValue(strategy = GenerationType.IDENTITY)`. | Persistência JPA, chave primária e geração de identidade. |
| bug06 | Um usuário criado com o nome Rafael retornou `nome=null`. | `Usuario.java`, construtor: `nome = nome` atribuía o parâmetro a ele mesmo. | Alterada a atribuição para `this.nome = nome`. | Construtores, estado do objeto e uso de `this`. |
| bug07 | Filmes com duração `0` e `-20` foram criados normalmente. | `Conteudo.java`: construtor e setter não validavam `duracaoMinutos`. | Centralizada no setter a rejeição de valores `<= 0`; o construtor passou a usar o setter e a API retorna HTTP 400 com mensagem clara. | Encapsulamento, validação de estado e exceções. |
| bug08 | A busca pelo conteúdo 999 retornou `null`, como se fosse uma resposta válida. | `ConteudoController.java`: um `catch (Exception)` vazio engolia `ConteudoNaoEncontradoException`. | Removido o `try/catch` vazio e a exceção passou a ser propagada ao handler global, que responde HTTP 404. | Exceções, propagação de erros e responsabilidade do controller. |
| bug09 | Um conteúdo com categoria `FICCAO` não foi retornado ao buscar pelo mesmo texto. | `ConteudoController.java`: categorias eram comparadas com `==`, que compara referências de objetos. | Substituída a condição por `categoria.equals(c.getCategoria())`. | Comparação de objetos, igualdade de Strings e coleções. |
| bug10 | Um usuário abaixo da classificação recebia erro genérico da API, sem a mensagem da regra. | `ClassificacaoIndicativaException` era checked e não possuía tratamento no `GlobalExceptionHandler`. | Alterada para `RuntimeException` e criado handler HTTP 403 que devolve a mensagem original. | Exceções checked e unchecked e tratamento global. |
| bug11 | Usuário com R$ 100,00 foi considerado sem saldo para pagar R$ 14,90, enquanto usuário com R$ 0,00 foi aceito. | `Usuario.java`: `temCreditosSuficientes` comparava `preco >= creditos`, invertendo a regra. | Corrigida a condição para `this.creditos >= preco`. | Regra de negócio, operadores relacionais e estado do objeto. |
| bug12 | Um filme indisponível foi alugado e reduziu os créditos de R$ 100,00 para R$ 90,10. | `Usuario.java`: o método `alugar` não verificava `Conteudo.isDisponivel()`. | Adicionada validação inicial que lança `ConteudoIndisponivelException` antes de qualquer débito. | Regra de negócio, exceções e preservação de estado. |

## Parte 2 — Ajustes de Clean Code

| # | Onde estava | Qual princípio/boas práticas era violado | O que eu mudei |
|---|---|---|---|
| clean01 | `Conteudo.duracaoMinutos` e seus acessos no controller | O atributo público quebrava o encapsulamento e permitia acesso direto ao estado interno. | O atributo passou a ser privado e o controller passou a usar `getDuracaoMinutos()`. |
| clean02 | Campos de repository nos três controllers | A injeção por campo com `@Autowired` escondia dependências e dificultava testes isolados. | Os repositories passaram a ser campos `final` recebidos por construtores explícitos. |
| clean03 | Método `Usuario.alugar()` | Os nomes `c` e `p` não comunicavam o papel das variáveis na regra de aluguel. | Renomeados para `conteudo` e `precoAluguel`, preservando todo o comportamento. |
| clean04 | `ConteudoController.listarPorCategoria()` | O controller buscava todos os registros e executava uma filtragem manual, duplicando uma responsabilidade do repository. | A consulta passou a chamar diretamente `conteudoRepository.findByCategoria(categoria)`. |
| clean05 | Método privado `calcularDescontoAntigo()` em `ConteudoController` | O método não possuía chamadas e mantinha uma regra antiga fora do contrato atual. | Removidos o método morto e seu comentário explicativo. |
| clean06 | Final de `ConteudoController` | Um bloco de cupom permanecia comentado como possível código futuro, gerando ruído e referências inexistentes. | Removidos o TODO e o código comentado; uma futura regra de cupons deverá ser implementada e versionada quando definida. |

---

## Parte 3 — Perguntas de reflexão

### 1. Injeção de dependência (Aula 13)
Os controllers recebem os repositories via `@Autowired` (ex.: `ConteudoController`
usa `ConteudoRepository`). Explique por que o Spring precisa gerenciar esses objetos
em vez de criarmos com `new ConteudoRepository()`. O que exatamente o Spring faz ao
injetar um bean, e por que isso não funcionaria com um `new` comum?

No código atual, o `ConteudoController` recebe o `ConteudoRepository` pelo construtor.
Esse repository é uma interface, portanto nem seria possível criar sua instância diretamente com `new`.
Ao iniciar a aplicação, o Spring Data JPA gera uma implementação dessa interface e registra esse objeto como bean.
Depois, o Spring encontra o construtor do controller e injeta nele a instância gerenciada do repository.
Essa implementação também recebe a infraestrutura necessária para acessar o banco, como o `EntityManager`.
Se criássemos um objeto manualmente, ele ficaria fora do ciclo de vida e das configurações mantidas pelo Spring.
A injeção pelo construtor ainda deixa a dependência explícita e facilita substituí-la em testes isolados.

### 2. JDBC vs Spring Data JPA (Aulas 12 e 13)
Na Aula 12 escrevemos um `ProdutoDAO` na mão com `Connection`, `PreparedStatement` e
`ResultSet`. Aqui o `ConteudoRepository` tem 2 linhas e faz CRUD completo. Compare as
duas abordagens: o que o Spring Data JPA automatiza, o que o JDBC/DAO ainda resolve
melhor, e como o `findByCategoria` consegue funcionar sem implementação.

No JDBC, nós escrevemos o SQL e controlamos manualmente a conexão, os parâmetros e a leitura do `ResultSet`.
Também precisamos converter cada linha retornada em objeto e garantir o fechamento correto dos recursos.
No projeto, `ConteudoRepository` herda de `JpaRepository`, que já fornece operações como `save`, `findAll` e `findById`.
O JPA ainda faz o mapeamento entre as entidades Java e as tabelas, reduzindo bastante o código repetitivo.
O JDBC ou um DAO próprio pode ser melhor quando precisamos de SQL muito específico, otimizações ou consultas complexas.
O método `findByCategoria` funciona porque o Spring Data interpreta seu nome e gera a consulta usando o atributo `categoria`.
Assim, o controller consulta diretamente por categoria sem conhecer SQL nem executar uma filtragem manual.

### 3. Exceções checked vs unchecked (Aula 11)
A `ClassificacaoIndicativaException` estourava como um erro genérico do servidor,
sem mensagem útil para o cliente. Explique a diferença entre `extends Exception` e
`extends RuntimeException` no contexto desse bug, e como você fez a mensagem da
regra (classificação indicativa) chegar de forma clara ao cliente da API.

Uma exceção que estende `Exception` é checked, então o compilador exige que ela seja tratada ou declarada com `throws`.
Quando ela estende `RuntimeException`, torna-se unchecked e pode atravessar as camadas sem obrigar cada método a capturá-la.
Nesse projeto, a classificação indicativa representa uma violação de regra de negócio durante o aluguel.
Por isso, `ClassificacaoIndicativaException` foi alterada para estender `RuntimeException`.
O `throws` ainda presente em `Usuario.alugar()` serve como documentação, mas não é obrigatório para uma unchecked exception.
No `GlobalExceptionHandler`, um `@ExceptionHandler` específico captura essa exceção e retorna o status HTTP 403.
O corpo da resposta usa `e.getMessage()`, fazendo a mensagem com idade, título e classificação chegar ao cliente.

### 4. Sobrescrita vs sobrecarga (Aula 7)
Um dos bugs compilava sem nenhum erro: o método da `Serie` parecia sobrescrever
`calcularPrecoAluguel`, mas na verdade sobrecarregava. Explique a diferença entre
override e overload nesse caso e por que a anotação `@Override` teria impedido o bug.

### 5. Onde blindar o objeto? (Aulas 3, 4 e 13)
Vimos bugs de dados inválidos aceitos (duração negativa, créditos negativos, campos
nulos). Em quais lugares (construtor, setter, método do model) cada tipo de validação
deve ficar? Justifique usando os bugs que você encontrou e explique por que validar só
em um lugar não foi suficiente.

### 6. Abstração e interface (Aulas 8 e 9)
`Conteudo` é abstrata e `Promocionavel` é uma interface. Explique a diferença de
propósito entre as duas nesse projeto e o que mudaria no código se o Documentário
passasse a ter promoções — quais classes/linhas seriam tocadas e quais ficariam
intactas? O que isso diz sobre o design do sistema?

---


```
