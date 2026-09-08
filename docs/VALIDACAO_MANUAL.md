# Roteiro de validação manual — StreamFIAP

Este roteiro permite reproduzir os testes finais da API em ambiente local. Os comandos foram preparados para Windows PowerShell e consideram uma inicialização limpa do banco H2 em memória.

> A configuração H2 deve ser usada somente para teste local. Não publique o application.properties alterado nem substitua os placeholders do Oracle no repositório.

## 1. Preparar o banco H2 local

Abra src/main/resources/application.properties.

Comente temporariamente as quatro propriedades do Oracle:

~~~properties
#spring.datasource.url=jdbc:oracle:thin:@oracle.fiap.com.br:1521:ORCL
#spring.datasource.driverClassName=oracle.jdbc.OracleDriver
#spring.datasource.username=SEU_RM
#spring.datasource.password=SUA_SENHA
~~~

Descomente o bloco H2:

~~~properties
spring.datasource.url=jdbc:h2:mem:streamfiap
spring.datasource.driverClassName=org.h2.Driver
spring.datasource.username=sa
spring.datasource.password=
~~~

Salve o arquivo com Ctrl+S.

## 2. Iniciar a aplicação no Eclipse

1. Abra StreamFiapApplication.java.
2. Clique com o botão direito no editor.
3. Selecione **Run As > Java Application**.
4. Aguarde o Console apresentar mensagens equivalentes a:

~~~text
Tomcat started on port 8080
Started StreamFiapApplication
~~~

Mantenha a aplicação aberta. Execute os comandos seguintes em uma nova janela do PowerShell.

## 3. Validar duração inválida

Monte o JSON e grave-o temporariamente em arquivo para preservar corretamente as aspas:

~~~powershell
$invalidBody = @{
    titulo = "Filme Invalido"
    categoria = "TESTE"
    duracaoMinutos = 0
    classificacaoEtaria = 0
    disponivel = $true
    estreia = $false
} | ConvertTo-Json

$invalidBody | Set-Content "$env:TEMP\filme-invalido.json" -Encoding UTF8

curl.exe -i -X POST "http://localhost:8080/api/conteudos/filme" -H "Content-Type: application/json" --data-binary "@$env:TEMP\filme-invalido.json"
~~~

Resultado esperado:

- status HTTP 400;
- corpo {"erro":"A duração deve ser maior que zero"};
- nenhum conteúdo salvo.

## 4. Cadastrar os conteúdos válidos

### 4.1 Filme

~~~powershell
$filmeBody = @{
    titulo = "Matrix"
    categoria = "FICCAO"
    duracaoMinutos = 136
    classificacaoEtaria = 14
    disponivel = $true
    estreia = $true
} | ConvertTo-Json

$filme = Invoke-RestMethod -Method Post -Uri "http://localhost:8080/api/conteudos/filme" -ContentType "application/json" -Body $filmeBody
$filme | ConvertTo-Json
~~~

Resultado esperado: filme cadastrado com id=1, título Matrix e todos os dados preservados.

~~~powershell
$precoFilme = Invoke-RestMethod -Method Get -Uri "http://localhost:8080/api/conteudos/$($filme.id)/preco-promocional"
"PRECO PROMOCIONAL DO FILME: $precoFilme"
~~~

Resultado esperado: 11.92. Podem aparecer casas decimais adicionais por causa da representação de double.

### 4.2 Série

~~~powershell
$serieBody = @{
    titulo = "Dark"
    categoria = "FICCAO"
    duracaoMinutos = 60
    classificacaoEtaria = 16
    disponivel = $true
    numeroTemporadas = 5
} | ConvertTo-Json

$serie = Invoke-RestMethod -Method Post -Uri "http://localhost:8080/api/conteudos/serie" -ContentType "application/json" -Body $serieBody
$serie | ConvertTo-Json
~~~

Resultado esperado: série com id=2, dados herdados preenchidos e cinco temporadas.

~~~powershell
$precoSerie = Invoke-RestMethod -Method Get -Uri "http://localhost:8080/api/conteudos/$($serie.id)/preco-promocional"
"PRECO PROMOCIONAL DA SERIE: $precoSerie"
~~~

Resultado esperado: 19.6.

### 4.3 Documentário

~~~powershell
$documentarioBody = @{
    titulo = "Planeta Terra"
    categoria = "DOCUMENTARIO"
    duracaoMinutos = 90
    classificacaoEtaria = 0
    disponivel = $false
    tema = "Natureza"
} | ConvertTo-Json

$documentario = Invoke-RestMethod -Method Post -Uri "http://localhost:8080/api/conteudos/documentario" -ContentType "application/json" -Body $documentarioBody
$documentario | ConvertTo-Json
~~~

Resultado esperado: documentário com id=3, tema Natureza e indisponível.

~~~powershell
$precoDocumentario = Invoke-RestMethod -Method Get -Uri "http://localhost:8080/api/conteudos/$($documentario.id)/preco-promocional"
"PRECO DO DOCUMENTARIO: $precoDocumentario"
~~~

Resultado esperado: 0.0.

## 5. Validar listagem e busca por categoria

~~~powershell
$conteudos = Invoke-RestMethod -Method Get -Uri "http://localhost:8080/api/conteudos"
$conteudos | ConvertTo-Json -Depth 5

