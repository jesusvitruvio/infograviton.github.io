# _data/

Dados estruturados do site, em YAML. O Jekyll os expõe aos templates como
`site.data.<arquivo>`. Nada aqui vira página por si só — são as páginas e os
layouts que leem estes arquivos.

## Conteúdo

- `members.yml` — os membros do grupo. **Fonte única**: as páginas de membros
  em inglês e em português são geradas deste arquivo. Cada entrada tem `slug`
  (nome do arquivo da foto), `role` (`pi`, `postdoc`, `phd`, `masters`),
  instituição, Lattes/ORCID/site, agência de fomento e um blurb em cada idioma
  (`blurb_en` / `blurb_pt`).
- `research.yml` — as linhas de pesquisa, com `title_en`/`title_pt` e
  `body_en`/`body_pt`. Alimenta /research/ e /pt/pesquisa/.
- `i18n.yml` — tudo que é texto de interface: rótulos de navegação, títulos do
  site, nomes de papéis, rotas (`paths`) de cada idioma. Os templates acessam
  via `site.data.i18n[page.lang]`.

## Como modificar

Membro novo: acrescente uma entrada em `members.yml` (as duas línguas se
atualizam sozinhas) e, se houver foto, salve-a em `assets/img/members/`.
Linha de pesquisa nova: uma entrada em `research.yml`, sempre com os textos nos
dois idiomas. Página nova no site: registre a rota nos `paths` de **ambos** os
idiomas em `i18n.yml`, senão o menu quebra a paridade.

Cuidado com indentação: YAML é sensível a espaços, e o formatador do VS Code
está desativado neste workspace justamente para não corrompê-la.
