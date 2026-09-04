```
streamfiap-bughunt/
│
├── pom.xml
├── .gitignore
├── AVALIACAO_README_TEMPLATE.md
│
├── .mvn/
│   └── wrapper/
│
└── src/
    └── main/
        ├── java/
        │   └── br/com/fiap/streamfiap/
        │       │
        │       ├── StreamFiapApplication.java
        │       │
        │       ├── model/
        │       │   ├── Conteudo.java
        │       │   ├── Filme.java
        │       │   ├── Serie.java
        │       │   ├── Documentario.java
        │       │   ├── Usuario.java
        │       │   └── Promocionavel.java
        │       │
        │       ├── repository/
        │       │   ├── ConteudoRepository.java
        │       │   └── UsuarioRepository.java
        │       │
        │       ├── controller/
        │       │   ├── ConteudoController.java
        │       │   ├── UsuarioController.java
        │       │   └── AluguelController.java
        │       │
        │       └── exception/
        │           ├── ClassificacaoIndicativaException.java
        │           ├── ConteudoNaoEncontradoException.java
        │           ├── ConteudoIndisponivelException.java
        │           ├── CreditosInsuficientesException.java
        │           └── GlobalExceptionHandler.java
        │
        └── resources/
            └── application.properties
