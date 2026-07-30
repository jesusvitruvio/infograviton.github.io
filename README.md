# QGI — Quantum Gravitation and Information

Site do grupo de pesquisa em gravitação e informação quântica da Universidade
Federal do ABC. Em inglês, com matemática em LaTeX, tema claro e escuro, e o
arquivo completo do journal club **InfoGraviton**.

Este documento é longo de propósito. Ele supõe que você programa, mas **não**
supõe que você conheça Jekyll, Liquid, YAML ou SCSS — cada um é apresentado
quando aparece pela primeira vez. Se você só quer publicar uma mudança rápida,
leia a seção [Tarefas do dia a dia](#tarefas-do-dia-a-dia) e ignore o resto.

---

## Sumário

- [O modelo mental: gerador de site estático](#o-modelo-mental-gerador-de-site-estático)
- [As tecnologias, e o papel de cada uma](#as-tecnologias-e-o-papel-de-cada-uma)
- [Rodando localmente](#rodando-localmente)
- [Mapa do repositório](#mapa-do-repositório)
- [Os quatro conceitos do Jekyll que este site usa](#os-quatro-conceitos-do-jekyll-que-este-site-usa)
- [Tarefas do dia a dia](#tarefas-do-dia-a-dia)
- [Aparência: cores, fontes e tipografia](#aparência-cores-fontes-e-tipografia)
- [Publicação](#publicação)
- [Armadilhas conhecidas](#armadilhas-conhecidas)
- [Glossário](#glossário)

---

## O modelo mental: gerador de site estático

A ideia central, e a única que realmente importa entender:

> **Você não edita o site. Você edita os arquivos-fonte, e um programa gera o
> site a partir deles.**

Esse programa é o **Jekyll**. Ele lê o conteúdo do repositório — textos em
Markdown, listas em YAML, moldes em HTML — combina tudo e escreve uma pasta de
HTML puro em `_site/`. É essa pasta que vai para o ar.

```
arquivos-fonte  ──► Jekyll (build) ──►  _site/  ──►  navegador
   (você edita)                       (gerado, nunca editado à mão)
```

Três consequências práticas disso:

**Não existe banco de dados nem servidor de aplicação.** O site publicado é um
conjunto de arquivos `.html` estáticos. Nada é calculado quando o visitante
acessa a página; tudo foi decidido no momento do build. Isso torna o site
rápido, barato (hospedagem gratuita no GitHub Pages) e praticamente imune a
invasão.

**A pasta `_site/` é descartável.** Ela é regenerada a cada build e está no
`.gitignore`. Se você editar algo lá, perde na próxima compilação. Todo
underscore no início de um nome (`_site`, `_data`, `_layouts`) sinaliza "coisa
do Jekyll", não uma página do site.

**Repetição é resolvida no build, não em tempo de execução.** A lista de
membros existe uma única vez, em `_data/members.yml`; o Jekyll a percorre e
escreve o HTML de cada cartão. Você edita um lugar, a página inteira muda.

---

## As tecnologias, e o papel de cada uma

Cinco linguagens, cada uma com um trabalho bem delimitado. Nenhuma delas é
difícil isoladamente — a confusão costuma vir de não saber qual é qual.

### YAML — dados e configuração

Formato de dados baseado em indentação, como um JSON sem chaves nem vírgulas.
Guarda listas e pares chave-valor. É o que descreve **quem** são os membros,
**quais** são as linhas de pesquisa e **como** o site está configurado.

```yaml
- slug: alves            # um item de lista começa com "- "
  name: Níckolas de Aguiar Alves
  roles:                 # indentação define o aninhamento
    - kind: phd
      start: '2023-09'
```

Regra de ouro: **indentação é sintaxe**. Dois espaços a mais ou a menos mudam o
significado, e nunca use tabulação. Este é o formato onde erros são mais fáceis
de cometer e mais silenciosos — veja
[Armadilhas conhecidas](#armadilhas-conhecidas).

### Markdown — texto com formatação leve

O `**negrito**` e `# título` que você já conhece do GitHub. Aqui, é como os
resumos de seminário são escritos. O Jekyll converte para HTML no build.
Fórmulas em LaTeX passam intactas: `$E = mc^2$` inline, `$$...$$` em bloco.

### HTML — a estrutura das páginas

O esqueleto: parágrafos, títulos, listas, links. Se você já leu HTML alguma
vez, o que está nas páginas deste site é HTML comum, sem nada exótico.

### Liquid — a lógica dos moldes

Esta provavelmente é a única novidade real. Liquid é uma linguagem de
*templates*: HTML com marcações que o Jekyll executa no build. Tem duas
sintaxes, e distinguir as duas resolve 90% da leitura:

| Sintaxe | O que faz | Exemplo |
|---|---|---|
| `{{ ... }}` | **imprime** um valor | `{{ page.title }}` |
| `{% ... %}` | **executa** lógica (não imprime nada) | `{% for m in members %}` |

Os comandos que aparecem neste repositório são poucos:

```liquid
{% assign t = site.data.strings %}          atribui uma variável
{% for m in site.data.members %} ... {% endfor %}   itera sobre uma lista
{% if page.hero %} ... {% endif %}           condicional
{% include member_card.html member=m %}      insere um trecho reutilizável
{{ paper.date | date: "%Y" }}                 aplica um filtro (o "|")
```

Duas particularidades que economizam tempo depois:

O hífen em `{%-` e `-%}` remove o espaço em branco em volta da tag. É puramente
cosmético, para o HTML gerado não ficar cheio de linhas vazias.

Filtros (`|`) **não funcionam dentro de uma condição**. Isto é inválido em
Liquid e falha silenciosamente:

```liquid
{% if src | slice: 0, 7 == "http://" %}      ✗ não faça isso
```

O correto é calcular antes e comparar depois:

```liquid
{% assign p7 = src | slice: 0, 7 %}
{% if p7 == "http://" %}                      ✓
```

### SCSS — o visual

SCSS é CSS com alguns recursos extras (aninhamento, `@import`). O Jekyll o
compila para CSS no build. Neste site as cores **não** usam variáveis do SCSS;
usam *custom properties* do CSS (`--bg`, `--text`, `--accent`), que continuam
existindo no navegador em tempo de execução. É isso que permite o alternador de
tema claro/escuro trocar a paleta inteira mudando um atributo no `<html>`, sem
recompilar nada.

---

## Rodando localmente

Jekyll é escrito em **Ruby**, então é de Ruby que você precisa — não de Node.

### Instalação

**Windows:** instale o
[RubyInstaller **with DevKit**](https://rubyinstaller.org/downloads/). A versão
sem DevKit não compila as extensões nativas de algumas gems e falha no meio da
instalação.

**macOS:** `brew install ruby` (o Ruby que vem com o sistema é antigo e
protegido).

**Linux:** `sudo apt install ruby-full build-essential zlib1g-dev` ou o
equivalente da sua distribuição.

### Primeira execução

```bash
bundle install        # instala as gems listadas no Gemfile
bundle exec jekyll serve
```

Abra <http://localhost:4000>. `bundle exec` garante que o Jekyll executado é o
da versão travada no `Gemfile.lock`, e não outro que você tenha instalado
globalmente.

O servidor observa os arquivos e reconstrói ao salvar — **exceto o
`_config.yml`**, que só é lido na inicialização. Se mexeu nele, derrube com
`Ctrl+C` e suba de novo.

### Comandos úteis

```bash
bundle exec jekyll serve --livereload   # recarrega o navegador sozinho
bundle exec jekyll serve --drafts       # inclui rascunhos
bundle exec jekyll build                # só gera _site/, sem servidor
bundle exec jekyll build --verbose      # útil quando algo não aparece
```

Se uma página sumiu ou saiu em branco, o motivo quase sempre é **front matter
inválido** — veja [Armadilhas conhecidas](#armadilhas-conhecidas).

---

## Mapa do repositório

```
├── _config.yml              configuração global (lido só no início do build)
├── Gemfile / Gemfile.lock   dependências Ruby
│
├── _data/                   ← DADOS: edite aqui, não no HTML
│   ├── members.yml            todas as pessoas, atuais e egressas
│   ├── research.yml           as 5 linhas de pesquisa
│   └── strings.yml            rótulos de interface, rotas e títulos
│
├── _layouts/                ← MOLDES de página inteira
│   ├── default.html           cabeçalho, navegação, rodapé, scripts
│   └── seminar.html           página individual de seminário
│
├── _includes/               ← TRECHOS reutilizáveis (parciais)
│   ├── member_card.html       um cartão de pessoa
│   ├── member_status.html     deduz situação a partir das datas
│   ├── alumnus_row.html       linha compacta de egresso
│   ├── publication_list.html  publicações agrupadas por ano
│   ├── teaching_list.html     disciplinas ou notas de aula (course/notes)
│   ├── include_seminar.html   um item na lista de seminários
│   ├── bibitem.html           uma referência bibliográfica formatada
│   ├── source.html            formatação por tipo (artigo, livro, tese...)
│   ├── authorlist.html        "A, B, and C"
│   ├── authorname.html        nome com link; resolve `slug` em members.yml
│   ├── journaldata.html       revista, volume, número, páginas
│   ├── eprintdata.html        arXiv, INSPIRE, estado da publicação
│   ├── pubstate.html          "In preparation", "Forthcoming" etc.
│   ├── hyperlink.html         resolve DOI / handle / URL
│   └── typeset_date_lang.html data do seminário, com hora se for futuro
│
├── _sass/                   ← ESTILOS (compilados para CSS)
│   ├── base/_variables.scss   cores, fonte, medidas, recuo de parágrafo
│   ├── base/_fonts.scss       @font-face da Montserrat (legado, não usada)
│   ├── base/_typography.scss  corpo do texto, títulos, justificação
│   ├── base/_media.scss       imagens, tabelas, equações
│   ├── layout/_navigation.scss  cabeçalho, menu e alternador de tema
│   └── layout/_components.scss  apresentação, membros, publicações, ensino
│
├── _seminars/               ← COLEÇÃO: um arquivo por seminário (tem página)
├── _bibliography/           ← COLEÇÃO: referências (sem página própria)
├── _suggestions/            ← COLEÇÃO: leituras sugeridas (sem página própria)
├── _teaching/               ← COLEÇÃO: disciplinas e notas de aula (idem)
│
├── assets/
│   ├── css/main.scss          ponto de entrada dos estilos
│   ├── fonts/                 .otf da Montserrat (legado)
│   └── img/
│       ├── banner-home.jpg    foto ao lado da apresentação na home
│       └── members/           fotos, nomeadas pelo slug do membro
│
├── index.html               home
├── members/                 Members
├── research/                Research ▾  → Branches
│   └── publications/          ↳ Publications (artigos e teses)
├── teaching/                Teaching
├── upcoming/                InfoGraviton ▾ → Upcoming
├── past/                      ↳ Past
├── suggestions/               ↳ Suggestions
├── join/                    How to Join
│
├── .github/workflows/pages.yml   build e publicação
├── .vscode/settings.json    desliga o formatador que corrompe front matter
└── _site/                   GERADO — não edite, não commite
```

Uma pasta com `index.html` dentro vira uma URL limpa: `members/index.html` é
servido em `/members/`.

O menu tem seis itens no topo: Home, Members, **Research▾**, Teaching,
**InfoGraviton▾** e How to Join. Os dois com seta são submenus — o primeiro
reúne as linhas de pesquisa e a produção escrita do grupo (artigos e teses,
juntos em Publicações); o segundo, a agenda e o arquivo do journal club. O rótulo do submenu não é uma página, apenas
abre a lista. A exceção é *Branches*, que corresponde a `/research/`, então esse
endereço existe e funciona.

---

## Os quatro conceitos do Jekyll que este site usa

### 1. Front matter

Todo arquivo que o Jekyll deve processar começa com um bloco YAML entre duas
linhas de `---`. Esse bloco define **metadados** da página, acessíveis no molde
como `page.<chave>`.

```html
---
layout: default          ← qual molde envolve esta página
title: Members           ← vira o <h1> e o <title>
description: ...         ← meta description, usada por buscadores
---
<p>O conteúdo da página começa aqui.</p>
```

O front matter pode estar **vazio** — `---` seguido de `---` — e isso não é um
erro: é como se sinaliza "processe este arquivo" para algo que não precisa de
metadados. É exatamente o caso de `assets/css/main.scss`.

### 2. Layouts (moldes)

Um layout é a moldura. `_layouts/default.html` contém tudo que se repete em
toda página — `<head>`, cabeçalho, navegação, rodapé, scripts — e um
`{{ content }}` no meio, onde o conteúdo da página é injetado.

`_layouts/seminar.html` é aninhado: ele declara `layout: default` no próprio
front matter, então acrescenta a estrutura de um seminário (título, data,
palestrante, referências) e o resultado ainda passa pelo molde geral.

### 3. Includes (parciais)

Trechos reutilizáveis em `_includes/`, inseridos com `{% include %}` e capazes
de receber parâmetros:

```liquid
{% include member_card.html member=m lang='pt' %}
```

Dentro do include, os parâmetros chegam com o prefixo `include.`:
`include.member`, `include.lang`. Fora disso, um include vê as mesmas variáveis
globais que a página (`site`, `page`).

É aqui que vive a máquina de citações herdada do repositório original:
`bibitem.html` recebe uma entrada bibliográfica e delega a `source.html`, que
decide o formato conforme o `type` (artigo, livro, tese, capítulo). Adicionar um
novo tipo de referência é acrescentar um `when` naquele `case`.

### 4. Coleções e arquivos de dados

Duas formas de guardar conjuntos de itens, com uma diferença decisiva:

**Coleções** (`_seminars/`, `_bibliography/`, `_suggestions/`) — um **arquivo
por item**, com front matter e, opcionalmente, corpo em Markdown. Declaradas no
`_config.yml`:

```yaml
collections:
  seminars:
    output: true      # cada item ganha uma página própria
  bibliography:
    output: false     # os dados existem, mas não geram páginas
```

Em Liquid, acessa-se com `site.seminars`, `site.bibliography`. Use coleções
quando cada item for volumoso ou precisar de página própria.

**Arquivos de dados** (`_data/*.yml`) — uma **lista dentro de um só arquivo**,
sem páginas. Acessa-se com `site.data.members`, `site.data.strings`. Use quando os
itens forem curtos e você quiser editar todos de uma vez.

Neste site: seminários são coleção (têm página, resumo longo); membros são
dados (oito entradas curtas, mais prático num arquivo).

---

## Tarefas do dia a dia

### Publicar qualquer mudança

```bash
git add .
git commit -m "descrição do que mudou"
git push
```

O GitHub reconstrói e publica em um ou dois minutos. Não existe passo de deploy
manual.

### Adicionar um seminário

Crie `_seminars/AAAA-MM-DD.md` (o nome do arquivo define a URL):

```yaml
---
title: Título da palestra
date: 2026-08-13T13:30-03:00      # com fuso; -03:00 é São Paulo
speaker:
    - name: Nome do Palestrante
      orcid: 0000-0000-0000-0000  # opcional: vira link no nome
language: English
bibliography:
    - danielson2022BlackHolesDecoherenceQuantumSuperpositions
---
Resumo em Markdown. Matemática com $...$ e $$...$$.
```

Cada chave em `bibliography` é o **nome do arquivo** correspondente em
`_bibliography/`, sem o `.md`. Sem correspondência, a página exibe
"Missing entry" em vez de falhar o build — erro visível, e de propósito.

O seminário aparece automaticamente em "Próximos" ou "Anteriores" conforme a
data comparada ao momento do build. Como `future: true` está no `_config.yml`,
datas futuras são publicadas normalmente.

### Adicionar uma referência bibliográfica

Crie `_bibliography/chave.md` contendo **apenas** front matter:

```yaml
---
type: article
author:
    - name: D. L. Danielson
    - name: R. M. Wald
title: Black holes decohere quantum superpositions
journal: International Journal of Modern Physics D
volume: '31'
number: '14'
pages: '2241003'
date: 2022-10-01
doi: 10.1142/S0218271822410036
arxiv: '2205.06279'
arxivclass: quant-ph
---
```

Convenção de nome: `primeiroautorANOPalavrasDoTitulo`. Números vão entre
aspas para o YAML não os converter (um volume `08` viraria `8`).

Autores aceitam `orcid`, `lattes`, `inspire` ou `website`, e o nome vira link
automaticamente. `type` aceita `article`, `book`, `thesis`, `incollection`,
`unpublished` e outros — a lista completa está no `case` de
`_includes/source.html`.

### Adicionar uma publicação do grupo

Publicações são as entradas de `_bibliography/` marcadas com **`group: true`**:

```yaml
---
group: true
type: article
author:
    - name: Níckolas de Aguiar Alves
    - name: André G. S. Landulfo
...
```

Sem essa linha a referência continua citável nos seminários, mas não aparece em
Publicações. O agrupamento é por ano, decrescente; entradas com
`pubstate: inpreparation` vão para uma seção própria ao final.

> **Detalhe que já causou bug:** não filtre publicações por "data ausente". O
> Jekyll atribui `site.time` a documentos de coleção sem data, então o teste
> nunca dá verdadeiro. O critério confiável é o `pubstate`.

### O seletor de tipo em Publicações

No topo da página há filtros — *All*, *Articles*, *Theses*, *Dissertations*,
*Monographs* — que escondem e mostram itens sem recarregar a página. O tipo de
cada item sai de `thesis_level` quando existe e de `type` caso contrário, e vira
o atributo `data-kind` do `<li>`.

Três detalhes do funcionamento:

Filtros sem nenhum item **não são renderizados**. Como ainda não há monografias,
o botão *Monographs* não aparece — em vez de existir e não fazer nada. Cada
botão mostra a contagem ao lado do rótulo, e um ano que fica sem itens visíveis
esconde o próprio cabeçalho, para não deixar título órfão.

O filtro fica no endereço, como `/research/publications/#phd`, então dá para
mandar a alguém um link já filtrado. A página lê o hash ao carregar.

O seletor **só aparece se houver JavaScript**: ele nasce com o atributo
`hidden`, removido pelo script. Sem JS, o visitante vê a lista completa, que
continua perfeitamente legível — nenhum conteúdo depende do script para existir.

Para acrescentar um tipo, ponha o botão em `_includes/publication_list.html` com
`data-filter` igual ao valor que aparecerá em `data-kind`, e o rótulo em
`_data/strings.yml` sob `ui:`.

### Membros: um nome nunca é apagado

Este é o ponto mais importante do `_data/members.yml`, e vale entender antes de
editar qualquer coisa.

Em vez de gravar um estado — *"fulano é doutorando"* —, cada pessoa carrega uma
**lista de passagens datadas** pelo grupo. O site deduz o resto: quem está aqui
hoje, há quanto tempo, quando defendeu, e em que ordem as coisas aconteceram.

```yaml
- slug: alves
  name: Níckolas de Aguiar Alves
  lattes: '4933609438452718'
  orcid: 0000-0002-0309-735X
  website: https://aguiaralves.com
  roles:
    - kind: msc
      start: '2021-02'
      end: '2023-04'
      funding: FAPESP
    - kind: phd            # sem `end`: vínculo em curso
      start: '2023-09'
      funding: CAPES
  blurb: Uma ou duas frases sobre a pesquisa.
```

**A ausência de `end` é o que define quem está no grupo.** Quando alguém
defende, você acrescenta a data de término — e a pessoa migra sozinha da seção
*Current members* para *Alumni*, sem que ninguém precise apagar nada nem mover
blocos de arquivo. O registro histórico fica intacto, e quem fez mestrado e
doutorado aqui aparece com as duas passagens.

Consequências práticas de guardar datas em vez de rótulos:

O "since" do cartão é o ano da **primeira** passagem, não da atual. Níckolas
entrou em 2021 para o mestrado e hoje é doutorando: o site diz "PhD student ·
since 2021", que é a informação certa sobre há quanto tempo ele está no grupo.

Nada precisa ser recalculado à mão quando o tempo passa. A lógica está em
`_includes/member_status.html`, que não imprime nada — só define variáveis que
o cartão e a linha de egresso consomem.

**Campos.** `kind` aceita `pi`, `postdoc`, `phd`, `msc` e `undergrad`, com
rótulos em `_data/strings.yml` (`roles:` para a forma longa, `degrees:` para a
curta usada nos egressos). `start` e `end` são **texto entre aspas**, no
formato `'AAAA'` ou `'AAAA-MM'` — sem aspas o YAML converteria em data e o
`slice` do ano deixaria de funcionar. `institution` só aparece quando difere da
UFABC; `funding` e `note` são opcionais.

**Ordem na página.** A separação entre atuais e egressos é automática; a ordem
*dentro* de cada bloco é a ordem do arquivo. É deliberado: ordenar por data em
Liquid exigiria contorções, e a ordem manual permite decidir quem aparece
primeiro.

Fotos vão em `assets/img/members/<slug>.jpg`, quadradas, ao menos 400×400.
Quem não tiver foto aparece com as iniciais — nada quebra.

Para criar um nível novo, acrescente o rótulo em `_data/strings.yml` sob
`roles:` **e** sob `degrees:`. Nenhuma outra mudança é necessária: a página não
conhece a lista de níveis.

### Autores por slug

Quem está em `_data/members.yml` não precisa ser redigitado nas referências.
Em vez de repetir nome e identificadores, passe o slug:

```yaml
author:
    - slug: alves               # nome, ORCID e site vêm de members.yml
    - slug: landulfo
    - name: George E. A. Matsas  # quem é de fora continua escrito à mão
```

As duas formas convivem na mesma lista, e vale para `author`, `advisor`,
`instructor` e `speaker`. Campos escritos ao lado do `slug` têm precedência,
o que permite exceções pontuais.

O ganho é manutenção: corrigir o ORCID de alguém em `members.yml` corrige em
todas as citações do site de uma vez. Um slug inexistente aparece como
"Unknown member: xxx" na página — erro visível, não silencioso.

### Editar as linhas de pesquisa

`_data/research.yml`, com `title` e `body`. O `id` vira âncora, permitindo
links diretos como `/research/#infrared`.

### Adicionar uma tese, dissertação ou monografia

Teses **não têm coleção nem página próprias**: são entradas de `_bibliography/`
com `type: thesis`, e aparecem na página de Publicações misturadas aos artigos,
no grupo do ano correspondente. A ideia é que a página reúna toda a produção
escrita do grupo numa linha do tempo única.

Crie o arquivo com a mesma convenção de nome das outras referências,
`primeiroautorANOPalavrasDoTitulo.md`:

```yaml
---
group: true                  # sem isto não aparece em Publicações
type: thesis
thesis_level: masters        # phd | masters | undergrad

author:
    - name: Ian Bernardes Barcellos

title: Black Holes and the Generalized Second Law of Thermodynamics

advisor:
    - name: André G. S. Landulfo
      lattes: '2705752886744456'   # ou orcid, website, inspire

institution: Federal University of ABC
location: Santo André, SP
date: 2018-01-01             # define o ano e a posição na lista

src: https://...              # link permanente; opcional
---
```

O `thesis_level` vira o rótulo exibido — "Master's dissertation", "Doctoral
thesis" ou "Undergraduate monograph" —, definido em `_data/strings.yml`. O campo
`thesistype`, em texto livre, continua funcionando para referências de terceiros
que você só queira citar num seminário.

O `src` deve ser o endereço permanente no Repositório Institucional da UFABC ou
na BDTD. Com ele, o título vira link; sem ele, fica texto simples e nada quebra.
**Não versione os PDFs**: uma tese passa facilmente de 10 MB, e todo mundo que
clonar o repositório pagaria esse custo para sempre.

> **Nota histórica:** houve uma coleção `_theses/` com três abas separadas
> (Theses, Dissertations, Monographs), depois absorvidas por Publicações. A
> vantagem foi reaproveitar a máquina de citações que já existia —
> `bibitem.html` → `source.html` — em vez de manter um segundo mecanismo em
> paralelo. O histórico do git preserva a versão anterior.

### Adicionar uma disciplina ou notas de aula

A aba **Teaching** tem duas seções, e o campo `kind` decide em qual o
item aparece: `course` para disciplinas e minicursos, `notes` para notas de aula
e outros materiais. Crie um arquivo em `_teaching/`, com o nome no padrão
`ANO-identificador.md`:

```yaml
---
kind: notes                  # course | notes

title: Lectures on the Bondi–Metzner–Sachs group

instructor:
    - name: Níckolas de Aguiar Alves
      orcid: 0000-0002-0309-735X     # ou lattes, website, inspire

date: 2025-04-01             # usado para ordenar
year: 2025                   # exibido, se não houver `term`
term: 2025.1                 # opcional: quadrimestre/semestre, tem precedência

level: Graduate              # opcional
venue: Minicourse at the I São Paulo School on Gravitational Physics

description: >-
  Um parágrafo sobre o conteúdo.

# Lista de links. Use para PDF, vídeo, repositório, página da disciplina.
materials:
    - title: arXiv 2504.12521
      src: https://arxiv.org/abs/2504.12521

# Alternativa a `materials`: um único link, aplicado ao próprio título.
# src: https://...
---
```

Para **hospedar notas de aula no próprio site**, coloque o PDF em
`assets/teaching/` e aponte `src` para `/assets/teaching/arquivo.pdf`. Vale o
mesmo alerta das teses: arquivos grandes ficam no histórico do git para sempre,
então prefira o arXiv ou um repositório institucional quando o material for
volumoso.

Seções sem nenhum item exibem "Nothing listed yet" em vez de ficarem vazias —
a aba já funciona antes de haver conteúdo.

### Criar uma página nova

1. Crie `nome/index.html` com front matter `layout: default`, `title:` e
   `description:`.
2. Em `_data/strings.yml`, acrescente a rota em `paths:` e o rótulo em `nav:`.
3. Em `_layouts/default.html`, adicione o item ao menu — dentro de um
   `<ul class="dropdown-content">` se pertencer a Research ou InfoGraviton.

Sempre referencie a rota por `{{ t.paths.nome | relative_url }}`, nunca com um
caminho absoluto escrito à mão: é isso que permite ao site funcionar tanto na
raiz quanto numa subpasta.

### Trocar a foto da home

Substitua `assets/img/banner-home.jpg`. Ela aparece ao lado do texto de
apresentação, numa coluna de 17rem, recortada em 3:2 pelo centro via
`object-fit: cover` — então o assunto principal deve estar no meio da imagem.
Se mudar as dimensões do arquivo, atualize os atributos `width` e `height` em
`index.html`: eles reservam o espaço e evitam o salto de layout durante o
carregamento. Abaixo de 820px de largura a grade colapsa e a foto vai para
baixo do texto.

---

## Aparência: cores, fontes e tipografia

Tudo começa em **`_sass/base/_variables.scss`**. As cores são custom properties
do CSS declaradas em dois blocos:

```scss
:root                     { --bg: #ffffff;  --text: #1b1b1a;  ... }
:root[data-theme="dark"]  { --bg: #17191c;  --text: #e9e7e2;  ... }
```

O alternador no cabeçalho põe ou remove `data-theme="dark"` no `<html>`, e a
paleta inteira muda. A escolha fica no `localStorage`; na primeira visita, vale
a preferência do sistema operacional. Um script curto no `<head>` aplica o tema
**antes** da primeira pintura, evitando o flash branco.

Para mudar a identidade visual, mexa só nesse arquivo — mantendo os nomes das
variáveis, todo o resto acompanha.

**Fonte:** [Inter](https://rsms.me/inter/), fonte variável carregada do Google
Fonts pelo `<link>` no `<head>` de `_layouts/default.html`. Uma única família
serve todo o site, e as variações vêm de `font-weight` (300 para o título de
destaque, 400 no corpo, 600 nos títulos, 700 na marca).

A família é definida em `--font-sans`. Antes usávamos Montserrat, declarada como
*uma família por peso* (`MontserratLight`, `MontserratSemiBold`…) porque cada
arquivo `.otf` precisava de um `@font-face` próprio — daí o esquema antigo de
variáveis `--font-light`, `--font-semibold` e afins, hoje removido.

Os `.otf` da Montserrat continuam versionados em `assets/fonts/` e declarados em
`_sass/base/_fonts.scss`, sem custo: o navegador só baixa fontes efetivamente
usadas. Para voltar a ela, troque `--font-sans` em `_variables.scss` por
`'MontserratRegular', ...` e remova o `<link>` do Google Fonts.

> **A Inter é a única dependência externa do site.** Se isso incomodar — por
> privacidade ou para não depender de terceiros —, baixe os `.woff2` da Inter
> para `assets/fonts/`, declare-os em `_fonts.scss` e remova o `<link>`. O resto
> do CSS não muda.

**Tipografia do texto corrido:** justificado com hifenização automática. A
hifenização depende do atributo `lang` do `<html>`, que o layout preenche a
partir de `page.lang` — é isso que faz o navegador separar sílabas segundo as
regras de cada idioma.

O recuo de primeira linha segue a convenção de cada tradição, via a variável
`--indent-para`:

```scss
:root            { --indent-para: 0;     }  /* inglês: sem recuo */
html[lang^="pt"] { --indent-para: 2.8em; }  /* português: ~1,25 cm da ABNT */
```

Usar uma variável, e não um seletor por idioma nas regras de parágrafo, é
intencional: `html[lang^="pt"] .prose p` teria especificidade maior que as
exceções (`.prose blockquote p`, `.prose li p`) e voltaria a indentar citações e
listas. Com variável, a especificidade fica plana e as exceções prevalecem.

---

## Publicação

O site é servido pelo **GitHub Pages**, que reconstrói a cada push na branch
principal, através do workflow em `.github/workflows/pages.yml`.

Esse workflow existe por um motivo: o build **clássico** do GitHub Pages usa
Jekyll 3.10 e ignora o `Gemfile` do repositório, enquanto o desenvolvimento
local usa Jekyll 4. Com o workflow, a produção instala exatamente as gems do
`Gemfile`, então o que você vê em `jekyll serve` é o que vai ao ar — e a
[lista fechada de plugins](https://pages.github.com/versions/) do Pages deixa de
limitar o projeto.

### O push compila; a publicação espera aprovação

Um push na `main` **não** coloca o site no ar sozinho. O workflow tem dois
jobs, e só o primeiro roda automaticamente:

**Compilar e verificar** roda sempre. Gera o site, passa o `html-proofer` nos
links internos e escreve um resumo na página do run — quantas páginas saíram,
o tamanho, quantos seminários, e os problemas encontrados. É o que você lê para
decidir.

**Publicar** fica em *Waiting* até alguém aprovar, na própria página do run
(botão **Review deployments**). Você recebe notificação do GitHub quando há
algo esperando.

O efeito prático é que um erro de sintaxe ou um link quebrado aparece para você
antes de aparecer para o público, sem tirar de você a liberdade de dar push
quantas vezes quiser.

### Configuração, uma única vez

1. **Settings → Pages → Source** → escolha **GitHub Actions**.
2. **Settings → Environments → github-pages → Required reviewers** → marque a
   caixa, adicione a si mesmo (e quem mais puder aprovar) → **Save protection
   rules**.

O passo 2 é o que cria o portão. Sem ele o site publica direto a cada push, que
era o comportamento anterior. A proteção de environment é gratuita em
repositórios públicos.

Para reverter ao automático, basta remover os revisores exigidos — nada muda no
workflow. E para publicar sem push, use **Actions → Build e publicação → Run
workflow**.

### Sobre a URL

O repositório se chama `infograviton.github.io`. Num usuário comum ele é
servido em `<usuario>.github.io/infograviton.github.io/`, o que é feio e exige
configurar `baseurl`. Numa **organização chamada `infograviton`**, o mesmo
repositório é reconhecido como site de organização e servido na raiz limpa:

```
https://infograviton.github.io/
```

É para isso que o nome aponta. Ao transferir (**Settings → Transfer
ownership**), lembre-se de que o GitHub redireciona os links do repositório,
**mas não a URL antiga do Pages**, e de atualizar o remote local:

```bash
git remote set-url origin https://github.com/infograviton/infograviton.github.io.git
```

---

## Armadilhas conhecidas

### O formatador do VS Code corrompe o front matter

**A mais perigosa da lista, e já aconteceu.** O formatador de HTML do VS Code
não entende front matter: ele trata o YAML entre os `---` como conteúdo HTML e
"corrige" a indentação ao salvar. Blocos de texto dobrado perdem o recuo:

```yaml
hero_sub: >-
A research group at the Federal University of ABC       ← quebrado
```

O YAML fica inválido, o Jekyll descarta o front matter inteiro e a página perde
título e layout — **sem mensagem de erro**. Já apagou o título da home em
inglês uma vez.

Proteção: `.vscode/settings.json` desliga o `formatOnSave` para HTML, YAML e
Markdown **neste workspace**. Está versionado, então protege todos os
colaboradores. Não remova. Alternativa, se quiser manter a formatação: instale a
extensão **Liquid** (Shopify), que reconhece os arquivos como Liquid e respeita
o front matter.

### `bundle install` falha compilando gem nativa (yajl-ruby, nokogiri…)

Sintoma típico, em Ruby recente:

```
github-pages was resolved to 8, which depends on
  jekyll was resolved to 1.2.0, which depends on
    pygments.rb ... yajl-ruby
error: 'rb_cFixnum' undeclared
```

**Causa.** A gem `github-pages` fixa Jekyll 3.10 e um conjunto de dependências
antigas. Em Ruby 4.x elas não compilam, o Bundler recua na resolução até
`github-pages` **8** (de 2013) e tenta compilar `yajl-ruby`, que usa
`rb_cFixnum` — constante removida do Ruby na versão 2.4.

**Solução, já aplicada neste repositório:** o `Gemfile` usa **Jekyll puro**
(`gem "jekyll", ">= 4.3", "< 5"`) em vez de `github-pages`. É seguro porque
este site não usa nenhum plugin do GitHub Pages, e porque o build no servidor
ignora o `Gemfile` de qualquer forma.

Se o erro voltar depois de mexer nas dependências, apague o lock e resolva de
novo:

```bash
rm Gemfile.lock        # Remove-Item Gemfile.lock no PowerShell
bundle install
```

Um `Gemfile.lock` gerado pela era `github-pages` é incompatível com o `Gemfile`
atual e provoca exatamente esse recuo de versões.

> **Consequência a aceitar:** o Jekyll local (4.x) não é idêntico ao do
> servidor (3.10 via `github-pages`). Para este site a diferença é inócua — só
> se usam recursos presentes nas duas versões. Mas se algum dia adicionar
> plugins, confirme antes que estão na
> [lista permitida do GitHub Pages](https://pages.github.com/versions/).

### O `Gemfile.lock` precisa declarar todas as plataformas

O Bundler registra no lock as plataformas para as quais resolveu as gems. Um
`bundle install` rodado só no Windows produz:

```
PLATFORMS
  x64-mingw-ucrt
```

Isso **quebra o GitHub Actions** (que roda em `ubuntu-latest`) e o
`bundle install` de qualquer colaborador em Linux ou macOS. Depois de mudar
dependências, acrescente as plataformas — o comando só registra, não instala:

```bash
bundle lock --add-platform x86_64-linux
bundle lock --add-platform aarch64-linux
bundle lock --add-platform arm64-darwin
bundle lock --add-platform x86_64-darwin
```

O `Gemfile.lock` **deve ser commitado**: é ele que garante que todos usem as
mesmas versões e que o cache de gems do CI funcione.

### Windows não tem base de fusos horários

O `_config.yml` declara `timezone: America/Sao_Paulo`. O Windows não possui a
base *zoneinfo* que existe no Unix, e o Jekyll falha com
`No source of timezone data could be found`. O `Gemfile` resolve com um bloco
condicional:

```ruby
platforms :mingw, :x64_mingw, :mswin, :jruby do
  gem "tzinfo", ">= 1", "< 3"
  gem "tzinfo-data"
end
```

Só instala no Windows; Linux e macOS ignoram. (`:windows` substituiu os antigos
`:mingw`, `:x64_mingw` e `:mswin`, que o Bundler agora reporta como
depreciados.) O `Gemfile` também declara `csv`, `base64`, `bigdecimal` e
`logger` — removidas do conjunto padrão a partir do Ruby 3.4, mas ainda usadas
pelo Jekyll — e `webrick`, fora do stdlib desde o Ruby 3.0.

### Alterações no `_config.yml` não recarregam

O arquivo é lido uma única vez, na inicialização. Reinicie o servidor.

### Quebras de linha CRLF

Alguns arquivos do repositório original usam CRLF (`\r\n`). O Jekyll lida bem
com isso, mas scripts e `sed` que esperam `\n` falham em silêncio. Se uma
edição automatizada "não pegou" num arquivo específico, verifique com
`file` ou `cat -A`.

### `page` dentro de um include

Includes veem o `page` da **página que os chamou**, não do item que estão
renderizando. Em `typeset_date_lang.html`, `page.lang` é o idioma da página
sendo montada — o que é justamente o desejado ali, mas confunde na primeira
leitura.

### Uma página em branco quase sempre é YAML

Nesta ordem: front matter inválido (indentação), depois indentação em `_data/`,
depois um `{% endif %}` faltando. Rode `bundle exec jekyll build --verbose` e
leia a primeira mensagem, não a última.

---

## Glossário

| Termo | Significado |
|---|---|
| **Jekyll** | O gerador: lê os fontes, escreve `_site/` |
| **build** | O ato de gerar o site; local ou no servidor do GitHub |
| **site estático** | HTML pronto, sem banco de dados nem código no servidor |
| **front matter** | Bloco YAML entre `---` no topo de um arquivo |
| **Liquid** | Linguagem de template: `{{ imprime }}`, `{% executa %}` |
| **layout** | Molde de página inteira, com `{{ content }}` no meio |
| **include** | Trecho reutilizável de template, com parâmetros |
| **coleção** | Conjunto de itens, um arquivo cada (`_seminars/`) |
| **arquivo de dados** | Lista num único YAML (`_data/members.yml`) |
| **gem** | Pacote Ruby |
| **Bundler** | Gerenciador de gems; `bundle exec` usa as versões travadas |
| **SCSS** | CSS estendido, compilado no build |
| **custom property** | Variável CSS (`--bg`), viva no navegador |
| **slug** | Identificador curto usado em URLs e nomes de arquivo |

---

## Onde pedir ajuda

- [Documentação do Jekyll](https://jekyllrb.com/docs/) — comece por
  *Structure*, *Front Matter* e *Collections*
- [Referência do Liquid](https://shopify.github.io/liquid/) — a lista de
  filtros é a parte mais consultada
- [Filtros próprios do Jekyll](https://jekyllrb.com/docs/liquid/filters/) —
  `where_exp`, `group_by_exp`, `relative_url`
- [Versões e plugins do GitHub Pages](https://pages.github.com/versions/) —
  o que pode ser usado em produção
