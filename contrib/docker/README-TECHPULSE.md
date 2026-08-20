# Implantação TechPulseSP — uso interno

Este fork existe para hospedar o WorkAdventure **internamente** na TechPulseSP.
Nada aqui altera o comportamento do upstream: a adaptação vive num único arquivo
de override, `contrib/docker/docker-compose.techpulse.yaml`.

## Licença — leia antes de mudar o uso

O WorkAdventure é **AGPL-3.0 com a Commons Clause**. Duas consequências práticas:

- **Commons Clause:** não é permitido *vender* — o texto define "Sell" como
  fornecer a terceiros, por remuneração ou outra contrapartida, um produto ou
  serviço cujo valor derive inteira ou substancialmente da funcionalidade do
  software. **Uso interno da própria empresa não é atingido.** Oferecer isto
  como serviço a clientes exigiria licença comercial da TheCodingMachine.
- **AGPL-3.0:** ao modificar e dar acesso pela rede, os usuários têm direito ao
  código-fonte correspondente. Como este fork é público, isso já está atendido.

Se o uso deixar de ser interno, revise as duas cláusulas antes.

## Como subir

```bash
cd contrib/docker
cp .env.prod.template .env        # o .gitignore já ignora o .env
# preencher no mínimo:
#   SECRET_KEY                            (openssl rand -hex 32)
#   DOMAIN                                (nome de domínio, NÃO um IP)
#   ACME_EMAIL
#   VERSION                               (fixar uma tag, ex. v1.33.2)
#   MAP_STORAGE_AUTHENTICATION_USER
#   MAP_STORAGE_AUTHENTICATION_PASSWORD

docker compose -f docker-compose.prod.yaml -f docker-compose.techpulse.yaml up -d
```

## Requisitos que não são negociáveis

**Domínio, não IP.** O upstream avisa: WebRTC exige HTTPS com certificado
válido, e certificado não é emitido para endereço IP. Um serviço de DNS curinga
(`sslip.io`, `nip.io`) resolve isso sem registrar domínio, porque entrega um
**nome** válido.

**`VERSION` fixa.** O upstream alerta que a tag `master` evolui e pode passar a
exigir variáveis de ambiente novas sem aviso. Fixe uma versão e atualize
deliberadamente.

**Credenciais do map-storage.** `MAP_STORAGE_ENABLE_BASIC_AUTHENTICATION` vem
`true` por padrão e o serviço **não sobe** sem usuário e senha — ele falha com
`Missing AUTHENTICATION_USER` e entra em loop de reinício. O sintoma é
enganoso: o Traefik ignora container reiniciando, então os routers do
map-storage simplesmente não aparecem e a aplicação abre sem mapas.

## O que esta instalação NÃO tem

Sem **LiveKit** e sem **Coturn**, ambos por exigirem portas UDP:

| Faltando | Efeito, conforme a documentação do upstream |
|---|---|
| LiveKit | áudio/vídeo só peer-to-peer, em bolhas de até **4 pessoas** |
| Coturn | cerca de **15%** dos usuários (em redes que bloqueiam P2P) não estabelecem áudio/vídeo |

Também fica de fora o `room-api`: ele pede um entrypoint gRPC que o Traefik do
host não define. É a API de automação externa — opcional.

O mundo 2D, a movimentação, o chat de texto e vídeo em grupos pequenos
funcionam. Reuniões grandes, não.

## Atualizando a partir do upstream

```bash
git remote add upstream https://github.com/workadventure/workadventure.git   # uma vez
git fetch upstream
git merge upstream/master
```

Como a adaptação está isolada no arquivo de override, o merge não deve
conflitar. Depois do merge, confira se o upstream introduziu variáveis novas em
`contrib/docker/.env.prod.template` e replique no seu `.env`.
