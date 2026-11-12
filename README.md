# whatsapp-neway-automacoes

Nó para o Node-RED que recebe e envia mensagens de WhatsApp pelo caminho do WhatsApp Web: o aparelho é vinculado por **QR code** no próprio editor, sem a API oficial e sem servidor externo.

Neway Automações.

Aviso: é um cliente não oficial e viola os termos de uso do WhatsApp. O número pode ser suspenso. Use um número secundário, em uso pessoal e reativo. Os limites embutidos reduzem o risco, não o eliminam.

Requisitos: Node.js 20 ou mais novo e internet na instalação (o pacote baixa a biblioteca Baileys).

## O que vem no pacote

Na paleta, no grupo **whatsapp neway automacoes**, aparece um nó:

- **WhatsApp**: recebe e envia mensagens de texto.

A conta WhatsApp (login e conexão) é um nó de configuração: é criada de dentro do nó **WhatsApp** (campo **Conta**, botão do lápis) e fica listada em **Configuration nodes**.

## Instalar

1. Baixe o arquivo `.tgz` deste repositório.
2. No Node-RED, abra o menu (três barras), **Gerenciar paleta**, aba **Instalar**, e envie o arquivo `.tgz`.
3. Confirme. Na paleta aparece o grupo **whatsapp neway automacoes**. Se não aparecer, reinicie o Node-RED.

## Conta (feita uma vez)

1. Arraste um nó **WhatsApp** para o fluxo, dê duplo clique e crie uma conta (lápis).
2. Faça o **Deploy**.
3. Abra a conta de novo e clique em **Conectar / Gerar QR**.
4. No celular: WhatsApp, **Aparelhos conectados**, **Conectar um aparelho**, e escaneie o QR.

A sessão fica salva na pasta de dados do Node-RED e sobrevive a reinícios. O botão **Desvincular** encerra o vínculo.

## Usar o nó WhatsApp

| Campo | O que faz |
|---|---|
| Nome | só um rótulo para você reconhecer o nó no fluxo (opcional) |
| Conta | a conta WhatsApp vinculada acima |
| Tipo | define o papel do nó: **Recebimento** (só saída) ou **Envio** (só entrada) |
| Saídas por número ou grupo | só no Recebimento. Lista de números (com código do país, só dígitos, ex.: `5511999998888`) ou ids de grupo (terminam em `@g.us`). Cada item cria uma saída extra no nó, que recebe também as mensagens dele; a saída 1 continua recebendo tudo |
| Número | só no Envio. Destino fixo do envio: um número (só dígitos, com código do país) ou o id de um grupo (termina em `@g.us`). **Vazio** = o destino vem no `msg.payload` |

Na conta (nó de configuração):

| Campo | O que faz |
|---|---|
| Nome | rótulo da conta |
| Conectar / Gerar QR | mostra o QR para vincular o celular |
| Desvincular | remove o aparelho do WhatsApp e apaga a sessão salva |

### Tipo: Recebimento (só saída)

Emite uma mensagem para cada mensagem recebida (conversas individuais e grupos):

```
msg.payload = { numero, mensagem, nome, grupo, tipo }
```

| Campo | Conteúdo |
|---|---|
| `numero` | número de quem escreveu, só dígitos, com código do país |
| `mensagem` | o texto (vazio se for mídia sem legenda) |
| `nome` | nome que a pessoa usa no WhatsApp |
| `grupo` | o id do grupo (termina em `@g.us`) se veio de um grupo; senão `null` |
| `tipo` | `texto`, `imagem`, `video`, `audio`, `documento`, `figurinha`, `localizacao` ou `contato` |

Imagens, vídeos, áudios e outros arquivos chegam só como aviso (com a legenda, se houver): o arquivo em si não é baixado. Status e listas de transmissão são ignorados. Mensagens que você mesmo envia em outras conversas pelo celular não aparecem; só a conversa com o seu próprio número é entregue.

