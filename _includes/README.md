# _includes/

Fragmentos reutilizáveis de template, chamados com
`{% include arquivo.html param=valor %}`. É aqui que vive a "maquinaria" de
citações e as peças das páginas.

## Conteúdo

Sistema de citações (herdado do InfoGraviton e usado por seminários,
sugestões e publicações):

- `bibitem.html` — ponto de entrada: renderiza uma referência completa.
- `source.html` — formata por tipo (`article`, `book`, `thesis`, `tv`…).
- `authorlist.html` / `authorname.html` — lista de autores; nomes viram links
  se a entrada tiver `website`, `inspire`, `orcid` ou `lattes`.
- `journaldata.html`, `eprintdata.html`, `hyperlink.html`, `pubstate.html` —
  detalhes de periódico, arXiv/INSPIRE, DOI e estado de publicação.
- `relative_or_absolute_url.html`, `resourcefile.html` — resolução de links e
  anexos.

Peças das páginas do grupo:

- `member_card.html` — cartão de um membro (foto ou iniciais, cargo, links).
  Recebe `member` e `lang`.
- `members_section.html` — uma seção de membros por papel (`role`) e idioma.
- `publication_list.html` — publicações do grupo (entradas de `_bibliography/`
  com `group: true`), agrupadas por ano, "em preparação" ao final.
- `include_seminar.html` — item de seminário nas listagens; com
  `abstract=true` mostra o resumo truncado.
- `typeset_date_lang.html` — datas: futuras com hora e fuso, passadas só com o
  dia; formato segue o idioma da página.

## Como modificar

Tipo novo de referência: acrescente um `when` em `source.html`. Campo novo no
cartão de membro: edite `member_card.html` e adicione o dado em
`_data/members.yml`. Regra prática: se algo aparece em mais de uma página,
pertence a um include; se aparece numa só, pode ficar na própria página.
