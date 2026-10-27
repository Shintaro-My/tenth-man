# tenth-man

Um plugin de deliberação com vários modelos (uma skill e os agentes que ocupam os assentos do conselho) para [Codex](https://developers.openai.com/codex) e [Claude Code](https://claude.com/claude-code). Use-o para uma escolha de design, a revisão de uma política ou outra decisão que não deva depender da resposta de um único modelo. Agentes que usam modelos diferentes votam e um assento fixo sempre argumenta contra, então é menos provável que o conselho chegue a um acordo compartilhando o mesmo ponto cego.

[English](README.md) | [日本語](README.ja.md) | [简体中文](README.zh-CN.md) | [繁體中文](README.zh-TW.md) | [한국어](README.ko.md) | [Español](README.es.md) | [Português (Brasil)](README.pt-BR.md) | [Deutsch](README.de.md) | [Français](README.fr.md)

Os membros votantes avaliam cada proposta. Um **décimo homem** (tenth man) à parte deve argumentar contra cada proposta, independentemente da própria opinião, e dizer que evidência refutaria cada objeção. A sessão principal preside três rodadas, resolve as preocupações graves e redige uma ata que mostra o que foi acordado, por quê e o que continua em aberto.

A skill sempre apresenta primeiro o plano do conselho e espera a sua aprovação. Executar um conselho nunca aplica as conclusões ao seu projeto automaticamente.

## Como funciona

1. O presidente propõe os itens, os modelos, as regras, o custo e a pasta de saída.
2. Rodada 1: os membros votam; o décimo homem contesta cada item.
3. Rodada 2: os membros reconsideram as preocupações graves e respondem às objeções mais relevantes.
4. Rodada 3: os membros votam sobre a minuta da ata; o décimo homem confronta o resultado com as evidências.
5. O presidente escreve `.tenth-man/<date>-<slug>/minutes.md`.

Os votos são **a favor**, **a favor com condições**, **contra** e **preocupação grave**. O acordo exige a maioria dos membros votantes e nenhuma preocupação grave pendente. O décimo homem não tem voto.

## Instalação no Codex

### Como plugin

```sh
codex plugin marketplace add Shintaro-My/tenth-man
codex plugin add tenth-man@tenth-man
```

Para um checkout local, adicione o caminho dele:

```sh
codex plugin marketplace add /path/to/tenth-man
codex plugin add tenth-man@tenth-man
```

Invoque a skill do plugin digitando `$tenth-man:council` seguido da pergunta, ou escolha-a em `/skills`:

```text
$tenth-man:council Devemos mover o estado da sessão do Redis para cookies assinados?
```

### Como skill independente

Copie `skills/council` para um diretório de skills do Codex:

```sh
cp -r skills/council ~/.agents/skills/
```

Quando instalada dessa forma, invoque-a como `$council`.

Para um único repositório, copie-a para `<repo>/.agents/skills/`. Este repositório também contém em `.codex/agents/` perfis do Codex sem papel fixo para os assentos; o Codex detecta esses perfis automaticamente no checkout local. Para usar os perfis em todos os repositórios:

```sh
cp .codex/agents/seat-*.toml ~/.codex/agents/
```

A skill continua funcionando sem essa cópia: ela pode iniciar subagentes com as mesmas atribuições explícitas de modelo e manter as conversas dos subagentes entre as rodadas. O empacotamento como plugin não instala automaticamente os arquivos TOML de agentes personalizados.

## Instalação no Claude Code

### Como plugin

```text
/plugin marketplace add Shintaro-My/tenth-man
/plugin install tenth-man@tenth-man
```

Para testar um checkout local:

```sh
claude --plugin-dir /path/to/tenth-man
```

Invoque-a com:

```text
/tenth-man:council Devemos mover o estado da sessão do Redis para cookies assinados?
```

### Manualmente

```sh
cp -r skills/council ~/.claude/skills/
cp agents/seat-*.md ~/.claude/agents/
```

Na instalação manual, o comando é `/council`; os nomes dos agentes não têm o prefixo `tenth-man:`.

## Presets de modelos

O presidente propõe `light` para uma pergunta pequena e `standard` nos demais casos. Ao aprovar o plano, você pode escolher um preset ou alterar qualquer assento.

O décimo homem roda no mesmo modelo que o membro com maior capacidade de raciocínio. A classificação vem de resultados medidos, não de números de versão; as colunas “Décimo homem” abaixo refletem essa classificação em outubro de 2026. Quando a posição de um modelo novo é desconhecida, o presidente pergunta a você. Para usar outro modelo, diga isso ao dar a aprovação.

### Codex

| Preset | Membros votantes | Décimo homem | Execuções por rodada |
|---|---|---|---|
| `light` | GPT-6 Astra, GPT-6.1 Sol, GPT-6.1 Terra | GPT-6 Astra | 4 |
| `standard` | GPT-6 Astra, GPT-6.1 Sol, GPT-6.1 Terra, GPT-6 Sol | GPT-6 Astra | 5 |
| `no-astra` | GPT-6.1 Sol, GPT-6.1 Terra, GPT-6 Sol | GPT-6.1 Sol | 4 |

Esses padrões usam apenas modelos disponibilizados pelo host do Codex e pela conta atuais. O presidente informa os modelos indisponíveis e nunca substitui um deles sem avisar. Se o limite de concorrência do host for menor que o número de assentos do conselho, os agentes rodam em levas a partir do mesmo briefing, sem alterações; nenhuma contagem é feita até que todos terminem.

Os agentes personalizados do Codex correspondentes são `seat_astra`, `seat_sol`, `seat_terra`, `seat_previous_sol` e o opcional de menor custo `seat_luna`. As definições TOML deles fixam o modelo e o esforço de raciocínio e mantêm o trabalho do conselho em modo somente leitura.

### Claude Code

| Preset | Membros votantes | Décimo homem | Execuções por rodada |
|---|---|---|---|
| `light` | Fable, Opus, Sonnet | Fable | 4 |
| `standard` | Fable, Opus, Sonnet, Opus 5, Opus 4.8 | Fable | 6 |
| `no-fable` | Opus, Sonnet, Opus 5 | Opus | 4 |

Todos os presets mantêm um assento do Opus mesmo quando o Fable está mais bem classificado, porque o Opus às vezes encontra problemas que o Fable deixa passar.

O plugin do Claude inclui em `agents/` cinco agentes sem papel fixo para ocupar os assentos. Um briefing atribui a cada execução o papel de membro ou de décimo homem.

### Adicionar um modelo do Claude

Um modelo mais antigo do Claude pode ser selecionado pelo campo `model:` de uma definição de agente. Copie um dos arquivos de agente incluídos, altere `name`, `description` e `model` e salve-o em `~/.claude/agents/` para todos os projetos ou em `.claude/agents/` para um único projeto:

```markdown
---
name: seat-opus-4-7
description: A seat in a tenth-man council, on Opus 4.7.
model: claude-opus-4-7
effort: high
tools: Read, Grep, Glob, Bash, Write
---
```

Mantenha o mesmo conteúdo sem papel fixo dos arquivos de agente incluídos. Assim, o presidente pode atribuir `seat-opus-4-7` como membro ou como décimo homem.

### Usar seus próprios agentes do Claude

Qualquer definição de agente do Claude já existente pode ocupar um assento. O presidente começa o prompt dizendo que, nesta tarefa, o agente ocupará um assento no conselho em vez de fazer seu trabalho habitual, e então atribui o papel e o briefing comum. Isso é útil para agentes de revisão fixados em um modelo mais antigo.

### Quando um modelo do Claude fixado é descontinuado

Se um modelo fixado deixar de ser oferecido, o agente desse assento não consegue iniciar. O presidente o remove e avisa você; ele nunca o substitui por outro modelo sem avisar. Exclua ou atualize o arquivo desse agente para corresponder aos modelos disponíveis na sua conta.

## Custo e requisitos

Cada assento corresponde a uma execução completa de subagente por rodada. Em três rodadas, o preset `standard` do Codex usa 15 execuções e o preset `standard` do Claude Code usa 18.

- Codex: uma versão atual com subagentes habilitados e acesso aos modelos selecionados.
- Claude Code: as ferramentas Agent e SendMessage, notificações de conclusão de agentes em segundo plano e acesso aos modelos selecionados.

O chat do ChatGPT não é um destino compatível no momento, porque não consegue executar subagentes com um modelo diferente para cada assento.

## Por que “décimo homem”?

A skill se inspira na regra do “décimo homem”, uma norma israelense fictícia do filme *World War Z*, de 2013. Quando nove pessoas chegam à mesma conclusão a partir das mesmas informações, a décima pessoa presume que elas estão erradas e investiga o oposto. A regra é fictícia, mas existem práticas semelhantes:

- Depois que a inteligência militar israelense interpretou mal os sinais de alerta antes da Guerra do Yom Kippur, em 1973, ela criou uma função para contestar as próprias avaliações, comumente chamada de *ipcha mistabra*, que em aramaico significa “o oposto é provável”.
- O Talmude (Sanhedrin 17a) registra que, se todos os juízes votarem pela condenação em um caso de pena capital, o acusado é libertado; um resultado unânime sugere que ninguém argumentou adequadamente pela defesa.
- Nos processos de canonização católicos, o “advogado do diabo” argumentava contra o candidato e testava os pontos fracos dos relatos sobre sua vida e dos milagres atribuídos a ele. O cargo foi formalizado em 1587 e bastante reduzido em 1983.

## Licença

MIT. Veja [LICENSE](LICENSE).
