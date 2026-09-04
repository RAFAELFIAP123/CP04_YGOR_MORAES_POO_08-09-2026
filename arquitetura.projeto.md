**Resumo Geral:**

1. O projeto separa responsabilidades em Model, Repository, Controller e Exception. O Model concentra as regras de negócio de conteúdos, preços, promoções e aluguel; 

2. Os Repositories fazem a persistência com Spring Data JPA;

3. Os Controllers recebem as requisições HTTP; 

4. O tratamento global de exceções transforma erros de domínio em respostas da API.



**Model e POO**

Conteudo é a abstração principal. Filme, Serie e Documentario são especializações. Promocionavel
representa uma capacidade comum às classes que participam da promoção. Usuario representa quem
realiza o aluguel. Esse desenho permite aplicar herança, abstração, encapsulamento e polimorfismo.

**Repository**

ConteudoRepository e UsuarioRepository permitem consultar e persistir entidades sem que o Controller
precise implementar diretamente a lógica de acesso ao banco.

**Controller**

ConteudoController, UsuarioController e AluguelController são a entrada HTTP da aplicação. O Controller
deve coordenar a requisição sem absorver indevidamente as regras de negócio que pertencem ao Model.

**Exceptions**

As exceções customizadas representam falhas de negócio, como conteúdo inexistente, conteúdo
indisponível, idade incompatível e créditos insuficientes. O GlobalExceptionHandler centraliza a
transformação dessas exceções em respostas para o cliente
