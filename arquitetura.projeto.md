1. O projeto separa responsabilidades em Model, Repository, Controller e Exception. O Model concentra as regras de negócio de conteúdos, preços, promoções e aluguel; 

2. Os Repositories fazem a persistência com Spring Data JPA;

3. Os Controllers recebem as requisições HTTP; 

4. O tratamento global de exceções transforma erros de domínio em respostas da API.
