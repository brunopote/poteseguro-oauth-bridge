# Ponte de retorno do OAuth — PoteSeguro

Uma página estática, e só.

## Por que existe

O Mod_Security da hospedagem do `poteseguro.com.br` recusa com **406** toda
requisição cujo parâmetro de query tenha valor começando por `https://` — GET
ou POST, arquivo estático ou dinâmico. Medido:

    ?x=https://example.com    -> 406
    ?code=…&state=…           -> 302  (passa)

O Google **sempre** devolve `iss=https://accounts.google.com` no retorno do
OAuth. Resultado: o consentimento acontecia, o Google redirecionava, e o
retorno morria antes de chegar ao servidor. Nenhum token era gravado.

Não há como pedir ao Google que não envie o `iss`, e o cPanel desta conta não
expõe controle do Mod_Security.

## O que esta página faz

Recebe o retorno do Google, descarta tudo o que a regra rejeita e reencaminha
para o sistema apenas `code`, `state` e `error` — que passam.

## O que ela NÃO faz

Não decide nada e não guarda nada. O `state` continua assinado com HMAC e
conferido no servidor; o `code` é de uso único e trocado por token pelo
back-end do PoteSeguro.

## Quando apagar

Assim que a hospedagem liberar a regra do Mod_Security para o domínio. Aí o
retorno do Google volta a apontar direto para
`https://poteseguro.com.br/broker/google-oauth/callback` e este repositório
deixa de ser necessário.
