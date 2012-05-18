Title:    RPC 서버가 별건가요?
Package:  Seoul.pm
Category: perl
Category: Seoul.pm
Author:   keedi

저자
-----

[@keedi][twitter-keedi] - Seoul.pm 리더, Perl덕후,
[거침없이 배우는 펄][yes24-4433208]의 공동 역자, keedi.k _at_ gmail.com


시작하며
---------

전산학과에서는 보통 학부 3, 4학년 사이에 RPC와 관련해 한 번씩은 스쳐 지나갑니다.
혹자는 운영체제를 공부하면서 알게 되기도 하고, 또는 분산시스템을 공부하면서
알게 되기도 하고, 때로는 기억은 나지 않지만 과제를 하다가 알게 되기도 합니다.
RPC는 원격 프로시저 호출(Remote Procedure Call)로써 쉽게 말하면 이기종간의
플랫폼 사이에서 함수를 호출해서 서버의 자원을 사용해 연산한 후 그 결과를
전달 받는 것으로 전통적이며 대표적인 명세로는 CORBA나 SOAP, XML-RPC 등이 있습니다.
일반적인 전산학과의 학부 수업 과제라면 원격 프로시저 호출을 이용해서
계산기 RPC 서버와 클라이언트를 구현하곤 합니다.
Perl을 사용하면 이런 RPC 서버를 작성하는 것이 얼마나 간단한 일인지 살펴보겠습니다.



준비물
-------

필요한 모듈은 다음과 같습니다.

- [CPAN의 AnyEvent::MPRPC 모듈][cpan-anyevent-mprpc]
- [CPAN의 Moose 모듈][cpan-moose]
- [CPAN의 Getopt::Long::Descriptive 모듈][cpan-getopt-long-descriptive]

데비안 계열의 리눅스를 사용하고 있다면 다음 명령을 이용해서 모듈을 설치합니다.

    #!bash
    $ sudo apt-get install libanyevent-perl libmoose-perl libgetopt-long-descriptive-perl
    $ sudo cpan AnyEvent::MPRPC

직접 [CPAN][cpan]을 이용해서 설치한다면 다음 명령을 이용해서 모듈을 설치합니다.

    #!bash
    $ sudo cpan AnyEvent::MPRPC Moose Getopt::Long::Descriptive

사용자 계정으로 모듈을 설치하는 방법을 정확하게 알고 있거나
`perlbrew`를 이용해서 자신만의 Perl을 사용하고 있다면
다음 명령을 이용해서 모듈을 설치합니다.

    #!bash
    $ cpan AnyEvent::MPRPC Moose Getopt::Long::Descriptive



계산기 원격 함수
-----------------

RPC 프로그래밍에서 계산기 함수 구현은 프로그래밍 언어 교재의
첫 번째 예제인 *Hello World* 예제에 준할 정도로 늘상 선택하는
단골 예제입니다.

계산기라면 적어도 사칙 연산은 할 수 있어야 하겠죠?
덧셈, 뺄셈, 곱셈, 나눗셈을 지원해보도록 하죠.

- add()
- subtract()
- multiply()
- divide()

원격 함수는 이기종 시스템 간의 통신을 통해 데이터를 주고 받기
때문에 서로가 이해할 수 있는 프로토콜 규격이 필요합니다.
특히 함수의 인자로 전달하는 자료를 변환하는 과정을
마샬링(marshalling)이라고 합니다.
우리가 사용할 CPAN의 AnyEvent::MPRPC는 [MessagePack][messagepack-home]을
이용해서 마샬링 과정을 수행합니다.
MessagePack은 일본의 CPAN 저자인 [FURUHASHI Sadayuki][cpan-gfuji]가
작성한 라이브러리입니다.
바이너리 기반의 객체 직렬화 라이브러리로 구조화된 객체를 서로 다른 언어
사이에서 주고 받는 것이 가능하며, 매우 가볍고 빠른 것이 특징입니다.
C/C++, Perl은 물론 PHP, Python, Ruby, Java, Haskell, JavaScript 등의 언어를
지원하기 때문에 이런 RPC 시스템을 만들때 좋은 선택이라고 할 수 있습니다.