A lista **Saídas por número** cria saídas extras: a saída 1 recebe tudo; cada número (ou id de grupo, terminado em `@g.us`) adicionado cria uma saída que recebe também as mensagens dele. O que você escreve na conversa com o seu próprio número também é entregue.

### Tipo: Envio (só entrada, sem saída)

Com o campo **Número** vazio, o destino vem no `msg.payload`, que aceita número ou id de grupo:

```
msg.payload = { numero: "5511999999999", mensagem: "Olá" }
msg.payload = { numero: "<id do grupo>@g.us", mensagem: "Olá" }
msg.payload = { grupo: "<id do grupo>@g.us", mensagem: "Olá" }
```

Com o campo **Número** preenchido (número ou id de grupo), o destino é fixo e `msg.payload` pode ser só o texto, ou `{ mensagem }`:

```
msg.payload = "Olá"
```

Só texto. Ligar a saída do Recebimento na entrada do Envio responde a quem escreveu (ou ao grupo). Mensagens recusadas (sem contato recente, texto repetido, fila cheia) aparecem como aviso no log e na etiqueta do nó.

## Grupos

O envio para grupo segue a mesma proteção das pessoas: o grupo só é liberado depois que **outra pessoa escreve nele** enquanto o nó está online, e fica liberado por 30 dias. O que você mesmo escreve no grupo pelo celular não conta, e o histórico anterior à conexão também não. Um grupo recém-criado, em que ninguém escreveu com o nó online, é recusado com a mensagem de que não escreveu nos últimos 30 dias.

Para descobrir o id de um grupo: ligue um nó **WhatsApp** em Recebimento a um nó debug, peça a alguém para escrever no grupo e leia `msg.payload.grupo` (termina em `@g.us`). Use esse id no campo **Número** do Envio ou no `msg.payload`.

## Regras fixas de envio

Não têm campo de configuração, para proteger o número:

- intervalo aleatório de 5 a 15 s entre mensagens, com pausa longa ocasional (30 a 90 s);
- no máximo 4 por minuto, 60 por hora e 200 por dia; o excedente espera numa fila de 50;
- só envia para quem (pessoa ou grupo) escreveu nos últimos 30 dias, exceto o seu próprio número;
- marca a conversa como lida e mostra "digitando" proporcional ao texto;
- o mesmo texto não vai para mais de 2 números em 10 minutos;
- texto de até 4000 caracteres.

## Reconexão

Queda comum: tenta de novo com espera crescente (5 s até 5 min). Se o vínculo for removido pelo celular, a sessão é apagada e um novo QR é pedido. Se o mesmo número entrar em outro lugar, o nó não reconecta sozinho (uma sessão por número).

## Problemas comuns

| Sintoma | O que fazer |
|---|---|
| QR não aparece | fazer o Deploy antes e clicar em **Conectar / Gerar QR** |
| Mensagem não sai | conferir se o contato escreveu nos últimos 30 dias e se há fila |
| Grupo recusado ("não escreveu para você nos últimos 30 dias") | alguém precisa escrever no grupo com o nó online; confirmar que o id termina em `@g.us` e foi colocado no campo **Número** ou em `msg.payload` |
| Editor mostra o nó como era antes da atualização | reiniciar o Node-RED |

## Atualizar

Baixe o `.tgz` atual e instale do mesmo jeito (Gerenciar paleta, Instalar, enviar o arquivo). A sessão do WhatsApp é mantida.

## Desinstalar

1. Apague **todos os nós WhatsApp** dos fluxos.
2. Apague a **conta WhatsApp**: menu, **Configuration nodes**, escolha a conta e exclua (se quiser, clique em **Desvincular** antes).
3. Faça o **Deploy**.
4. Menu, **Gerenciar paleta**, aba **Nós**, ache `whatsapp-neway-automacoes` e clique em **Remover**.

## Segurança

A sessão (chaves do WhatsApp) fica em `whatsapp_<id>/` na pasta de dados do Node-RED. Quem tem acesso a essa pasta ou ao editor consegue usar o seu WhatsApp. Proteja o administrador do Node-RED com senha se ele estiver exposto na rede.
