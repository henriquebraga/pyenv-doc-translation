# Comandos de referência

Assim como o `git`, o `pyenv` utiliza subcomandos que são baseados no primeiro argumento.

Os subcomandos mais comuns são:

* [`pyenv commands`](#pyenv-commands)
* [`pyenv local`](#pyenv-local)
* [`pyenv global`](#pyenv-global)
* [`pyenv shell`](#pyenv-shell)
* [`pyenv install`](#pyenv-install)
* [`pyenv uninstall`](#pyenv-uninstall)
* [`pyenv rehash`](#pyenv-rehash)
* [`pyenv version`](#pyenv-version)
* [`pyenv versions`](#pyenv-versions)
* [`pyenv which`](#pyenv-which)
* [`pyenv whence`](#pyenv-whence)


## `pyenv commands`

Listagem de todos os comandos pyenv disponíveis.

## `pyenv local`

Define a versão do Python que rodará no local específico da aplicação, escrevendo o nome da versão como um arquivo `.python-version` no diretório atual. Esta versão sobrescreverá a versão global, e poderá substituir qualquer variável ambiente `PYENV_VERSION` configurada por padrão ou com o comando `pyenv shell`.

    $ pyenv local 2.7.6

Quando executado sem um número de versão, o `pyenv local` utiliza a versão atualmente configurada para olocal. Você também pode desabilitar a versão local:

    $ pyenv local --unset

Versões antigas do pyenv armazenavam especificações da versão local em um arquivo chamado `.pyenv-version`. Para ter compatibilidade com versões anteriores, o pyenv lê  uma versão local especificada em um arquivo `.pyenv-version`, mas um arquivo`.python-version` no mesmo diretório terá precedência.


### `pyenv local` (avançado)

Você pode especificar múltiplas versões Python para um mesmo local de uma única vez.

Imagine que você tenha duas versões do Python, a 2.7.6 e a 3.3.3. Se você preferir a 2.7.6 em relação a 3.3.3 por exemplo:

    $ pyenv local 2.7.6 3.3.3
    $ pyenv versions
      system
    * 2.7.6 (set by /Users/yyuu/path/to/project/.python-version)
    * 3.3.3 (set by /Users/yyuu/path/to/project/.python-version)
    $ python --version
    Python 2.7.6
    $ python2.7 --version
    Python 2.7.6
    $ python3.3 --version
    Python 3.3.3

Ou, se você preferir a 3.3.3 em relação a 2.7.6:

    $ pyenv local 3.3.3 2.7.6
    $ pyenv versions
      system
    * 2.7.6 (set by /Users/yyuu/path/to/project/.python-version)
    * 3.3.3 (set by /Users/yyuu/path/to/project/.python-version)
      venv27
    $ python --version
    Python 3.3.3
    $ python2.7 --version
    Python 2.7.6
    $ python3.3 --version
    Python 3.3.3


## `pyenv global`

Define a versão global do Python para ser utilizada em todos os shells, escrevendo o nome da versão no arquivo `~/.pyenv/version`. Esta versão pode ser sobrescrita para uma aplicação específica com `.python-version`, ou defininda na variável ambiente `PYENV_VERSION`.

    $ pyenv global 2.7.6

A nome da versão especial `system` diz para o pyenv utilizar o sistema Python (detectado por sua busca no `$PATH`).

Quando executado sem um número de versão, o `pyenv global` adotará a configuração de versão global atual.

### `pyenv global` (avançado)

Você pode especificar múltiplas versões para uma versão global do Python, de uma única vez.

Imagine que você tenhas as seguintes versões do Python, a 2.7.6 e a 3.3.3. Se você preferir a 2.7.6 em relação a 3.3.3, por exemplo:

    $ pyenv global 2.7.6 3.3.3
    $ pyenv versions
      system
    * 2.7.6 (set by /Users/yyuu/.pyenv/version)
    * 3.3.3 (set by /Users/yyuu/.pyenv/version)
    $ python --version
    Python 2.7.6
    $ python2.7 --version
    Python 2.7.6
    $ python3.3 --version
    Python 3.3.3

Ou, se você preferir a 3.3.3 em relação a 2.7.6:

    $ pyenv global 3.3.3 2.7.6
    $ pyenv versions
      system
    * 2.7.6 (set by /Users/yyuu/.pyenv/version)
    * 3.3.3 (set by /Users/yyuu/.pyenv/version)
      venv27
    $ python --version
    Python 3.3.3
    $ python2.7 --version
    Python 2.7.6
    $ python3.3 --version
    Python 3.3.3


## `pyenv shell`

Define uma versão específica para o shell do Python definido na configuração da variável de ambiente `PYENV_VERSION`. Esta versão subtituirá as versões específicas do aplicativo e a versão global.

    $ pyenv shell pypy-2.2.1

Quando for executado sem um número de versão, o `pyenv shell` utilizará o valor da versão atual da `PYENV_VERSION`. Você também pode desabilitar a versão do SHELL:

    $ pyenv shell --unset

Observe que você precisará da integração do shell dos pyenvs ativados (passo 3 das instruções de instalação) para utilizar este comando. Se você preferir não utilizar esta integração, você pode simplesmente definir o `PYENV_VERSION` assim:

    $ export PYENV_VERSION=pypy-2.2.1

### `pyenv shell` (avançado)

Você pode definir múltiplas versões do `PYENV_VERSION`, ao mesmo tempo.

Imagine que você tem duas versões do python, a 2.7.6 e a 3.3.3. Se você preferir a 2.7.6 em relação a 3.3.3, faça:

    $ pyenv shell 2.7.6 3.3.3
    $ pyenv versions
      system
    * 2.7.6 (set by PYENV_VERSION environment variable)
    * 3.3.3 (set by PYENV_VERSION environment variable)
    $ python --version
    Python 2.7.6
    $ python2.7 --version
    Python 2.7.6
    $ python3.3 --version
    Python 3.3.3

Ou, se você preferir a 3.3.3 em relação a 2.7.6:

    $ pyenv shell 3.3.3 2.7.6
    $ pyenv versions
      system
    * 2.7.6 (set by PYENV_VERSION environment variable)
    * 3.3.3 (set by PYENV_VERSION environment variable)
      venv27
    $ python --version
    Python 3.3.3
    $ python2.7 --version
    Python 2.7.6
    $ python3.3 --version
    Python 3.3.3


## `pyenv install`

Instala uma versão do Python (using [`python-build`](https://github.com/yyuu/pyenv/tree/master/plugins/python-build)).

    Uso: pyenv install [-f] [-kvp] <versão>
         pyenv install [-f] [-kvp] <arquivo-de-definição>
         pyenv install -l|--list

      -l/--list             Lista todas as versões disponíveis
      -f/--force            Instala a versão mesmo que ela já esteja instalada
      -s/--skip-existing    Ignora a instalação se aquela versão já estiver instalada

      Opções do python-build:

      -k/--keep        Manter a árvores do código em $PYENV_BUILD_ROOT após a instalação
                       (por padrão em $PYENV_ROOT/sources)
      -v/--verbose     Modo detalhado: mostra o status da compilação para a saída padrão (stdout)
      -p/--patch       Aplica um patch do stdin antes da compilação
      -g/--debug       Cria uma versão de depuração

Para listar todas as versões disponíveis do Python, incluindo o Anaconda, Jython, pypy, e o stackless, utilize:

    $ pyenv install --list

Em seguida instale as versões desejadas:

    $ pyenv install 2.7.6
    $ pyenv install 2.6.8
    $ pyenv versions
      system
      2.6.8
    * 2.7.6 (set by /home/yyuu/.pyenv/version)

## `pyenv uninstall`

Remove uma versão específica do Python.

    Uso: pyenv uninstall [-f|--force] <version>

       -f  Tenta remover a versão especificada sem solicitar confirmação do usuário.
           Se a versão não existir, não será exibida mensagem de erro.


## `pyenv rehash`

Instala o shims para todas as versões de binários do Python encontrados pelo pyenv (por exemplo,
`~/.pyenv/versions/*/bin/*`). Execute este comando depois de instalar uma nova versão do Python, ou instalar um gerenciador de pacotes que ofereça binários.

    $ pyenv rehash


## `pyenv version`

Mostra a versão atual (ativa) do Python, junto com as informações já definidas.

    $ pyenv version
    2.7.6 (set by /home/yyuu/.pyenv/version)


## `pyenv versions`

Lista todas as versões conhecidas do Python, e mostra um asterisco junto a versão ativa.

    $ pyenv versions
      2.5.6
      2.6.8
    * 2.7.6 (set by /home/yyuu/.pyenv/version)
      3.3.3
      jython-2.5.3
      pypy-2.2.1


## `pyenv which`

Mostra o caminho completo para o executável que pyenv irá chamar quando você digitar um comando específico.

    $ pyenv which python3.3
    /home/yyuu/.pyenv/versions/3.3.3/bin/python3.3


## `pyenv whence`

Lista todas as versões instaladas do Python com o comando específico.

    $ pyenv whence 2to3
    2.6.8
    2.7.6
    3.3.3

