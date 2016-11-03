# Gerenciador de Versões Python Simples: pyenv

[![Participe do chat em https://gitter.im/yyuu/pyenv](https://badges.gitter.im/Join%20Chat.svg)](https://gitter.im/yyuu/pyenv?utm_source=badge&utm_medium=badge&utm_campaign=pr-badge&utm_content=badge)

[![Build Status](https://travis-ci.org/yyuu/pyenv.svg?branch=master)](https://travis-ci.org/yyuu/pyenv)

> Tradução da [documentação oficial do pyenv](https://github.com/yyuu/pyenv)

O pyenv permite que vocẽ alterne facilmente entre múltiplas versões do Python. É simples, não-obtrusivo e segue a tradição do UNIX de fazer apenas uma coisa e fazê-la bem feita.

Esse projeto foi feito a partir de um _fork_ do [rbenv](https://github.com/rbenv/rbenv) e do
[ruby-build](https://github.com/rbenv/ruby-build), modificando-os para Python.

![Screenshot](https://i.gyazo.com/699a58927b77e46e71cd674c7fc7a78d.png)


### O que o pyenv _faz…_

* Permite que você **modifique a versão global do Python** de cada usuário.
* Oferece suporte para **versões de Python por projeto**.
* Permite que você **sobrescreva a versão do Python** com uma variável de ambiente
* Busca comandos em **diferentes versões do Python de uma só vez**. Isso pode ser útil para rodar testes em múltiplas versões do Python com [tox](https://pypi.python.org/pypi/tox).


### Diferentemente do pythonbrew e pythonz, o pyenv _não faz…_

* **Depende do próprio Python.** O pyenv foi feito puramente em shell script. Não há problema de _bootstrap_ com o Python.
* **Precisa ser carregado no shell.** Ao invés disso, a abordagem do _shim_ funciona adicionando um diretório ao seu `$PATH`.
* **Gerencia virtualenv.** Claro, você pode criar [virtualenv](https://pypi.python.org/pypi/virtualenv)
    você mesmo, ou usar o [pyenv-virtualenv](https://github.com/yyuu/pyenv-virtualenv)
    para automatizar o processo.


----


## Índice

* **[Como funciona](#como-funciona)**
  * [Entendendo o PATH](#entendendo-o-PATH)
  * [Entendendo shims](#entendendo-Shims)
  * [Escolhendo a versão do Python](#escolhendo-a-versão-do-Python)
  * [Identificando o local de instalação do Python](#identificando-o-local-de-instalação-do-python)
* **[Instalação](#instalação)**
  * [Checkout básico do GitHub](#checkout-básico-do-github)
    * [Atualizando](#atualizando)
    * [Homebrew no Mac OS X](#homebrew-no-mac-os-x)
    * [Configurações avançadas](#configuracoes-avancadas)
    * [Desinstalando versões do Python](#desinstalando-versoes-do-python)
* **[Guia de commandos](#guia-de-comandos)**
* **[Desenvolvimento](#desenvolvimento)**
  * [Histórico de versões (em inglês)](https://github.com/yyuu/pyenv/blob/master/CHANGELOG.md)
  * [Licença (em inglês)](https://github.com/yyuu/pyenv/blob/master/LICENSE)

----


## Como funciona

De maneira geral o pyenv intercepta os comandos Python usando executáveis
shim injetados em seu `PATH`, determinando qual versão do Python foi especificada
pela sua aplicação, e em seguinda passando seus comandos à instalação correta do Python.


### Entendendo o PATH

Quando você roda um comando como `python` or `pip`, seu sistema operacional
procura em uma lista de diretórios para encontrar um arquivo executável com aquele
nome. ESsa lista de diretórios está em uma variável de ambiente chamada `PATH`, com um sinal de dois pontos separando cada diretório na lista:

    /usr/local/bin:/usr/bin:/bin


A busca em diretórios do `PATH` é feita da esquerda para a direita, então um 
executável encontrado no começo da lista tem prioridade sobre um exectável encontrado no final dela.
Nesse exemplo, a busca acontece primeiro no diretório `/usr/local/bin`, depois no `/usr/bin` e
por fim no `/bin`.

### Entendendo shims

O pyenv trabalha inserindo um diretório de _shims_ na frente de seu
`PATH`:

    ~/.pyenv/shims:/usr/local/bin:/usr/bin:/bin

Através de um processo chamado _rehashing_, o pyenv mantém shims naquele
diretório para bater cada comando Python com as versões instaladas do Python — `python`, `pip`, e assim por diante.

Shims são executáveis leves que simplesmente passam seu comando pelo pyenv. Então
com o pyenv instalado, quando você roda, por exemplo, `pip`, o seu sistema operacional
faz o seguinte:

* Procuram o seu `PATH` por um executável chamado `pip`
* Encontra o pyenv shim chamado `pip` no começo do seu `PATH`
* Roda o shim chamado `pip`, que passa o comando junto ao pyenv 

### Escolhendo a versão do Python

Quando você executa um shim, o pyenv determinal qual versão do Python usar procurando nos seguintes locais, nesta ordem:

1. A variável de ambiente (se especificada) `PYENV_VERSION`. Você pode usar o comando  [`pyenv shell`](COMMANDS.md#pyenv-shell)  para configurar esta variável de ambiente na sua sessã atual do shell.

2. O arquivo específico de aplicação`.python-version` no diretório atual (se presente). Você pode modificar o arquivo do diretório atual `.python-version` com o  [`pyenv local`](COMMANDS.md#pyenv-local)

3. O primeiro arquivo `.python-version`  encontrado (se existir) procurando em cada diretório pai, até chegar à raíz do seu sistema de arquivos.

4. O arquivo global `~/.pyenv/version` . Você pode modificar este arquivo usando o comando [`pyenv global`](COMMANDS.md#pyenv-global). Se a versão global do arquivo não estiver presente, o pyenv assume que você quer usar o Python nativo do seu sistema operacional. (Em outras palavras, qualquer versão que rodaria se o pyenv não estivesse no seu `PATH`.)

**Nota:** Você pode ativar múltiplas versões ao mesmo tempo, incluindo múltiplas versões do Python 2 ou Python 3 simultaneamente. Isso permite a utilização paralela de Python 2 ou Python 3, o que é obrigatório com ferramentas como `tox`. Por exemplo, para configurar seu path para primeiramente usar o Python nativo do seu sistema operacional e depois usar o Python 3 (configurado para 2.7.9 e 3.4.2 neste exemplo), mas ter também Python 3.3.6, 3.2 e 2.5 disponíveis em seu `PATH`, você iria primeiro instalar as verões que faltam com `pyenv install`, e depois configurar `pyenv global system 3.3.6 3.2 2.5`. A partir de então é possível encontrar o caminho completo do executável destas versões do Python utilizando `pyenv which`. Por exemplo: `pyenv which python2.5` deve exibir `$PYENV_ROOT/versions/2.5/bin/python2.5`, e `pyenv which python3.4` deve exibir o caminho para Python 3 do sistema.


### Identificando o local de instalação do Python

Uma vez que o pyenv determinou qual versão do Python de acordo com a especificação da sua aplicação, o pyenv passa seus comandos para a instalação correspondente do Python. Cada versão do Python é instalada em seu próprio diretório em:

    ~/.pyenv/versions

Por exemplo, você pode ter estas versões instaladas:

* `~/.pyenv/versions/2.7.8/`
* `~/.pyenv/versions/3.4.2/`
* `~/.pyenv/versions/pypy-2.4.0/`

Para o pyenv os nomes das versão são simplesmente diretórios em: `~/.pyenv/versions`.

----

## Instalação

Se você estiver em um Mac OS X, considere [instalar usando o Homebrew](#homebrew-on-mac-os-x).

### O instalador automático

Visite meu outro projeto:
https://github.com/yyuu/pyenv-installer

### Checkout básico do GitHub

Esse procedimento instala a última versão do pyenv e torna mais fácil fazer um _fork_ e contribuir para o projeto:

1. **Faça checkout do pyenv onde você deseja que ele seja instalado.**
   Um bom lugar para isso é `$HOME/.pyenv` (porém você pode instalá-lo onde quiser).

        $ git clone https://github.com/yyuu/pyenv.git ~/.pyenv


2. **Defina a variável de ambiente `PYENV_ROOT`** para apontar para o caminho onde o repositório pyenv
 está clonado e adicione `$PYENV_ROOT/bin` em seu`$PATH` para ter acesso ao utilitário de linha de comando `pyenv`.

        $ echo 'export PYENV_ROOT="$HOME/.pyenv"' >> ~/.bash_profile
        $ echo 'export PATH="$PYENV_ROOT/bin:$PATH"' >> ~/.bash_profile

    **Nota para quem usa Zsh**: Modifique seu arquivo `~/.zshenv` ao invés de `~/.bash_profile`.  
    **Nota para Ubuntu e Fedora**: Modifique seu arquivo `~/.bashrc` ao invés de `~/.bash_profile`.

3. **Adicione `pyenv init` ao seu shell** para habilitar shims e completar comandos automaticamente.
   Por favor certifique-se de que `eval "$(pyenv init -)"` está presente no fim da configuração do shell já que o `PATH` é manipulado durante a inicialização

        $ echo 'eval "$(pyenv init -)"' >> ~/.bash_profile
    
    **Aviso Geral**: Em alguns sistemas a variável `BASH_ENV` é configurada apontando
    para `.bashrc`. Nesses sistemas provavelmente você deve colocar o `eval "$(pyenv init -)` mencionado acima  no `.bash_profile`, e **não** em `.bashrc`. Caso contrário você pode 
     ter commportamentos inesperados, como um acabar com o pyenv em loop infinito. 
    Veja [#264](https://github.com/yyuu/pyenv/issues/264) para mais detalhes.

4. **Reinicie seu shell para que as mudanças no PATH tenham efeito.**
   Agora você pode começar a usar o pyenv.

        $ exec $SHELL

5. **Instale as versões do Python dentro de `$PYENV_ROOT/versions`.**
   Por exemplo, para baixar e instalar Python 2.7.8, execute:

        $ pyenv install 2.7.8

   **NOTA:** Se você precisar passar opçãos de configuração, por favor use
   a variável de ambiente ```CONFIGURE_OPTS```.
   
   **NOTA:** Se você quiser usar um proxy para baixar as versões, por favor use as variáveis de ambiente `http_proxy` e `https_proxy`.

   **NOTA:** Se você estiver com problemas para instalar alguma versão do Python, por favor visite a página wiki
   [Problemas comuns ao rodar](https://github.com/yyuu/pyenv/wiki/Common-build-problems)

#### Atualizando

Se você instalou pyenv usando as instruções acima, você pode atualizar sua instalação à qualquer momento usando
git.

Para atualizar para a última versão de desenvolvimento do pyenv, use `git pull`:

    $ cd ~/.pyenv
    $ git pull

Para atualizar para uma versão específica do pyenv, faça checkout da tag correspondente:

    $ cd ~/.pyenv
    $ git fetch
    $ git tag
    v0.1.0
    $ git checkout v0.1.0

### Desinstalando o pyenv

A simplicidade do pyenv torna fácil a sua desabilitação temporária ou mesmo desinstalá-lo do sistema.

1. Para **desabilitar** o pyenv gerenciando as versões do Python, simplesmente remova a linha 
  `pyenv init` da sua configuração de inicialização do shell. Isso removerá os diretórios shims do pyenv do PATH e futuros 
   comandos como `python` irão executar a versão do Python que o sistema tinha antes do pyenv.

  `pyenv` irá ainda estar acessível na linha de comando, mas suas aplicações Python não serão afetadas pela
   troca de versão.

2. Para **desinstalar** o pyenv por completo, execute o passo 1 e então remova o seu diretório raíz.
   Isso irá **apagar todas as versões Python** que estavam instaladas no diretório`` `pyenv root`/versions/ ``:

        rm -rf `pyenv root`

   Se você instalou pyenv usando um gerenciador de pacotes, como uma última etapa, execute a remoção do pacote pyenv.
   Segue exemplo pelo Homebrew:

        brew uninstall pyenv

## Guia de Comandos

### Homebrew no MAC OS X

Você também pode instalar o pyenv usando o gerenciador de pacotes [Homebrew](http://brew.sh) para o Mac OS X.

    $ brew update
    $ brew install pyenv


Para atualizar o pyenv no futuro, utilize `upgrade` ao invés de `install`.

Depois da instalação, você precisará adicionar `eval "$(pyenv init -)"` ao seu perfil (como informado pelo  
Homebrew — para exibir essas informações novamente, utilize `brew info pyenv`). Você só precisa adicioná-lo ao seu perfil uma vez.

Então, siga o resto dos passos pós-instalação em [Checkout básico do GitHub](#checkout-basico-do-github) acima, começando com a instrução nº 4 (_reinicie seu shell para que as mudanças no PATH tenham efeito_).

### Configurações avançadas

Pule essa seção ao menos que você precise saber o que cada linha do seu shell está fazendo

`pyenv init` é o único comando que ultrapassa a linha de carregar novos comandos no seu shell.
 Vindos do rvm, alguns de vocês devem ser contrários à esta ideia. Segue o que o na verdade `pyenv init` faz:

1. **Configura os locais dos shims.** Este é o único requisito para o pyenv funcionar adequadamente. Você pode fazer isso colocando 
   `~/.pyenv/shims` no início do seu `$PATH`.

2. **Instala completar de forma automática na linha de comando.** Isto é completamente opcional mas muito útil.
   Rodando o script com `~/.pyenv/completions/pyenv.bash` essa estapa configura o completar automático na linha de comando. 
   Há também um `~/.pyenv/completions/pyenv.zsh` para usuários Zsh.

3. **Refazer shims.** De tempos em tempos você precisa re-executar seus arquivos shim. Fazendo isso na inicialização é uma formda de garantir que tudo está atualizado. Você sempre pode executar `pyenv rehash` manualmente no entanto.

4. **Instalar o despachante (_dispatcher_) sh.** Isso também é opcional, mas permite ao pyenv e seus plugins mudar variáveis no seu shell, tornando comandos como `pyenv shell` possíveis. O despachante não faz nada anormal como sobrescrever `cd` ou hackear seu prompt do shell, mas se por alguma razão você precisar que o `pyenv` seja um script real ao invés de uma função shell, você pode pular essa etapa sem problemas.

Para ver exatamente o que ocorre por baixo dos panos, execute `pyenv init -`.

### Desinstalando versões do Python

Com o tempo você irá acumular versões do python em seu diretório
`~/.pyenv/versions`.

O comando `pyenv uninstall` automatiza automatiza o processo de remoção de versões antigas do Python.

Como forma alternativa, simplesmente utilize `rm -rf` no diretório da versão que você deseja remover. Você pode encontrar o diretório de uma versão específica do Python com o comando
 `pyenv prefix` , Exemplo: `pyenv prefix 2.6.8`.

----

## Guia de commandos

Veja [COMMANDS.md](COMMANDS.md).

----

## Variáveis de ambiente

Você pode modificar como o pyenv funciona com as seguintes configurações:

nome | padrão | descrição
-----|---------|------------
`PYENV_VERSION` | | Especifica a versão do python utilizada.<br>Veja também [`pyenv shell`](#pyenv-shell)
`PYENV_ROOT` | `~/.pyenv` | Define o diretório onde focarão as versões do Python e o shims.<br>Veja também `pyenv root`
`PYENV_DEBUG` | | Informações sobre os retornos de depuração.<br>Veja também: `pyenv --debug <subcommand>`
`PYENV_HOOK_PATH` | [_see wiki_][hooks] | Lista de locais, separados por vírgulas, a serem buscados pelos ganchos (_hooks_) do pyenv.
`PYENV_DIR` | `$PWD` | Diretório para iniciar a busca por arquivos`.python-version`.

## Desenvolvimento

O código fonte do pyenv está [hospedado no GitHub](https://github.com/yyuu/pyenv).  Ele é leve, modular, e fácil de entender, mesmo se você não for um hacker de linha de comando.

Os testes são executados usando [Bats](https://github.com/sstephenson/bats):

    $ bats test
    $ bats/test/<file>.bats

Por favor, fique à vontade para submeter sugestões e bugs em como [_issues_](https://github.com/yyuu/pyenv/issues).

  [pyenv-virtualenv]: https://github.com/yyuu/pyenv-virtualenv#readme
  [hooks]: https://github.com/yyuu/pyenv/wiki/Authoring-plugins#pyenv-hooks
