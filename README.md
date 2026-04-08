# sched

Gerenciador de agendamentos cron em **Português do Brasil**.

Esqueça sintaxe cron. Escreva o que você quer em português:

```
sched rodar toda segunda às 09:00 backup.sh
sched rodar fim do mês relatorio.sh
sched rodar a cada 2 horas verifica.sh
sched rodar no dia 15 às 08:30 pagamento.sh
sched rodar hoje às 18:00 backup.sh
sched rodar dia 20/04 às 14:00 script.sh
```

## Dependências

- `gawk`, `mawk` ≥ 1.3.4, ou qualquer awk com suporte a `strftime`/`systime`
- `crontab` (qualquer implementação POSIX)
- `column` (util-linux)

## Instalação

```bash
git clone https://github.com/coelhorb/sched
sudo install -m 755 sched/sched /usr/local/bin/sched
```

Ou sem privilégios de root:

```bash
mkdir -p ~/.local/bin
install -m 755 sched/sched ~/.local/bin/sched
# garanta que ~/.local/bin esteja no seu PATH
```

## Uso

```
sched rodar <frequência> [às HH:MM] <comando> [opções]
sched listar
sched descreva <id>
sched deletar [id] [-s | --sim]
sched ajuda
```

### Opções do `rodar`

| Opção | Descrição |
|---|---|
| `-s` / `--sim` | Agenda sem pedir confirmação |
| `--log <arquivo>` | Salva saída do comando em arquivo de log (caminho relativo → `/tmp/sched/`) |

### Horário

Após a frequência, você pode especificar um horário com `às HH:MM`:

```bash
sched rodar diariamente às 08:30 backup.sh
sched rodar a cada 10 dias às 14:00 reset-trial.sh
sched rodar toda segunda às 09:00 reuniao.sh
```

Quando o horário é omitido, o padrão é **12:00** (meio-dia).

> Frequências baseadas em minutos ou horas (`a cada N minutos`, `a cada N horas`, `por hora`) ignoram o horário especificado — o intervalo já define o tempo de execução.

## Frequências suportadas

| Sintaxe | Exemplo |
|---|---|
| `a cada N minutos` | `a cada 30 minutos` |
| `a cada N horas` | `a cada 6 horas` |
| `a cada N dias` | `a cada 3 dias` |
| `a cada N semanas` | `a cada 2 semanas` |
| `a cada N meses` | `a cada 3 meses` |
| `a cada hora` / `por hora` | — |
| `diariamente` / `todo dia` | — |
| `semanalmente` | — |
| `mensalmente` / `mensal` | — |
| `fim do mês` / `fim de mês` | último dia real do mês |
| `último dia do mês` | idem |
| `no dia N` / `dia N` | N entre 1 e 28, recorrente todo mês |
| `toda segunda` … `toda domingo` | com ou sem acento, com ou sem `-feira` |
| `a cada 2 segundas` … | a cada N semanas no dia especificado |
| `hoje` | execução única hoje |
| `amanhã` | execução única amanhã |
| `dia DD/MM` | execução única na data especificada |
| `dia DD/MM/AAAA` | execução única na data especificada |

### Fim de mês

`fim do mês` executa **sempre no último dia real** do mês (28 em fevereiro, 30 em abril, 31 em janeiro, etc.), não apenas no dia 28:

```bash
sched rodar fim do mês às 23:00 fatura.sh --sim
```

### Dia fixo do mês

Limitado ao dia 28 para funcionar em todos os meses:

```bash
sched rodar no dia 15 às 10:00 pagamento.sh
```

### Dias da semana

Aceita todas as variações:

```bash
sched rodar toda segunda às 09:00 reuniao.sh
sched rodar toda terça-feira deploy.sh
sched rodar a cada 2 sextas relatorio.sh
```

### Execução única

Para rodar um comando uma única vez, use `hoje`, `amanhã` ou uma data específica. O agendamento se remove automaticamente do crontab após a execução:

```bash
sched rodar hoje às 18:00 backup.sh
sched rodar amanhã às 09:30 relatorio.sh
sched rodar dia 20/04 às 14:00 script.sh
sched rodar dia 20/04/2026 às 14:00 script.sh
```

O horário é obrigatório para execuções únicas (não há padrão recorrente que defina quando rodar).

## Exemplos

```bash
# Backup diário às 3h com log
sched rodar diariamente às 03:00 backup.sh --log backup.log

# Verificação a cada 2 horas
sched rodar a cada 2 horas verifica.sh

# Reset de trial a cada 10 dias às 14h, sem confirmação
sched rodar a cada 10 dias às 14:00 reset-trial.sh --sim --log reset.log

# Relatório no último dia do mês
sched rodar fim do mês relatorio.sh --sim --log relatorio.log

# Reunião toda segunda às 9h
sched rodar toda segunda às 09:00 reuniao.sh

# Cobrar clientes quinzenalmente (padrão: meio-dia)
sched rodar a cada 2 semanas cobranca.sh

# Execução única hoje às 18h
sched rodar hoje às 18:00 backup.sh

# Execução única em data específica
sched rodar dia 20/04 às 14:00 script.sh

# Listar agendamentos
sched listar

# Ver detalhes de um agendamento
sched descreva 2

# Remover agendamento sem confirmação
sched deletar 3 --sim
```

### Saída do `sched listar`

```
Agendamentos:

ID  Frequência                      Próxima Execução  Comando       Log
#1  diariamente às 03:00            08/04 03:00       backup.sh     /tmp/sched/backup.log
#2  último dia do mês ao meio-dia   30/04 12:00       relatorio.sh  -
#3  toda segunda-feira às 09:00     13/04 09:00       reuniao.sh    -
#4  a cada 2 semanas ao meio-dia    21/04 12:00       cobranca.sh   -
#5  no dia 15 de cada mês às 10:00  15/04 10:00       pagamento.sh  -
#6  uma vez em 20/04 às 14:00       20/04 14:00       script.sh     -
```

## Como funciona

O `sched` armazena metadados como comentários diretamente no crontab do usuário:

```
# sched-id:1 | sched-log:/tmp/sched/backup.log | sched-cmd:backup.sh
0 3 * * * mkdir -p /tmp/sched && { echo "[...] backup.sh"; backup.sh; echo '---'; } >> /tmp/sched/backup.log 2>&1
```

Para execuções únicas, o comando no crontab inclui um wrapper de auto-remoção que apaga o próprio agendamento após a execução. O `listar` exibe apenas o comando original — o wrapper fica transparente.

Os IDs são renumerados automaticamente ao deletar entradas. Linhas do crontab que não foram criadas pelo `sched` são preservadas intactas.

## Licença

MIT