서버 모듈
----------

[AnyEvent::MPRPC][cpan-anyevent-mprpc] 모듈은 `mprpc_server` 함수를
호출하면 `AnyEvent::MPRPC::Server` 모듈의 생성자를 호출해서
`AnyEvent::MPRPC::Server` 객체를 생성합니다.
[Moose][cpan-moose]를 이용해서 호스트명과 포트명을 변경할 수 있도록
속성으로 지정하고, RPC 함수를 등록시키는 코드는 다음과 같습니다.

    package Abigail::Proxy::Calculator;
    
    use 5.010;
    use strict;
    use warnings;
    
    use Moose;
    use namespace::autoclean;
    
    use AnyEvent::MPRPC;
    
    has host => (
        is      => 'ro',
        isa     => 'Str',
        default => '127.0.0.1',
    );
    
    has port => (
        is      => 'ro',
        isa     => 'Num',
        default => 8080,
    );
    
    sub run {
        my $self = shift;
    
        my $cv = AE::cv;
    
        my $server = AnyEvent::MPRPC::Server->new(
            address   => $self->host,
            port      => $self->port,
        );
    
        $server->reg_cb(
            # Your RPC method goes here...
        );
    
        $cv->recv;
    }
    
    __PACKAGE__->meta->make_immutable;
    1;

네! 50줄이 채 되지 않는 간단한 코드입니다.
기본으로 로컬호스트와 8080 포트를 사용하도록 했습니다.
이제 사칙 연산을 할 수 있는 원격 메소드를 등록해야겠죠.
이 메소드 등록은 `reg_cb` 메소드를 이용해서 수행합니다.
`run` 메소드를 다시 보충해볼까요?

    sub run {
        my $self = shift;
    
        my $cv = AE::cv;
    
        my $server = AnyEvent::MPRPC::Server->new(
            address   => $self->host,
            port      => $self->port,
        );
    
        $server->reg_cb(
            add => sub {
                my ( $res_cv, $params ) = @_;
    
                my ( $num1, $num2 ) = @$params;
                my $ret = $num1 + $num2;
    
                $res_cv->result($ret);
            },
            subtract => sub {
                my ( $res_cv, $params ) = @_;
    
                my ( $num1, $num2 ) = @$params;
                my $ret = $num1 - $num2;
    
                $res_cv->result($ret);
            },
            multiply => sub {
                my ( $res_cv, $params ) = @_;
    
                my ( $num1, $num2 ) = @$params;
                my $ret = $num1 * $num2;
    
                $res_cv->result($ret);
            },
            divide => sub {
                my ( $res_cv, $params ) = @_;
    
                my ( $num1, $num2 ) = @$params;
                my $ret = $num1 / $num2;
    
                $res_cv->result($ret);
            },
        );
    
        $cv->recv;
    }

4칙 연산 기능을 모두 넣었습니다.
인자 개수라던가 받는 방식도 모두 동일하기 때문에
연산하는 한 줄의 코드만 달라질 뿐 나머지는 대동소이합니다.



서버 실행 스크립트
-------------------

모듈을 만들었으니 이 모듈을 사용해서 RPC 서버를
실행시키는 실행 스크립트가 필요하겠군요.
RPC 서버로서 필요한 모든 기능은 이미 서버 모듈에
다 들어있기 때문에 실행 스크립트 역시 간결합니다.

    #!/usr/bin/env perl
    
    use 5.010;
    use utf8;
    use strict;
    use warnings;
    use Getopt::Long::Descriptive;
    
    use Abigail::Proxy::Calculator;
    
    my $host = '127.0.0.1';
    my $port = 8080;
    
    my ( $opt, $usage ) = describe_options(
        "%c %o ...",
        [ 'host=s', "proxy server (default: $host)", { default => $host } ],
        [ 'port=i', "proxy port   (default: $port)", { default => $port } ],
        [],
        [ 'help|h', 'print usage message and exit' ],
    );
    
    print($usage->text), exit if $opt->help;
    
    my $proxy = Abigail::Proxy::Calculator->new(
        host => $opt->host,
        port => $opt->port,
    );
    $proxy->run;

