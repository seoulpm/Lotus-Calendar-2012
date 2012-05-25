Title:    사용자 locale에 맞춰 자동으로 인코딩,디코딩용 캐릭터셋 지정하기

저자
-----

[@gypark][twitter-gypark] - [홈페이지](http://gypark.pe.kr/wiki/Perl)

시작하며
---------

Console에서 수행되는 Perl 프로그램을 만들 때, 입출력 데이타에 한글 또는 다른 ASCII범위를
벗어나는 글자가 있다면, 이를 적절하게 인코딩, 디코딩해야 할 것입니다.
문제는 캐릭터셋으로 UTF-8을 쓰는 리눅스 쉘과, cp949를 쓰는 윈도우 명령 프롬프트 창에서
같이 쓸 수 있는 코드를 작성하려면 현재 사용 중인 캐릭터셋이 무엇인지를 먼저 알아내고
그에 맞춰서 `encode()`와 `decode()`의 인자를 조절해주어야 한다는 것입니다.
이 글에서는 별도의 수고 없이, 현재 프로그램이 실행되고 있는 시스템 환경에 따라서
적절하게 인코딩과 디코딩을 할 수 있는 방법을 설명합니다.



기존에 제가 쓰던 방법
-------

아예 문자열 데이타를 처음부터 끝까지 바이트 스트림으로 처리한다면, 즉 인코드나 디코드를 하지
않고 처리한다면 상관이 없겠습니다만, 정규표현식이나 각종 스트링 처리 함수(`length()`,
`substr()` 등)를 편하게 쓰려면 아무래도 디코드를 하게 됩니다.

특히나, 입력받은 데이타를 일일이 디코드하거나, 데이타를 출력하기 전에 일일이
인코드해 주는 것은 매우 번거롭기 때문에, `STDIN`, `STDOUT` 등 표준입출력 파일핸들에
`binmode`를 사용하여 인코딩 레이어를 추가해주곤 합니다.

저는 일단 `$^O` 특수 변수에 담긴 운영체제 정보를 읽어서 윈도우인지 아닌지를 알아내고,
윈도우가 아닐 경우는 다시 `LANG` 환경 변수를 읽어서 캐릭터셋 정도를 알아내도록 하였습니다.
(더 좋은 방법이 있다면 가르쳐 주시면 감사하겠습니다)

<pre class="brush: perl">
	# 윈도우라면 cp949를 사용한다
	if ( $^O =~ /MSWin/ ) {
		binmode STDIN,  ":encoding(cp949)";
		binmode STDOUT, ":encoding(cp949)";
		binmode STDERR, ":encoding(cp949)";
	}
	# 윈도우가 아니고 $LANG 변수에 utf-8이 포함되어 있는 경우
	elsif ( $ENV{LANG} and $ENV{LANG} =~ /utf-?8/i ) {
		binmode STDIN,  ":utf8";
		binmode STDOUT, ":utf8";
		binmode STDERR, ":utf8";
	}
	# 그렇지 않으면 euc-kr
	else {
		...
	}
</pre>

여기서 문제가 여러 가지가 있는데, 사실 저는 윈도우 아니면 리눅스만 쓰기 때문에, 맥에서는
어떻게 되는지 모릅니다. 또한 `$LANG` 변수에 utf-8이나 euc-kr이 아닌 다른 값이 들어가
있다면 이것을 제대로 처리할 수 없습니다.



Encode::Locale 모듈
-------

[CPAN의 Encode::Locale 모듈][cpan-encode-locale]을 사용하면, 프로그램이 실행되고 있는
시스템의 로케일 정보를 이용하여, 자동으로 캐릭터셋을 지정해 줍니다. 그 정보를
이용하면 앞의 코드와 같이 번거로운 분기문을 쓰지 않고도 적절한 인코딩 레이어를
지정해줄 수 있습니다.

Encode::Locale 모듈은 코어 모듈이 아니기 때문에 추가로 설치해주어야 합니다.

직접 [CPAN][cpan]을 이용해서 설치한다면 다음 명령을 이용해서 모듈을 설치합니다.

    #!bash
    $ sudo cpan Encode::Locale

사용자 계정으로 모듈을 설치하는 방법을 정확하게 알고 있거나
`perlbrew`를 이용해서 자신만의 Perl을 사용하고 있다면
다음 명령을 이용해서 모듈을 설치합니다.

    #!bash
    $ cpan Encode::Locale

윈도우에서 Strawberry Perl을 사용할 경우 cpan을 이용할 수 있습니다.

    C:> cpan Encode::Locale

모듈을 사용하는 예제는, 아주 간단합니다.

<pre class="brush: perl">
	use Encode;
	use Encode::Locale;

	binmode STDIN,  ":encoding(console_in)";
	binmode STDOUT, ":encoding(console_out)";
	binmode STDERR, ":encoding(console_out)";
</pre>

Encode 모듈에서 쓸 수 있는 `console_in`, `console_out` 두 개의 별칭이 생겼습니다.
이것은 시스템 로케일에 맞춰 자동으로 적절한 값으로 적용됩니다.


간단한 예제
---------

<pre class="brush: perl">
	#!/usr/bin/env perl
	# 이 스크립트는 UTF-8로 인코딩하여 저장하여야 함
	use strict;
	use warnings;

	use Encode;
	use Encode::Locale;

	use utf8;       # 문자열 상수를 decode된 스트링으로 처리

	binmode STDIN,  ":encoding(console_in)";
	binmode STDOUT, ":encoding(console_out)";
	binmode STDERR, ":encoding(console_out)";

	print "이름을 입력하세요:";     # 디코드된 스트링을 출력
	my $input = <>;                 # 입력은 인코딩된 바이트 스트림이지만
									# $input 에는 자동으로 디코드되어 들어간다
	chomp $input;
	print "안녕하세요, $input 님\n";
</pre>

위 코드를 다음 세 가지 환경에서 실행해보겠습니다.
- $LANG 변수에 UTF-8이 명시된 리눅스 쉘
- $LANG 변수에 EUC-KR이 명시된 리눅스 쉘
- 한글윈도우의 명령 프롬프트 창

(그림: 10-gypark-locale.png)

위 스크린샷을 보면 동일한 코드가 서로 다른 캐릭터셋을 사용하는 환경에서
이상없이 실행되는 것을 볼 수 있습니다.

파일 핸들에 인코딩 레이어를 추가하는 것 뿐 아니라, 일반적인 인코드와 디코드를 할 때도
역시 캐릭터셋에 신경쓰지 않고 할 수 있습니다. 캐릭터셋 이름을 적어주는 자리에
`locale`이라고 적어주면 그만입니다.

<pre class="brush: perl">
	# 바이트 스트림을 디코드해서 스트링으로 변환
	$string = decode(locale => $bytes);
	# 스트링을 인코드해서 바이트 스트림으로 변환
	$bytes = encode(locale => $string);
</pre>


이제는 프로그램을 어느 환경에서 실행할 것인지 고민하지 않고 편안하게 포터블한 코드를 만들 수 있겠습니다 :-)



참고
---------


- [Perl/한글](http://gypark.pe.kr/wiki/Perl/%ED%95%9C%EA%B8%80) - 네이버 펄 까페에 올렸던 "펄과 한글" 기초강좌글 합본입니다.
- [CPAN의 Encode::Locale 모듈][cpan-encode-locale]

