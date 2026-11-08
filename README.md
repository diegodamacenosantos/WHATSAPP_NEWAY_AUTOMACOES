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
| Número | só no Envio. Destino fixo do envio. **Vazio** = o destino vem no `msg.payload` |

Na conta (nó de configuração):

| Campo | O que faz |
|---|---|
| Nome | rótulo da conta |
| Conectar / Gerar QR | mostra o QR para vincular o celular |
| Desvincular | remove o aparelho do WhatsApp e apaga a sessão salva |

### Tipo: Recebimento (só saída)

Emite uma mensagem para cada texto recebido:

```
msg.payload = { numero, mensagem, nome, grupo, tipo }
```

A lista **Saídas por número** cria saídas extras: a saída 1 recebe tudo; cada número (ou id de grupo, terminado em `@g.us`) adicionado cria uma saída que recebe também as mensagens dele. O que você escreve na conversa com o seu próprio número também é entregue.

### Tipo: Envio (só entrada, sem saída)

```
msg.payload = { numero: "5511999999999", mensagem: "Olá" }
msg.payload = { grupo: "<id do grupo>@g.us", mensagem: "Olá" }
```

Só texto. Com o campo **Número fixo** preenchido, `msg.payload` pode ser só o texto, ou `{ mensagem }`.

## Regras fixas de envio

Não têm campo de configuração, para proteger o número:

- intervalo aleatório de 5 a 15 s entre mensagens, com pausa longa ocasional (30 a 90 s);
- no máximo 4 por minuto, 60 por hora e 200 por dia; o excedente espera numa fila de 50;
- só envia para quem escreveu nos últimos 30 dias (exceto o seu próprio número);
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
