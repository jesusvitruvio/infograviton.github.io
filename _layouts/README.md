# _layouts/

Moldes de página. Toda página do site declara (ou herda via `defaults` do
`_config.yml`) um `layout`, e o conteúdo dela é injetado no molde no lugar de
`{{ content }}`.

## Conteúdo

- `default.html` — o molde de tudo: `<head>` (MathJax, CSS, script anti-flash
  do tema escuro), cabeçalho com navegação bilíngue, seletor de idioma, botão
  de modo escuro, e rodapé institucional. Lê os rótulos de
  `site.data.i18n[page.lang]` e o link do outro idioma de `page.alt`.
- `seminar.html` — molde dos seminários (herda o default). Renderiza título,
  data/palestrante, resumo, a lista de referências (resolvidas contra
  `_bibliography/` pelas chaves em `bibliography:`) e anexos.

## Como modificar

Item novo no menu: edite a lista `nav-list` no `default.html` **e** acrescente
o rótulo em `_data/i18n.yml` nos dois idiomas. Mudanças de aparência
normalmente pertencem a `_sass/`, não aqui. Se criar um layout novo, associe-o
às páginas via front matter (`layout: nome`) ou por `defaults` no
`_config.yml`.

Atenção ao front matter dos layouts (o bloco `---` no topo do `seminar.html`):
o formatador automático de HTML corrompe esse bloco — está desativado neste
workspace via `.vscode/settings.json`.
