O projeto separa responsabilidades em Model, Repository, Controller e Exception. O Model concentra as regras de negócio de conteúdos, preços, promoções e aluguel; 

Os Repositories fazem a persistência com Spring Data JPA;

Os Controllers recebem as requisições HTTP; 

O tratamento global de exceções transforma erros de domínio em respostas da API.
