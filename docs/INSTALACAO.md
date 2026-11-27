# Instalação do armavita-meta-ads-mcp

Este é um servidor MCP que liga um assistente de IA (Claude, Cursor, Codex e outros) às suas contas de anúncios da Meta: Facebook, Instagram e Threads. Ele usa a Marketing API v27 e traz 131 ferramentas para campanhas, conjuntos, anúncios, criativos, relatórios, públicos, formulários de lead, catálogos e CAPI.

É uma variante do [projeto original](https://github.com/EfrainTorres/armavita-meta-ads-mcp). O código desta versão está [neste fork](https://github.com/felipebarbosavasconcelos13-coder/armavita-meta-ads-mcp), na tag da Release que você baixou. A licença é AGPL-3.0-only (arquivo `LICENSE`).

## 1. Baixe o arquivo do seu sistema

Na página da Release, baixe:

| Sistema | Arquivo |
| --- | --- |
| Windows 10/11 (64 bits) | `armavita-meta-ads-mcp-<versão>-windows-x64.zip` |
| Linux x64 (Ubuntu 22.04+, Debian 12+) | `armavita-meta-ads-mcp-<versão>-linux-x64.tar.gz` |
| macOS com chip Apple (M1 ou mais novo) | `armavita-meta-ads-mcp-<versão>-macos-arm64.tar.gz` |

Para Mac com Intel ou outros sistemas, compile a partir do código. Isso exige Rust 1.98, um compilador C/C++ e CMake:

```sh
cargo install --locked --git https://github.com/felipebarbosavasconcelos13-coder/armavita-meta-ads-mcp --tag <versão>
```

**Confira o download (recomendado).** Compare o resultado com a linha correspondente do arquivo `SHA256SUMS` da Release:

- Windows (PowerShell): `Get-FileHash .\armavita-meta-ads-mcp-<versão>-windows-x64.zip`
- Linux: `sha256sum armavita-meta-ads-mcp-<versão>-linux-x64.tar.gz`
- macOS: `shasum -a 256 armavita-meta-ads-mcp-<versão>-macos-arm64.tar.gz`

## 2. Coloque o executável numa pasta fixa

**Windows:** extraia o `.zip` e mova o `armavita-meta-ads-mcp.exe` para, por exemplo, `C:\Users\<você>\tools\armavita-meta-ads-mcp\`.
- Se o Windows avisar que o app é de um editor desconhecido, clique em "Mais informações" → "Executar assim mesmo". O executável não tem assinatura digital paga.

**Linux / macOS:**

```sh
tar -xzf armavita-meta-ads-mcp-<versão>-<sistema>.tar.gz
mkdir -p ~/.local/bin
cp armavita-meta-ads-mcp-<versão>-<sistema>/armavita-meta-ads-mcp ~/.local/bin/
chmod +x ~/.local/bin/armavita-meta-ads-mcp
```

No macOS, libere o arquivo uma vez para o sistema aceitar rodá-lo:

```sh
xattr -d com.apple.quarantine ~/.local/bin/armavita-meta-ads-mcp
```

**Pasta de mídia (opcional):** para enviar imagens, vídeos e feeds locais, ou para exportar relatórios em CSV, crie uma pasta e use o caminho dela em `META_MEDIA_ROOT`. Exemplo no Windows: `C:\Users\<você>\tools\armavita-meta-ads-mcp\media`.

## 3. Gere o seu token da Meta

Cada pessoa usa **o próprio token**. Nunca use o de outra pessoa: o token dá acesso a todas as contas que ela gerencia.

O mais estável é o token de um **usuário do sistema**, que não expira:

1. Em [developers.facebook.com](https://developers.facebook.com/), tenha um app do tipo **Business** ligado ao seu Business Manager.
2. No [Business Manager](https://business.facebook.com/settings), vá em **Usuários → Usuários do sistema → Adicionar** e crie um usuário do tipo Funcionário.
3. Em **Atribuir ativos**, dê a ele acesso às contas de anúncios, Páginas, pixels/conjuntos de dados e catálogos que o assistente vai usar.
4. Clique em **Gerar novo token**, escolha o app, a validade **Nunca** e estes escopos:
   - `ads_management`, `ads_read`, `business_management`
   - `pages_show_list`, `pages_read_engagement`, `pages_manage_ads`
   - `leads_retrieval`, `catalog_management`
5. Copie o token e guarde em local seguro. Ele não aparece de novo.

## 4. Configure no seu app

Use `--transport=stdio` e passe o token pela variável `META_ACCESS_TOKEN`. Nunca escreva o token numa conversa com a IA.

**Claude Code** (terminal):

```sh
claude mcp add-json -s user armavita-meta-ads-mcp '{"command":"C:/Users/<você>/tools/armavita-meta-ads-mcp/armavita-meta-ads-mcp.exe","args":["--transport=stdio"],"env":{"META_ACCESS_TOKEN":"<seu-token>","META_MEDIA_ROOT":"C:/Users/<você>/tools/armavita-meta-ads-mcp/media"}}'
```

**Claude Desktop:** edite o arquivo `claude_desktop_config.json` (menu Configurações → Desenvolvedor → Editar configuração) e acrescente dentro de `"mcpServers"`:

```json
"armavita-meta-ads-mcp": {
  "command": "C:/Users/<você>/tools/armavita-meta-ads-mcp/armavita-meta-ads-mcp.exe",
  "args": ["--transport=stdio"],
  "env": {
    "META_ACCESS_TOKEN": "<seu-token>",
    "META_MEDIA_ROOT": "C:/Users/<você>/tools/armavita-meta-ads-mcp/media"
  }
}
```

No Linux e no macOS, use o caminho `~/.local/bin/armavita-meta-ads-mcp`, escrito por extenso, por exemplo `/home/<você>/.local/bin/armavita-meta-ads-mcp`.

Outros apps (Cursor, Codex, VS Code etc.) usam os mesmos três itens: o executável, `--transport=stdio` e as variáveis de ambiente.

**Opções extras (variáveis de ambiente):**

| Variável | Para que serve |
| --- | --- |
| `META_MEDIA_ROOT` | Pasta única de onde o servidor lê mídias e onde grava relatórios CSV. |
| `META_ADS_TOOLSETS` | Carrega só alguns grupos de ferramentas, por exemplo `insights,audiences`. O grupo `core` está sempre ligado. Sem a variável, todas as ferramentas ficam ativas. |
| `META_APPSECRET_PROOF=1` + `META_APP_SECRET` | Assina cada chamada com a chave secreta do app (mais segurança). A chave precisa ser do mesmo app que gerou o token. |

Reinicie o app depois de salvar.

## 5. Teste

Peça ao assistente:

1. "Rode `inspect_access_token`". Deve mostrar o token válido, o tipo (`SYSTEM_USER`), se expira e os escopos que faltam.
2. "Liste minhas contas de anúncios" (`list_ad_accounts`).

Se aparecer `AUTH_REQUIRED` ou `AUTH_EXPIRED`, o token está ausente, foi digitado errado ou foi revogado.

## Segurança no dia a dia

- **Mudanças:** as que valem de verdade exigem confirmação explícita. Os planos de mudança só são aplicados com `APPLY_LIVE_META_ADS_CHANGES`, e exclusões com `CONFIRM_META_ADS_REMOVALS`. Revise o que a IA propõe antes de aprovar.
- **Pausados:** campanhas, conjuntos e anúncios novos são criados pausados.
- **IA generativa:** criativos com recursos de IA generativa precisam ter o preview revisado antes de o anúncio ir ao ar.

## Atualizar

Baixe a Release nova, substitua o executável (com o app fechado) e reinicie o app.
