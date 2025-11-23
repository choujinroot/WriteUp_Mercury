# WriteUp_Mercury
olá! aqui é o Caio, aka Choujin, e hoje detalharei como consegui resolver a máquina vulnerável Mercury, da série The Planets (resolvi começar por Mercúrio pra seguir a ordem (eu espero que Mercúrio seja o primeiro planeta do sistema))

### Primeira etapa: Varredura
visto que a Mercury não expõe o próprio ip, fiz uma varredura na rede utilizando do comando **sudo arp-scan -l** e obtive o seguinte resultado:
<img width="638" height="172" alt="image" src="https://github.com/user-attachments/assets/6e818a41-b50d-4587-84fb-b4913a670c44" />

depois de alguns testes, vi que o final 103 seria a vítima 

### Segunda etapa: Enumeração
com o ip em mãos, parti pra enumeração de serviços com o comando **namp -sC -sV -O -p- [ip alvo]** 

-sC: utiliza de scripts NSE padrão

-sV: checagem de versão dos serviços disponíveis

-O: checagem de sistema operacional

-p-: checa todas as portas

obtive então:

<img width="660" height="310" alt="image" src="https://github.com/user-attachments/assets/3dc7f565-91f9-423a-95d2-c2dabfb99945" />

uma porta ssh e uma http ambas abertas!!!

### Terceira Etapa: Exploit >:^)

na descrição do desafio deixava bem explícito que não seria necessário um brute force, então parti para o http pra ver do que se tratava e me deparei com a seguinte cena:

<img width="682" height="163" alt="image" src="https://github.com/user-attachments/assets/b97cb3dc-dc87-4e0d-993f-e96aa3723c82" />

nada, absolutamente nada.

mas aí decidi testar alguns diretórios padrão e aí sim me deparei com algo interessante

<img width="658" height="200" alt="image" src="https://github.com/user-attachments/assets/4a7d1ef5-bbc5-4163-b838-9b138ce74801" />

um diretório chamado "mercuryfacts"

<img width="703" height="592" alt="image" src="https://github.com/user-attachments/assets/85c2b2dc-f4a8-4e69-826e-5cdf27bb0897" />

e, acreditem ou não, mas ao clicar vamos à uma página contendo um fato sobre Mercúrio, cada um contendo um id

além disso, ao clicar em "see list" somos levados à to-do list do dev, e lá nos deparamos com um dos itens falando sobre banco de dados:

<img width="461" height="130" alt="image" src="https://github.com/user-attachments/assets/a453ef74-961f-4dbf-9c9e-5403cbdafc76" />

pensei então: "aqui deve ter um banco de dados em sql", rodei **sqlmap -u [url alvo] --dbs --batch**

--dbs: lista as tabelas disponíveis no banco de dados

--batch: ignora as perguntas feitas pela ferramenta

<img width="372" height="135" alt="image" src="https://github.com/user-attachments/assets/dadf1b1f-cb50-406c-b721-0896be0bd2c0" />

capturamos dois bancos de dados, então prossegui com **sqlmap -u [url alvo] --dump-all --batch**

--dump-all: captura todas as linhas de todas as tabelas disponíveis

<img width="771" height="437" alt="image" src="https://github.com/user-attachments/assets/fcf5da11-fcf7-48e2-af38-2377eb6463a3" />

agooora sim temos algo pra trabalhar, um username e uma senha 

tentei logar com ssh webmaster@[ip alvo] e ao inserir a senha estamos dentro!!! com um ls descobrimos a primeira flag

<img width="471" height="91" alt="image" src="https://github.com/user-attachments/assets/714f7e4d-b1cd-4c3a-a537-796bd674c069" />

prosseguimos com um cd no diretório mercur_proj e com um ls descobrimos um arquivo notes.txt

<img width="731" height="101" alt="image" src="https://github.com/user-attachments/assets/ea5155bf-2add-48b3-96ff-62783d6e9595" />

aqui teremos que lidar com base64, apesar de ter algumas ferramentas no terminal que decodificariam, eu utilizei de um decoder na internet, use o que for de sua preferência

após decodificado, temos a senha do usuário linuxmaster, então usamos **su linuxmaster** (ou todo o processo de logar com ssh igual eu porque esqueci que dava pra fazer assim) e a senha, e estamos dentro

depois de alguns ls sem resultado, rodei **sudo -l** pra checar quais comandos esse usuário podia rodar e me deparei com um shell script chamado check_syslog no caminho /usr/bin/check_syslog.sh

a partir daqui eu tive que me aprofundar um pouco nos conhecimentos de linux, confesso que escapou um pouco do meu conhecimento, o que na verdade é muito bom, sempre vai me ser bem vindo um conhecimento a mais em linux e cibersegurança

com um head descobri que o script rodava o comando tail para ler as últimas 10 entradas do syslog

<img width="501" height="61" alt="image" src="https://github.com/user-attachments/assets/b0f3e3f1-b0af-4783-9612-e417e330589d" />

 adicionei o diretório atual ao PATH com **export PATH="$(pwd):$PATH"** e criei um link simbólico pro vim chamado tail com **ln -s /usr/bin/vim tail** 

 por fim, rodei o comando **sudo –preserve-env=PATH /usr/bin/check_syslog.sh** pra entrar no vim como superuser e usei **:!/bin/sh** pra entrar no shell como superusuário (:! é usado no vim pra rodar comandos externos de terminal), após vasculhar um pouco chegamos ao arquivo root_flag no diretório root

 <img width="631" height="506" alt="image" src="https://github.com/user-attachments/assets/f4f963a5-ca24-47d3-8eca-52e83e9c80f1" />

 e com isso, enfim contemplamos o laboratório!!! :^) 

 devo dizer que foi um laboratório muito proveitoso, eu nunca vou deixar de gostar dessa sensação de um quebra-cabeças sendo formado peça por peça que o pentest traz e apesar de ser catalogado como fácil, esse laboratório me forçou a aprofundar meus conhecimentos de linux e escalonamento de privilégios com a utilização do ambiente preservado e do link simbólico, conceitos que até então eu não tinha visto/utilizado em minhas abordagens. Agradeço aos que tiraram um tempo pra ler, até uma próxima, ciaao :^)



























