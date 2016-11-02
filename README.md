# Gerenciador de Versões Python Simples: pyenv

[![Participe do chat em https://gitter.im/yyuu/pyenv](https://badges.gitter.im/Join%20Chat.svg)](https://gitter.im/yyuu/pyenv?utm_source=badge&utm_medium=badge&utm_campaign=pr-badge&utm_content=badge)

[![Build Status](https://travis-ci.org/yyuu/pyenv.svg?branch=master)](https://travis-ci.org/yyuu/pyenv)

pyenv permite que vocẽ facilmente mude entre múltiplas versões do Python. É simples, não-obtrusiva e segue a tradição de fazer uma coisa e fazê-la bem do UNIX.

Esse projeto foi feito "fork" de: [rbenv](https://github.com/rbenv/rbenv) and
[ruby-build](https://github.com/rbenv/ruby-build), e modificado para Python.

<img src="https://i.gyazo.com/699a58927b77e46e71cd674c7fc7a78d.png" width="735" height="490" />


### pyenv _faz..._

* Permite você **modificar a versão global do Python** em uma base por usuário. [per-user basis]
* Fornece suporte para **versões de Python por projeto**.
* Permite você **sobrescrever a versão do Python** com uma variável de ambiente
* Procura comandos de **versões diversas do Python de uma vez**.
  Isso pode ser útil testar entre as versões do Python com [tox](https://pypi.python.org/pypi/tox).


### Diferentemente do pythonbrew e pythonz, o pyenv não faz..._

* **Depende do próprio Python.** pyenv foi feito puramente de shell scripts.
    Não há problema de python bootstrap.
* **Precisa ser carregado no shell.** Ao invés disso, a abordagem do shim funciona adicionando um diretório em seu `$PATH`.
* **Gerencia virtualenv.** Claro, você pode criar [virtualenv](https://pypi.python.org/pypi/virtualenv)
    você mesmo, ou [pyenv-virtualenv](https://github.com/yyuu/pyenv-virtualenv)
    para automatizar o processo.


----


## Tabela de Conteúdos

* **[Como funciona](#como-funciona)**
  * [Entendendo o PATH](#entendendo-o-PATH)
  * [Entendendo Shims](#entendendo-Shims)
  * [Escolhendo a versão do Python](#escolhendo-a-versão-do-Python)
  * [Localizando o Caminho de Instalação do Python](#localizando-o-caminho-de-instalação-do-python)
* **[Instalação](#instalação)**
  * [Checkout básico do github](#checkout-básico-do-github)
    * [Atualizando](#atualizando)
    * [Homebrew on Mac OS X](#homebrew-on-mac-os-x)
    * [Configurações Avançadas](#configuracoes-avancadas)
    * [Desinstalando Versões do Python](#desinstalando-versoes-do-python)
* **[Comandos de referência](#comandos-de-referência)**
* **[Desenvolvimento](#desenvolvimento)**
  * [Version History](#version-history)
  * [License](#license)


----


## Como funciona
Por cima, o pyenv intercepta os comandos Python usando executáveis
shim injetados em seu `PATH`, determinando qual versão do Python foi especificada
pela sua aplicação, e passa seus comandos à instalação correta do Python.


### Entendendo o PATH
Quando você roda um comando como  `python` or `pip`, seu sistema operacional
procura em uma lista de diretórios para encontrar um arquivo executável com aquele
nome. ESsa lista de diretórios está em uma variável de ambiente chamada `PATH`, com cada
diretório na lista separada por dois pontos:

    /usr/local/bin:/usr/bin:/bin


Diretórios em `PATH` são buscados da esquerda para a direita, então um 
executável encontrado no começo da lista tem prioridade sob outra no final.
Neste exemplo, o diretório `/usr/local/bin` será buscado primeiro, depois `/usr/bin` e
então `/bin`.



### Entendendo Shims

pyenv trabalha inserindo um diretório de _shims_ na frente de seu
`PATH`:

    ~/.pyenv/shims:/usr/local/bin:/usr/bin:/bin

Através de um processo chamado _rehashing_, o pyenv mantém shims naquele
diretório para bater com cada comando Python pelas versões instaladas do Python - `python`, `pip`, e assim por diante.

Shims são executáveis leves que simplesmente passam seu comando junto ao pyenv. Então
com o pyenv instalado, quando você roda, por exemplo, `pip`, o seu sistema operacional
faz o seguinte:

* Procuram o seu `PATH` por um executável chamado `pip`
* Encontra o pyenv shim chamado `pip` no começo do seu `PATH`
* Roda o shim chamado `pip`, que passa o comando junto ao pyenv 

### Escolhendo a versão do Python

Quando você executa um shim, o pyenv determinal qual versão do python usar lendo nos seguintes caminhos, nesta ordem:
1. A variável de ambiente(se especificada) `PYENV_VERSION`. Você pode usar o comando  [`pyenv shell`](https://github.com/yyuu/pyenv/blob/master/COMMANDS.md#pyenv-shell)  para configurar esta variável de ambiente na sua sessã atual do shell.

2. O arquivo específico de aplicação`.python-version` no diretório atual(se presente). Você pode modificar o arquivo do diretório atual `.python-version` com o  [`pyenv local`](https://github.com/yyuu/pyenv/blob/master/COMMANDS.md#pyenv-local)


3. O primeiro arquivo `.python-version`  encontrado (se existir) procurando em cada diretório pai, até chegar à raíz do seu sistema de arquivos.
4. O arquivo global `~/.pyenv/version` . Você pode modificar este arquivo usando o comando
    [`pyenv global`](https://github.com/yyuu/pyenv/blob/master/COMMANDS.md#pyenv-global). Se a versão global do arquivo não estiver presente, o pyenv assume que você quer usar o "system"
   Python. (Em outras palavras, quaisquer versões que rodariam se o pyenv não esteve em seu
   `PATH`.)


**Nota:** Você pode ativar múltiplas versões ao mesmo tempo, incluindo múltiplas versões do Python2 ou Python3 simultaneamente. ISso permite a utilização paralela de Python2 ou Python3, e é obrigatório com ferramentas como `tox`. Por exemplo, para configurar seu path para primeiramente usar seu Python e Python2 `sistema`(system) (configure para 2.7.9 e 3.4.2 neste exemplo), mas também tenha Python 3.3.6, 3.2 e 2.5 disponíveis em seu `PATH`, um iria primeiro `pyenv install` as versões que faltam, então configurar `pyenv global system 3.3.6 3.2 2.5`. Neste momento, deve ser possível encontrar o caminho completo do executável destes utilizando `pyenv which`.
Exemplo: `pyenv which python2.5`
(deve exibir `$PYENV_ROOT/versions/2.5/bin/python2.5`) ou `pyenv which python3.4` (deveria exibir o caminho para Python3 do sistema).


### Localizando o Caminho de Instalação do Python

UMa vez que o pyenv determinou qual versão do Python sua aplicação especificou, passa o comendo junto à instalação correspondente do Python.
Cada versão do Python é instalada em seu próprio diretório em:
`~/.pyenv/versions`.

Por exemplo, você pode ter estas versões instaladas:

* `~/.pyenv/versions/2.7.8/`
* `~/.pyenv/versions/3.4.2/`
* `~/.pyenv/versions/pypy-2.4.0/`

Até onde se sabe do pyenv, nomes de versão são simplesmente diretórios em:
`~/.pyenv/versions`.


----


## Instalação

Se você estiver em um Mac OS X, considere [instalando com Homebrew](#homebrew-on-mac-os-x).


### O instalador automático

Visite o projeto:
https://github.com/yyuu/pyenv-installer


### Checkout básico do github

Isso fará você ter a última versão do pyenv e tormar mais fácil realizar um "fork" e contribuir para mudanças

1. **Faça checkout do pyenv onde você deseja que instale.**
   Um bom lugar para escolher é `$HOME/.pyenv` (porém você pode instalá-lo em outro lugar).

        $ git clone https://github.com/yyuu/pyenv.git ~/.pyenv


2. **Defina a variável de ambiente `PYENV_ROOT`** para apontar para o caminho onde o repositório pyenv
 está clonado e adicione `$PYENV_ROOT/bin` em seu`$PATH` para acesso ao utilitário de linha de comando `pyenv`.

        $ echo 'export PYENV_ROOT="$HOME/.pyenv"' >> ~/.bash_profile
        $ echo 'export PATH="$PYENV_ROOT/bin:$PATH"' >> ~/.bash_profile

    **Nota Zsh**: Modifique seu arquivo `~/.zshenv` ao invés de `~/.bash_profile`.  
    **Nota para Ubuntu e Fedora**: Modifique seu arquivo `~/.bashrc` ao invés de `~/.bash_profile`.

3. **Adicione `pyenv init` ao seu shell** para habilitar shims e completar automaticamente.
   Por favor certifique que `eval "$(pyenv init -)"` está presente no fim da configuração do shell já que o `PATH` é manipulado durante
   a inicialização

        $ echo 'eval "$(pyenv init -)"' >> ~/.bash_profile

    
    **Aviso Geral**: Existem algum sistemas onde a variável `BASH_ENV` é configurada apontando
    para `.bashrc`. Nesses sistemas você deveria quase que certamente colocar o que foi mencionado acima a linha
    `eval "$(pyenv init -)` dentro de `.bash_profile`, e **não** em `.bashrc`. Caso contrário você pode 
     ver um comportamento estranho, como o Otherwise you `pyenv` em looping infinito.
    Veja [#264](https://github.com/yyuu/pyenv/issues/264) para detalhes.

4. **Reinicie seu shell para que as mudanças no path tenham efeito.**
   Agora você pode começar a usar o pyenv.

        $ exec $SHELL

5. **Instale as versões do Python dentro de `$PYENV_ROOT/versions`.**
   Por exemplo, para baixar e instalar PYthon 2.7.8, execute:

        $ pyenv install 2.7.8

   **NOTA:** Se você precisar passar a opção de configuração para rodar, por favor use
   a variável de ambiente ```CONFIGURE_OPTS```.
   
   **NOTA:** Se você quiser usar um proxy para baixar, por favor use as variáveis de ambiente `http_proxy` e `https_proxy`.

   **NOTA:** Se você estiver com problemas em instalar alguma versão do Python, por favor visite a página wiki
   [Problemas comuns ao rodar](https://github.com/yyuu/pyenv/wiki/Common-build-problems)


#### Atualizando

Se você instalou pyenv usando as instruções acima, você pode atualizar sua instalação à qualquer momento usando
git.

Para atualizar para a última versão de desenvolvimento do pyenv, use `git pull`:

    $ cd ~/.pyenv
    $ git pull

Para atualizar para uma versão específica de liberação do pyenv, faça checkout da tag correspondente:

    $ cd ~/.pyenv
    $ git fetch
    $ git tag
    v0.1.0
    $ git checkout v0.1.0

### Desinstalando o pyenv

A simplicidade do pyenv torna fácil em temporariamente desabilitá-lo, ou desinstalá-lo do sistema.

1. Para **desabilitar** o pyenv gerenciando as versões do Python, simplesmente remova a linha 
  `pyenv init` da sua configuração de inicialização do shell. Isso removerá os diretórios shims do pyenv do PATH e futuros 
   comandos como `python` irão executar a versão do Python do sistema antes do pyenv.

  `pyenv` irá ainda estar acessível na linha de comando, mas suas aplicações Python não serão afetadas pela
   troca de versão.

2. Para **desinstalar** o pyenv por completo, execute o passo(1) e então remova o seu diretório raíz.
   Isso irá **apagar todas as versões Python** que estavam instaladas no diretório`` `pyenv root`/versions/ ``:

        rm -rf `pyenv root`

   Se você instalou pyenv usando um gerenciador de pacotes, como um passo final, execute a remoção do pacote pyenv.
   Segue exemplo pelo Homebrew:

        brew uninstall pyenv

## Referência de Comandos

### Homebrew no MAC OS X

Você também pode instalar o pyenv usando o [Homebrew](http://brew.sh)
package manager for Mac OS X.

    $ brew update
    $ brew install pyenv


Para atualizar o pyenv no futuro, utilize `upgrade` ao invés de `install`.

Depois da instalação, você precisará adicionar `eval "$(pyenv init -)"` para o seu perfil (como dito em ressalvas exibidas pelo 
Homebrew — para exibí-lo novamente, utilize `brew info pyenv`). Você só precisa adicioná-lo ao seu perfil uma vez.

Então, siga o resto dos passos pós-instalação em "Checkout básico do github" acima, começando com #4 ("reinicie seu shell para que as mudanças no path tenham efeito").

### Configurações Avançadas

Pule essa seção ao menos que você precise saber o que cada linha do seu shell está fazendo


`pyenv init` é o único comando que ultrapassa a linha de carregar comandos extras no seu shell.
 Vindos do rvm, alguns de vocês devem ser contrários à esta ideia. Segue o que o na verdade `pyenv init` faz:

1. **Configura os seus caminhos shims.** Este é o único requisito para o pyenv funcionar adequadamente. Você pode fazer isso antecipando:
   `~/.pyenv/shims` to your `$PATH`.

2. **Instalar completar de forma automática.** Isto é inteiramente opcional mas muito útil.
   Configurando a fonte como `~/.pyenv/completions/pyenv.bash` irá configurá-lo. 
   Há também um `~/.pyenv/completions/pyenv.zsh` para usuários Zsh.

3. **Refazer shims.** De tempos em tempos você precisa re-executar seus arquivos shim. Fazendo isso na inicialização certifica que tudo está atualizado. Você pode sempre executar `pyenv rehash` manualmente.

4. **Instalar o expedidor(dispatcher) sh.** Isso também é opcional, mas permite o pyenv e plugins mudar variáveis no seu shell atual, realizando linhas de comando como `pyenv shell` possíveis. O expedidor sh não faz nada anormal como sobrescrever `cd` ou hackear seu prompt do shell, mas se por alguma razão você precisar que o `pyenv` seja um script real ao invés de uma função shell, você pode seguramente pulá-lo.


Para ver exatamente o que ocorre por baixo dos panos, execute `pyenv init -`.


### Desinstalando Versões do Python

Conforme o tempo, você irá acumular versões do python em seu diretório
`~/.pyenv/versions`.

Para remover versões antigas do Python, o comando `pyenv uninstall` automatiza o processo de remoção.

Como forma alternativa, simplesmente utilize `rm -rf` no diretório da versão que você deseja remover. Você pode encontrar o diretório de uma versão particular do Python com o comando
 `pyenv prefix` , Exemplo: `pyenv prefix 2.6.8`.


----


## Comandos de referência

Veja [COMMANDS.md](COMMANDS.md).


----

## Variáveis de ambiente

Você pode modificar como o pyenv funciona com as seguintes configurações:

nome | padrão | descrição
-----|---------|------------
`PYENV_VERSION` | | Especifica a versão do python utilizada.<br>Veja também [`pyenv shell`](#pyenv-shell)
`PYENV_ROOT` | `~/.pyenv` | Define o diretório sobre o qual as versões do python e o shims ficarão.<br>Veja também `pyenv root`
`PYENV_DEBUG` | | Informação sobre os retornos de depuração.<br>Veja também: `pyenv --debug <subcommand>`
`PYENV_HOOK_PATH` | [_see wiki_][hooks] | Lista de caminhos, separados por vírgulas, procurado pelo pyenv hooks.
`PYENV_DIR` | `$PWD` | Diretório para iniciar a busca por arquivos`.python-version`.

## Desenvolvimento

O código fonte do pyenv está [hospedado no GitHub](https://github.com/yyuu/pyenv).  Ele é leve, modular, e fácil de entender, mesmo se você não for um hacker de linha de comando.

Os testes são executados usando [Bats](https://github.com/sstephenson/bats):

    $ bats test
    $ bats/test/<file>.bats

Por favor, fique a vontade para submeter sugestões e bugs em nossa [issue tracker](https://github.com/yyuu/pyenv/issues).

  [pyenv-virtualenv]: https://github.com/yyuu/pyenv-virtualenv#readme
  [hooks]: https://github.com/yyuu/pyenv/wiki/Authoring-plugins#pyenv-hooks
