source "https://rubygems.org"

# ---------------------------------------------------------------------------
# Jekyll puro, e NÃO a gem `github-pages`.
#
# A gem github-pages existe para espelhar localmente o ambiente do servidor,
# mas ela fixa Jekyll 3.10 e um conjunto de dependências antigas que não
# compilam em Ruby recente. Em Ruby 4.x o Bundler não consegue satisfazer o
# conjunto, recua até github-pages 8 (de 2013) e a instalação morre em
# yajl-ruby com "rb_cFixnum undeclared" — constante removida do Ruby em 2.4.
#
# Trocar por Jekyll moderno é seguro aqui por dois motivos:
#   1. este site não usa nenhum plugin do GitHub Pages (nem tema remoto);
#   2. o build do GitHub Pages ignora este Gemfile e usa o próprio conjunto
#      de gems — ou seja, o Gemfile só afeta o desenvolvimento local.
# ---------------------------------------------------------------------------
gem "jekyll", ">= 4.3", "< 5"

# O Windows não possui a base de fusos horários (zoneinfo) nativa do Unix.
# Sem estas gems, o Jekyll falha com "No source of timezone data could be found"
# ao ler `timezone: America/Sao_Paulo` do _config.yml.
# (`:windows` substitui os antigos :mingw/:x64_mingw/:mswin, hoje depreciados.)
platforms :windows, :jruby do
  gem "tzinfo", ">= 1", "< 3"
  gem "tzinfo-data"

  # Sem a wdm, o Jekyll monitora arquivos por polling no Windows — funciona,
  # mas consome CPU e demora a notar mudanças. Com ela, usa a API nativa.
  gem "wdm", ">= 0.1.0"
end

# Bibliotecas removidas do conjunto padrão do Ruby em versões recentes,
# mas ainda usadas pelo Jekyll e suas dependências.
gem "csv"
gem "base64"
gem "bigdecimal"
gem "logger"

# Servidor de desenvolvimento (fora do stdlib desde o Ruby 3.0).
gem "webrick", "~> 1.8"
