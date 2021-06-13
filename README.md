[![Build Status](https://travis-ci.org/justddreamm/lab09.svg?branch=master)](https://travis-ci.org/justddreamm/lab09)

# Лабораторная работа №09 Лагов Сергей ИУ8-22

Забиваем переменные

```
$ export GITHUB_TOKEN=***
$ export GITHUB_USERNAME=justddreamm
$ export PACKAGE_MANAGER=apt
$ export GPG_PACKAGE_NAME=gpg
```

Ставим *xclip* для доступа к буферу обмена

```
$ $PACKAGE_MANAGER install xclip
$ alias gsed=sed
$ alias pbcopy='xclip -selection clipboard'
$ alias pbpaste='xclip -selection clipboard -o'
```

Далее ставим пакет *github-release*, с которым пришлось изрядно повозиться. Сначала пришлось догнать, как ставить сам *go*, чтобы через него можно было скачать вышеупомянутый пакет, но приключения с ним ещё не закончились. В будущем мне придётся этот пакет использовать, и на команду *github-release --version* система будет мне выдавать сообщение, что *command not found*. Благо, мне подкинули статью-туториал, в котором это проблема решается. Необходимо было немного отредактировать файл ~/.profile дописать туда некоторые пути и переменные, после чего запустить этот скрипт, чтобы система выполнила вписанные команды

```
$ cd ${GITHUB_USERNAME}/workspace
$ pushd .
$ source scripts/activate
$ go get github.com/aktau/github-release
```

Ставим *GPG* и генерим секрктный ключ. Создаём переменные с публичным и секретным ключами, затем копируем его в буфер и вводим его на наш *github*

```
$ $PACKAGE_MANAGER install ${GPG_PACKAGE_NAME}
$ gpg --list-secret-keys --keyid-format LONG
$ gpg --full-generate-key
$ gpg --list-secret-keys --keyid-format LONG
$ gpg -K ${GITHUB_USERNAME}
$ GPG_KEY_ID=$(gpg --list-secret-keys --keyid-format LONG | grep ssb | tail -1 | awk '{print $2}' | awk -F'/' '{print $2}')
$ GPG_SEC_KEY_ID=$(gpg --list-secret-keys --keyid-format LONG | grep sec | tail -1 | awk '{print $2}' | awk -F'/' '{print $2}')
$ gpg --armor --export ${GPG_KEY_ID} | pbcopy
$ pbpaste
$ open https://github.com/settings/keys
$ git config user.signingkey ${GPG_SEC_KEY_ID}
$ git config gpg.program gpg
```

Скрипт для добавления заметок к тегу

```
$ test -r ~/.bash_profile && echo 'export GPG_TTY=$(tty)' >> ~/.bash_profile
$ echo 'export GPG_TTY=$(tty)' >> ~/.profile
```

Логинимся в *Travis CI*

```
$ travis login --github-token (token)
$ travis enable
```

Верифицируем тэг с месседжом и пушим

```
$ git tag -s v0.1.0.0
$ git tag -v v0.1.0.0
$ git push origin main --tags
```

Далее происхоят те вещи, о которых я написал выше. Система не видит *github-release* - пришлось найти решение

```
$ github-release --version
$ github-release info -u ${GITHUB_USERNAME} -r lab09
$ github-release release \
    --user ${GITHUB_USERNAME} \
    --repo lab09 \
    --tag v0.1.0.0 \
    --name "libprint" \
    --description "my first release"
```

Указываем необходимые значения

```
$ export PACKAGE_OS=`uname -s` PACKAGE_ARCH=`uname -m`
$ export PACKAGE_FILENAME=print-${PACKAGE_OS}-${PACKAGE_ARCH}.tar.gz
$ github-release upload \
    --user ${GITHUB_USERNAME} \
    --repo lab09 \
    --tag v0.1.0.0 \
    --name "${PACKAGE_FILENAME}" \
    --file _build/*.tar.gz
```

Скачали и распаковали наш релиз

```
$ github-release info -u ${GITHUB_USERNAME} -r lab09
$ wget https://github.com/${GITHUB_USERNAME}/lab09/releases/download/v0.1.0.0/${PACKAGE_FILENAME}
$ tar -ztf ${PACKAGE_FILENAME}
```