끝입니다! 이미 만든 `Abigail::Proxy::Calculator` 모듈을 적재한 다음
해당 모듈의 객체를 생성하고 `run` 메소드를 실행시키면, 여러분의
RPC 서버는 동작하기 시작합니다. "부우웅~"

[CPAN의 Getopt::Long::Descriptive 모듈][cpan-getopt-long-descriptive]을
사용해서 호스트와 인자를 명령줄에서 받을 수 있도록 했기 때문에,
`Abigail::Proxy::Calculator` 모듈의 객체 생성시 참고할 수 있습니다.
실행하는 방법은 다음과 같습니다.

    $ ./calculator.pl

다른 포트 번호를 쓰고 싶다면 `--port` 옵션을 사용하면 됩니다.

    $ ./calculator.pl --port 9999

호스트 이름을 변경하고 싶다면 `--host` 옵션을 사용하세요.
궁금하다면 `--help` 옵션을 사용해보세요.
내가 언제 이런 메시지를 만들었지하고 놀라게 될 것입니다. :)



클라이언트 스크립트
--------------------

서버를 실행시켰다면 이제 제대로 동작하는지 검증해야겠죠.
클라이언트 스크립트를 만들어보죠.
클라이언트 스크립트는 [AnyEvent::MPRPC][cpan-anyevent-mprpc] 모듈의
 `mprpc_client` 함수를 호출해 `AnyEvent::MPRPC::Client` 모듈의
생성하고, 이 객체를 이용해서 RPC 서버 측으로 RPC 호출을 수행합니다.

    #!/usr/bin/env perl
    
    use 5.010;
    use utf8;
    use strict;
    use warnings;
    
    use AnyEvent::MPRPC;
    use Getopt::Long::Descriptive;
    
    my $host = '127.0.0.1';
    my $port = 8080;
    
    my ( $opt, $usage ) = describe_options(
        "%c %o <add|subtract|multiply|divide> <num1> <num2>",
        [ 'host=s', "proxy server (default: $host)", { default => $host } ],
        [ 'port=i', "proxy port   (default: $port)", { default => $port } ],
        [],
        [ 'help|h', 'print usage message and exit' ],
    );
    
    my ( $command, $num1, $num2 ) = @ARGV;
    
    print($usage->text), exit if     $opt->help;
    print($usage->text), exit unless $command;
    print($usage->text), exit unless defined $num1;
    print($usage->text), exit unless defined $num2;
    
    my $client = mprpc_client $host, $port;
    
    my $d;
    given ($command) {
        $d = $client->call( $command => [ $num1, $num2 ] ) when 'add';
        $d = $client->call( $command => [ $num1, $num2 ] ) when 'subtract';
        $d = $client->call( $command => [ $num1, $num2 ] ) when 'multiply';
        $d = $client->call( $command => [ $num1, $num2 ] ) when 'divide';
        default { die "unknown command: $command\n" }
    }
    
    say $d->recv;

클라이언트 프로그램은 RPC 서버의 호스트 주소와 포트를 옵션으로 전달 받습니다.
또한 필수적으로 실행할 RPC 메소드 이름과 두 개의 숫자를 입력해야 합니다.
호스트와 포트 번호를 동일하게 사용하고 덧셈을 수행하면 다음처럼 실행합니다.

    $ ./calculator-client.pl add 4 6
    10
    $ 

명령줄 옵션 처리를 했기 때문에 호스트와 포트 번호를 바꾸어서 실행시킬 수도 있겠죠.

    $ ./calculator-client.pl --host=192.168.0.13 subtract 4 6
    -2
    $ ./calculator-client.pl --port 9999 multiply 4 6
    24
    $ ./calculator-client.pl --host=192.168.0.13 --port 9999 divide 4 6
    0.666666666666667
    $ 


생각해볼 거리
--------------

계산기 RPC 서버 모듈, 서버 실행기, 클라이언트 실행기는
무척 간단해서 맥이 풀릴 정도입니다.
하지만 이렇게 간단하게 RPC 호출의 ABC를 알았다는 것은 중요합니다.
RPC 서버의 원리가 궁금하다면 다음 두 모듈의 소스를 보면 많은 도움이 될 것입니다.

