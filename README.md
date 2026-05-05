<div align="center">

<img src="docs/git-craft_logo.png" alt="Git-Craft" height="72">

<br/>

**Template para criar servidores Minecraft em GitHub Codespaces.**

[![úteis](https://img.shields.io/badge/gerenciado_por-úteis-black?style=flat-square)](https://github.com/CaioJohnston/uteis)

</div>

---

## Estrutura

```
.devcontainer/
  devcontainer.json   # Config do Codespace: imagem, portas, startup
startup.sh            # Script que baixa e inicia o servidor
control-server.js     # HTTP + SSE + WebSocket API para console remoto
package.json          # Dependências do control-server (ws)
```

## Como usar

1. Acesse **Git-Craft** no site úteis
2. Conecte sua conta GitHub
3. Escolha o tipo de servidor, versão e máquina
4. Um Codespace é criado automaticamente com este template

O `startup.sh` roda sozinho via `postStartCommand` do devcontainer:

- Auto-atualiza do git ao iniciar
- Instala a versão correta do Java para a versão do Minecraft
- Baixa o JAR do servidor (Vanilla / Paper / Fabric / Forge / CurseForge)
- Aceita o EULA automaticamente
- Inicia o servidor em `tmux` (session: `mc`)
- Configura túnel TCP via playit.gg (com fallback para bore)
- Sobe o control server na porta `8081` para acesso ao console

## Portas

| Porta | Uso                                            |
| ----- | ---------------------------------------------- |
| 25565 | Minecraft Server (conexão dos jogadores)      |
| 8081  | Git-Craft Console API (HTTP + SSE + WebSocket) |

## API do Control Server (porta 8081)

| Método | Rota        | Auth                  | Descrição                                                     |
| ------- | ----------- | --------------------- | --------------------------------------------------------------- |
| GET     | `/status` | —                    | Estado: running, stage, config, IP do servidor, RAM, cmd_secret |
| GET     | `/log`    | —                    | Últimas N linhas do log (`?lines=200`)                       |
| GET     | `/sse`    | —                    | Server-Sent Events — streaming de log em tempo real            |
| GET     | `/config` | —                    | Lê configuração salva                                        |
| POST    | `/cmd`    | `X-Minehost-Secret` | Envia comando ao servidor (`{"command": "op me"}`)            |
| POST    | `/config` | —                    | Salva configuração (`{"type":"paper","version":"latest"}`)  |
| WS      | `/ws`     | —                    | WebSocket para streaming do console + envio de comandos         |

Requisições para `/cmd` precisam do header `X-Minehost-Secret` com o valor retornado em `/status` (`cmd_secret`).

## Configuração (`minehost.json`)

Criado automaticamente em `server/minehost.json`:

```json
{
  "type": "paper",
  "version": "latest",
  "jvmArgs": "-Xmx2048m -Xms1024m"
}
```

## Tipos de servidor suportados

- **vanilla** — servidor oficial Mojang
- **paper** — otimizado com suporte a plugins Bukkit
- **fabric** — moderno com sistema de mods leves
- **forge** — mods completos
- **curseforge** — modpacks do CurseForge (via link do .zip do serverfile)

## Variáveis de ambiente

Injetadas pelo úteis ao criar o Codespace:

| Variável               | Descrição                                                                      |
| ----------------------- | -------------------------------------------------------------------------------- |
| `MINEHOST_GIST_ID`    | ID do Gist para sincronização de estado                                        |
| `MINEHOST_TOKEN`      | Token GitHub para leitura/escrita do Gist                                        |
| `MINEHOST_CMD_SECRET` | Secret para autenticar comandos via `/cmd` (gerado automaticamente se ausente) |
| `SERVER_DIR`          | Diretório do servidor (padrão:`server/`)                                     |

## Sincronização via Gist

Quando `MINEHOST_GIST_ID` e `MINEHOST_TOKEN` estão definidos, o control-server sincroniza o estado a cada 5 segundos com um arquivo `state.json` no Gist. Isso permite ao frontend do úteis acompanhar o status e enviar comandos sem depender de port forwarding direto.

## Túnel TCP

O `startup.sh` expõe a porta 25565 publicamente via túnel TCP:

1. **playit.gg** — preferencial; solicita autenticação na primeira execução
2. **bore** — fallback automático se o playit.gg não estiver disponível

O endereço do servidor é escrito em `.server_ip` e exposto via `/status`.

## Nota

Este repo é gerenciado pelo Git-Craft. Modificações manuais são possíveis no Codespace, mas podem ser sobrescritas ao recriar o servidor.
