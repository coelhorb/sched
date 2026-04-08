# sched

Gerenciador de agendamentos cron em **Português do Brasil**.

Esqueça sintaxe cron. Escreva o que você quer em português:

```
sched rodar toda segunda backup.sh
sched rodar fim do mês relatorio.sh
sched rodar a cada 2 horas verifica.sh
sched rodar no dia 15 pagamento.sh
```

## Dependências

- `gawk` ≥ 4.0
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
sched rodar <frequência> <comando> [opções]
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
| `no dia N` / `dia N` | N entre 1 e 28 |
| `toda segunda` … `toda domingo` | com ou sem acento, com ou sem `-feira` |
| `a cada 2 segundas` … | a cada N semanas no dia especificado |

### Fim de mês

`fim do mês` executa **sempre no último dia real** do mês (28 em fevereiro, 30 em abril, 31 em janeiro, etc.), não apenas no dia 28:

```bash
sched rodar fim do mês fatura.sh --sim
```

### Dia fixo do mês

Limitado ao dia 28 para funcionar em todos os meses:

```bash
sched rodar no dia 15 pagamento.sh
```

### Dias da semana

Aceita todas as variações:

```bash
sched rodar toda segunda reuniao.sh
sched rodar toda terça-feira deploy.sh
sched rodar a cada 2 sextas relatorio.sh
```

## Exemplos

```bash
# Backup diário com log
sched rodar diariamente backup.sh --log backup.log

# Verificação a cada 2 horas
sched rodar a cada 2 horas verifica.sh

# Relatório no último dia do mês
sched rodar fim do mês relatorio.sh --sim --log relatorio.log

# Reunião toda segunda de manhã (edite o script para o horário)
sched rodar toda segunda reuniao.sh

# Cobrar clientes quinzenalmente
sched rodar a cada 2 semanas cobranca.sh

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

ID  Frequência             Próxima Execução  Comando       Log
#1  diariamente            08/04 00:00       backup.sh     /tmp/sched/backup.log
#2  última dia do mês      30/04 00:00       relatorio.sh  -
#3  toda segunda-feira     13/04 00:00       reuniao.sh    -
#4  a cada 2 sextas        11/04 00:00       cobranca.sh   -
#5  no dia 15 de cada mês  15/04 00:00       pagamento.sh  -
```

## Como funciona

O `sched` armazena metadados como comentários diretamente no crontab do usuário:

```
# sched-id:1 | sched-log:/tmp/sched/backup.log | sched-cmd:backup.sh
0 0 * * * mkdir -p /tmp/sched && { echo "[...] backup.sh"; backup.sh; echo '---'; } >> /tmp/sched/backup.log 2>&1
```

Os IDs são renumerados automaticamente ao deletar entradas. Linhas do crontab que não foram criadas pelo `sched` são preservadas intactas.

## Licença

MIT
