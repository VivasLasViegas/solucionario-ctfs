A última máquina estava em uma das redes internas da máquina 01 (`100.64.2.0/24`), porém, precisava de pivoteamento no host 1 para torná-la acessível.

Após o pivoting ser feito via `ligolo-ng` conseguimos acessar sua página Web.

![](Pasted%20image%2020260825112228.png)

Até então, um sistema de login do Sisbol. Após escanear possíveis diretórios encontramos a seguinte pista:

![](Pasted%20image%2020260825112608.png)

Credenciais de acesso. Logamos no portal e começamos a mexer nas configurações.

Após um longo tempo de buscas encontramos a seguinte requisição via `BurpSuite`

![](Pasted%20image%2020260826132527.png)

O parâmetro `filename=` é vulnerável a `LFI`, portanto, podemos fazer a leitura de arquivos do sistema com a permissão do `www-data`.

![](Pasted%20image%2020260825133500.png)

![](Pasted%20image%2020260825133911.png)

Com isso, encontramos a primeira flag.

A segunda flag é bem mais trabalhosa, durante uma das configurações alteradas podíamos definir "assinaturas" e "permissões" para cada tipo de usuário utilizar o boletim interno do Sisbol. Definindo a assinatura do usuário que possui permissão de assinar o documento, podemos explorar um possível vetor, que no caso a assinatura seria na verdade um documento malicioso como a sanitização do parâmetro de arquivos aceitos era frágil.

![](Pasted%20image%2020260825180528.png)

E logo abaixo o erro encontrado com um arquivo mal sanitizado.

![](Pasted%20image%2020260825172029.png)

Podemos inserir um arquivo `.php`e acessá-lo no navegador a fim de obter uma RevShell.

OBS: um erro que cometi aqui foi de colocar o meu IP do Kali como listener, falha, precisa ser o IP da máquina hospedeira do túnel (máquina 1) e, após isso, colocar um listener do ligolo nessa máquina a fim de fazer o redirecionamento pra porta reversa no meu Kali finalmente.

![](Pasted%20image%2020260826133210.png)

`[Agent] » listener_add --addr 0.0.0.0:4444 --to 127.0.0.1:4444 --tcp`

Ficando o fluxo: 

`100.64.2.201 → pivot:4444 → (ligolo) → Kali:4444`

Conseguimos estabelecer conexão, e temos uma credencial exposta num banco de dados antigo.

![](Pasted%20image%2020260825195638.png)

Fazendo a escalação com a credencial encontrada conseguimos a última flag do nosso CTF.

![](Pasted%20image%2020260825195919.png)