$ficcao = Invoke-RestMethod -Method Get -Uri "http://localhost:8080/api/conteudos/categoria/FICCAO"
$ficcao | Select-Object id, titulo, categoria
"QUANTIDADE FICCAO: $($ficcao.Count)"
~~~

Resultado esperado: a listagem contém Matrix, Dark e Planeta Terra. A busca por FICCAO retorna somente Matrix e Dark, com quantidade 2.

## 6. Cadastrar os usuários de teste

~~~powershell
$usuarioCreditoBody = @{
    nome = "Rafael"
    idade = 20
    creditos = 100.0
} | ConvertTo-Json
$usuarioCredito = Invoke-RestMethod -Method Post -Uri "http://localhost:8080/api/usuarios" -ContentType "application/json" -Body $usuarioCreditoBody

$usuarioSemCreditoBody = @{
    nome = "Usuario Sem Credito"
    idade = 20
    creditos = 0.0
} | ConvertTo-Json
$usuarioSemCredito = Invoke-RestMethod -Method Post -Uri "http://localhost:8080/api/usuarios" -ContentType "application/json" -Body $usuarioSemCreditoBody

$usuarioMenorBody = @{
    nome = "Usuario Menor"
    idade = 12
    creditos = 100.0
} | ConvertTo-Json
$usuarioMenor = Invoke-RestMethod -Method Post -Uri "http://localhost:8080/api/usuarios" -ContentType "application/json" -Body $usuarioMenorBody

$usuarioCredito, $usuarioSemCredito, $usuarioMenor | Select-Object id, nome, idade, creditos
~~~

Resultado esperado: IDs gerados automaticamente e todos os dados preservados.

## 7. Validar conteúdo indisponível

Use o usuário com créditos e o documentário indisponível:

~~~powershell
curl.exe -i -X POST "http://localhost:8080/api/alugueis?usuarioId=$($usuarioCredito.id)&conteudoId=$($documentario.id)"
~~~

Resultado esperado:

- HTTP 409;
- mensagem informando que Planeta Terra não está disponível;
- créditos do usuário continuam em 100.0.

Confirme o saldo:

~~~powershell
Invoke-RestMethod -Method Get -Uri "http://localhost:8080/api/usuarios/$($usuarioCredito.id)"
~~~

## 8. Validar classificação indicativa

~~~powershell
curl.exe -i -X POST "http://localhost:8080/api/alugueis?usuarioId=$($usuarioMenor.id)&conteudoId=$($filme.id)"
~~~

Resultado esperado:

- HTTP 403;
- mensagem informando que o usuário de 12 anos não pode assistir ao filme de classificação 14;
- créditos e disponibilidade permanecem inalterados.

## 9. Validar créditos insuficientes

~~~powershell
curl.exe -i -X POST "http://localhost:8080/api/alugueis?usuarioId=$($usuarioSemCredito.id)&conteudoId=$($filme.id)"
~~~

Resultado esperado:

- HTTP 422;
- mensagem Créditos insuficientes para alugar Matrix;
- saldo continua em 0.0;
- Matrix continua disponível.

## 10. Validar aluguel bem-sucedido

Este teste deve ser executado depois dos cenários de recusa, pois torna o filme indisponível.

~~~powershell
$aluguelValido = Invoke-RestMethod -Method Post -Uri "http://localhost:8080/api/alugueis?usuarioId=$($usuarioCredito.id)&conteudoId=$($filme.id)"
$aluguelValido | ConvertTo-Json

$usuarioDepoisAluguel = Invoke-RestMethod -Method Get -Uri "http://localhost:8080/api/usuarios/$($usuarioCredito.id)"
$filmeDepoisAluguel = Invoke-RestMethod -Method Get -Uri "http://localhost:8080/api/conteudos/$($filme.id)"

"CREDITOS RESTANTES: $($usuarioDepoisAluguel.creditos)"
"MATRIX DISPONIVEL: $($filmeDepoisAluguel.disponivel)"
~~~

Resultado esperado:

- aluguel concluído com HTTP 200;
- débito de R$ 14,90;
- saldo final de 85.1;
- Matrix com disponivel=false.

## 11. Validar conteúdo inexistente

~~~powershell
curl.exe -i -X GET "http://localhost:8080/api/conteudos/999"
~~~

Resultado esperado:

- HTTP 404;
- corpo {"erro":"Conteúdo não encontrado: 999"}.

## 12. Encerrar e restaurar a configuração

1. Pare a aplicação pelo botão vermelho **Terminate** no Console do Eclipse.
2. Desfaça somente a alteração local do application.properties:
   - reative o bloco Oracle com placeholders;
   - volte a comentar o bloco H2.
3. Confira no Git que o arquivo de propriedades não entrou em nenhum commit.

Como o H2 está em memória, todos os registros de teste são apagados quando a aplicação é encerrada.

## Checklist final

- [x] Aplicação inicia na porta 8080.
- [x] Filme, série e documentário são persistidos.
- [x] IDs de usuários são gerados.
- [x] Preços e promoção seguem o contrato.
- [x] Busca por categoria retorna os registros corretos.
- [x] Duração inválida retorna 400.
- [x] Classificação indicativa retorna 403.
- [x] Conteúdo indisponível retorna 409.
- [x] Créditos insuficientes retornam 422.
- [x] Conteúdo inexistente retorna 404.
- [x] Recusas não alteram créditos nem disponibilidade.
- [x] Aluguel válido debita créditos e torna o conteúdo indisponível.
