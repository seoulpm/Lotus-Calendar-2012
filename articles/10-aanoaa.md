title: 열째날 : bootstrap.pl - Easily jump into a project and start contributing
nav_name: 10

bootstrap.pl - Easily jump into a project and start contributing
================================================================

글쓴이
------
[@aanoaa](https://twitter.com/aanoaa)

# SYNOPSIS #

    $ script/bootstrap.pl

# DESCRIPTION #

프로젝트에 쉽게 참여하고 참여시키기 위해 사용하는 방법입니다.

1. `README.md` 강화
2. 모듈의존성은 [Carton][Carton]으로 해결
3. `script/bootstrap.pl`의 사용

## README.md ##

- 어떻게 local 환경에서 초기화 하고 실행 시킬수 있는지 자세하게 기록

        $ ./script/bootstrap.pl         # 모듈이 ./local/ 디렉토리에 설치됨
        $ script/bootstrap.pl --db-init # database 초기화가 필요하다면..
        $ ./run
    
        (위에 처럼 동작하려면, `db/schema.sql` 이랑, `db/data.sql` 그리고 `run` 파일이 필요합니다.)

## Carton ##

- perl 의 [Bundler][Bundler]입니다.

## bootstrap.pl ##

[ruby patterns from github codebase][ruby-patterns-from-githubs-codebase]을
보고 [github/github]을 뒤져서 [bootstrap-script][bootstrap-script]을
찾아서 [perl 버전][bootstrap.pl]으로 변경 했습니다.

1. [Carton][Carton]이 있는지 체크
2. `Makefile.PL` 의 md5 checksum 을 비교해서 변경 사항을 추적
3. `carton` 명령어를 이용해서 의존성이 있는 모듈을 설치

# RUN #

`README` 에서 실행방법을 알아보고, [Carton][Carton] 을 이용하는
`bootstrap.pl`을 사용해서 모듈의존성을 해결한 담에 실행합니다.

# SEE ALSO #

- [ruby patterns from github codebase][ruby-patterns-from-githubs-codebase]

[ruby-patterns-from-githubs-codebase]: https://speakerdeck.com/u/holman/p/ruby-patterns-from-githubs-codebase
[Carton]: http://search.cpan.org/~miyagawa/carton-v0.9.4/lib/Carton.pod
[Bundler]: http://gembundler.com/
[github/github]: https://github.com/github
[bootstrap-script]: https://github.com/github/github-services/blob/master/script/bootstrap
[bootstrap.pl]: https://gist.github.com/2761233
