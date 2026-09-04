O projeto separa responsabilidades em Model, Repository, Controller e Exception. O Model concentra as
regras de negócio de conteúdos, preços, promoções e aluguel; os Repositories fazem a persistência com
Spring Data JPA; os Controllers recebem as requisições HTTP; e o tratamento global de exceções
transforma erros de domínio em respostas da API.
