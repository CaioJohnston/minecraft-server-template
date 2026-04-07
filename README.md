# MineHost — Minecraft Server Template

Template repo para criar servidores Minecraft em GitHub Codespaces, gerenciados pelo [úteis.](https://github.com/CaioJohnston/uteis)

## Estrutura

```
.devcontainer/
  devcontainer.json   # Config do Codespace: imagem, portas, startup
startup.sh            # Script que baixa e inicia o servidor
control-server.js     # HTTP + WebSocket API para console remoto
package.json          # Dependências do control-server (ws)
```

## Como usar

1. Acesse **MineHost** no site úteis
2. Conecte sua conta GitHub
3. Escolha o tipo de servidor e versão
4. Um Codespace é criado automaticamente com este template

O `startup.sh` roda sozinho via `postStartCommand` do devcontainer:
- Baixa o JAR do servidor (Vanilla / Paper / Fabric / Forge)
- Aceita o EULA automaticamente
- Gera o mundo na primeira execução
- Inicia o servidor em `tmux` (session: `mc`)
- Sobe o control server na porta `8081` para acesso ao console

## Portas

| Porta | Uso |
|-------|-----|
| 25565 | Minecraft Server (conexão dos jogadores) |
| 8081  | MineHost Console API (HTTP + WebSocket) |

## API do Control Server (porta 8081)

| Método | Rota    | Descrição |
|--------|---------|-----------|
| GET    | `/status` | Estado do servidor + config atual |
| GET    | `/log`    | Últimas N linhas do log (`?lines=200`) |
| POST   | `/cmd`    | Envia comando ao servidor (`{"command": "op me"}`) |
| GET    | `/config` | Lê configuração salva |
| POST   | `/config` | Salva configuração (`{"type":"paper","version":"latest"}`) |
| WS     | `/ws`     | WebSocket para streaming do console em tempo real |

## Configuração (`minehost.json`)

Criado automaticamente em `/workspace/server/minehost.json`:

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

## Nota

Este repo é gerenciado pelo MineHost. Modificações manuais são possíveis no Codespace, mas podem ser sobrescritas ao recriar o servidor.
