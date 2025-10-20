# Executando o simulador

## Background

Pelo que eu entendi, o simulador tem (no mínimo) três partes:
- Access & Mobility Management Function (AMF)
  - Parte do free5gc, simula a parte de acesso ao core do 5G (o que é o "core"? Não sei!)
- UERANSIM gNodeB (gNB)
  - Simula a "torre" 5G
- UERANSIM user agent (UE)
  - Simula o usuário se conectando a alguma torre

Ainda não sei muito bem onde o NWDAF se conecta nisso.
Pelo que eu entendi, o `free5gc` nem implementa algum NWDAF, e os trabalhos que usam NWDAF com `free5gc` o implementam na mão.

## Executando

Vamos executar todos os serviços em contêineres Docker.

Leia os pré-requisitos disponíveis em https://github.com/edusporto/free5gc-compose.
Eles são:
- instalação do GTP5G kernel module (ver o repositório)
- instalação do Docker
- instalação do Docker Compose

Depois, clone o repositório (use minha versão levemente modificada):
```
git clone https://github.com/edusporto/free5gc-compose
cd free5gc-compose

cd base
git clone --recursive -j `nproc` https://github.com/free5gc/free5gc.git
cd ..
```

Construa as imagens:
```
make all
docker compose -f docker-compose-build.yaml build
```

Finalmente, execute os serviços:
```
docker compose -f docker-compose-build.yaml up
```

Este comando iniciará todos os serviços Docker listados em `docker-compose-build.yaml`. Isto inclui o gNB do UERANSIM e todas as network functions.

Para conectar o user agent, temos que registrá-lo usando o Webconsole do `free5gc`.
Caso esteja conectado a um servidor ssh, crie uma nova janela do terminal e conecte novamente colocando no final a flag `-L 5000:localhost:5000`. Isto fará forwarding da porta 5000 do servidor ao seu cliente. (repita este processo caso realize múltiplas conexões ssh seguidas)

Feito isso, acesse `localhost:5000` em seu navegador favorito. Vá para a aba Subscribers e crie um novo subscriber. As configurações devem ser as mesmas ao que está no arquivo `config/uecfg.yaml`. No meu caso, as configurações já eram as mesmas por padrão, então provavelmente você não precisará mexer nisso.

Finalmente, podemos conectar o user agent. Este user agent estará dentro do próprio contêiner do UERANSIM gNB. Acho que seria mais adequado que estivesse em um contêiner separado, mas podemos nos preocupar com isso na versão final do artigo.

No terminal novo que você criou na etapa anterior, acesse o shell do serviço do UERANSIM:
```
docker exec -it ueransim bash
```

Depois, inicie o user agent:
```
root@host:/ueransim# ./nr-ue -c config/uecfg.yaml
```

Pronto!

Para verificar se funcionou, abra outro terminal conectado ao contêiner do user agent (`docker exec -it ueransim bash`).
Se estiver funcionando, deve haver uma interface de rede `uesimtun0`. Podemos testar se ela consegue se conectar à internet com `ping -I uesimtun0 google.com`.