- [CPAN의 AnyEvent::MPRPC 모듈][cpan-anyevent-mprpc]
- [CPAN의 Data::MessagePack 모듈][cpan-data-messagepack]

혹, 어떤 사람은 이렇게 이야기 할 수도 있겠죠.

> "뭐야, 이거 라이브러리를 사용해서 간단한 거잖아!"

네, 맞습니다.
학부 과정에서 RPC를 체험하기 위해 CORBA나 SOAP, XML-RPC를 사용할 때
역시 잘 만들어진 라이브러리를 사용했었죠.
차이점이 있다면 똑같이 라이브러리를 써도 지금은 너무 간단하다는 것이려나요?
Perl과 CPAN을 이용하면 많은 일들이 간결해집니다.
인생은 길지 않잖아요? :-)

지금 구현한 간단한 RPC 서버는 몇가지 문제가 있습니다.

- 서버나, 클라이언트 양측 모두 로그가 없으며,
- 서버의 경우 인자 처리를 하고 있지 않으며,
- 0으로 나누는 등의 예외 처리가 없으며,
- 오류가 있을때 반환할 상태나 코드 값이 따로 없죠.

`AnyEvent::MPRPC` 모듈 문서를 조금 더 꼼꼼히 살펴보고 필요한 API가 있다면
찾아서 쓰고, 할 수 있는 최대한 우리의 계산기 RPC 서버를 견고하게 만들어보세요.
물론, 새로운 기능을 추가하고 싶다면 얼마든지요!
간단한 암호화를 해서 누군가(여러분의 회사? :)가 살펴보는 것을 막는다던가,
네트워크나 운영체제에서 막아놓아(여러분의 회사가? :) 사용할 수 없는
기능을 사용하기 위해 RPC 서버를 구현하는 것은 무척 재미있을 것 같지 않나요?

RPC 서버 이전에 네트워크 통신 및 비동기 메시지 처리가 궁금하다면
다음 모듈을 살펴보세요.

- [CPAN의 AnyEvent 모듈][cpan-anyevent]

Perl의 객체지향 프로그래밍이 궁금하다면 다음 모듈을 살펴보세요.

- [CPAN의 Moose 모듈][cpan-moose]

명령줄 처리가 궁금하다면 다음 모듈을 살펴보세요.

- [CPAN의 Getopt::Long::Descriptive 모듈][cpan-getopt-long-descriptive]



정리하며
---------

RPC라는 것이 어떻게 보면 굉장히 당연하고 쉬워보이지만,
막상 구현하려고 하면 처음에는 헤매곤 하는 것 중 하나입니다.
RPC의 기본을 구현하기 위해 소개한 모듈들을 살펴보는 것 역시
많은 도움이 될테고, 잘 만들어진 이 모듈들을 이용해서 매일 같이
필요한 기능을 분산해서 처리하도록 만드는 것도 무척 유용할 것입니다.
이기종 간의 통신이 가능하도록 다른 언어를 사용해서 윈도우즈 머신에서
리눅스의 RPC 서버에 접속한다던가, 활용할 곳은 너무나도 많겠죠.
잊지마세요. Perl과 CPAN이라면 원격 프로시저 호출은 10분이면 된다는 것을요!

Enjoy Your Perl! ;-)



[cpan-anyevent-mprpc]:          https://metacpan.org/module/AnyEvent::MPRPC
[cpan-anyevent]:                https://metacpan.org/module/AnyEvent
[cpan-data-messagepack]:        https://metacpan.org/module/Data::MessagePack
[cpan-getopt-long-descriptive]: https://metacpan.org/module/Getopt::Long::Descriptive
[cpan-gfuji]:                   https://metacpan.org/author/GFUJI
[cpan-moose]:                   https://metacpan.org/module/Moose
[cpan]:                         http://www.cpan.org/
[messagepack-home]:             http://msgpack.org/
[twitter-keedi]:                http://twitter.com/#!/keedi
[yes24-4433208]:                http://www.yes24.com/24/goods/4433208
