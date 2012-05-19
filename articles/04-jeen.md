넷째날 : 나의 Catalyst 답사기 II
================================

글쓴이
======

@JEEN_LEE, 하니(?:님|놈)아빠, 킨들러리스트

## 개요

[2010 년 크리스마스 달력 기사 - 나의 Catalyst 답사기 II](http://advent.perl.kr/2010/2010-12-12.html) 에서 Catalyst 를 사용할 때 저는 이렇게 한다라고 끄적여 놓았던 적이 있습니다.

오늘은 그 연장선상에서 얘기를 진행해볼까 합니다. 어디까지나 제가 사용할 때 주로 하는 일들에 대해서 입니다.


### [Catalyst::Plugin::Static::Simple](http://metacpan.org/module/Catalyst::Plugin::Static::Simple) 은 사용하지 않는다

최근 버젼인 Catalyst 5.9 이후에서는 PSGI 를 기본적으로 지원하게 되었죠. 물론 PSGI 를 기본적으로 지원하기 이전부터 저는 항상 PSGI 붙여서 쓰고는 했지만...

우선 C::P::Static::Simple 를 사용한다면 Catalyst App 에서 직접 css/js/html 등의 정적파일들을 받도록하는데, 이 경우에서 Catalyst App 에서 부하가 발생하게 되죠. 그래서 대부분의 경우는 제일 앞단의 서버에서 받아서 처리하도록 하는 걸로 알고 있습니다. 

저같은 경우에는 [Plack::Middleware::Static](http://metacpan.org/module/Plack::Middleware::Static) 으로 PSGI 레이어에서 받아서 처리하도록 했습니다.

<pre class="brush: perl">
    # app.psgi
    use MyApp;
    use Plack::Builder;
    my $app = MyApp->apply_default_middlewares(MyApp->psgi_app);
    
    builder {
        enable "Static",
            path => qr{^/(?:css|js|img|favicon\.ico)},
            root => MyApp->path_to('root/static');
        $app;
    };
</pre>

그러니까 이렇게 함으로 `C::P::Static::Simple` 을 사용하지 않을 수 있습니다. :-) 물론 이 이후로는 어플리케이션 서버 기동시에는

<pre class="brush: bash">
    $ plackup -Ilib [app.psgi]
</pre>

로 띄워야 제대로 정적파일들을 받아볼 수 있겠죠.

### Catalyst App 설정파일은 .pl 형식을 사용한다

아마 Catalyst App 을 생성할 시에 기본으로 만들어지는 형식은 INI 포맷으로 알고 있습니다만, 저는 일부로 .pl 형식으로 바꾸고 시작합니다. 이전에는 YAML 이 익숙해서 YAML 을 사용해왔죠.

이유가 있다면, 결정적으로 .pl 형식은 

<pre class="brush: perl">
    {
        name => 'MyApp',
        envAA => $ENV{envAA} || 'envAA-Value',
    };
</pre>

이런식의 조건분기가 가능해집니다. 설정파일 자체에 코드가 난립하면 관리하기 힘들어질 수 있지만, 개발환경이 달라짐에 따라 바꿔줄 필요가 있는 다양한 설정항목들을 환경변수에서 정의하도록 하고 `myapp.pl` 은 매번 환경이 바뀔 때 마다 해당 설정항목을 일일이 바꿀 필요가 없이 다른 개발자들과 공유할 수 있습니다.

예를들어 다른 웹서비스의 API 에 접속하기 위한 토큰값이나 계정정보를 설정항목에 적어야할 필요가 있고, 이것들을 깜빡하고 같이 커밋해버리거나 한다면 여간 야단도 아니겠죠.

또한 설정항목안에 다른 방대한 설정정보가 필요한 경우가 있다면

<pre class="brush: perl">
    use YAML::XS;
    {
        ....,
        meta => YAML::XS::LoadFile('conf/meta.yaml'),
    };
</pre>

하나의 설정항목 안에 다시 다른파일을 읽어서 집어넣어서 쓰고 있기도 하구요.

### 각 컴포넌트의 설정정보를 한 곳으로...

예를들어 `MyApp` 이라는 것을 만들었다면.. MyApp.pm 에서 아래와 같은 항목들을 발견하실 겁니다.

<pre class="brush: perl">
    package MyApp;
    
    ....
    __PACKAGE__->config(
        name => 'MyApp',
        # Disable deprecated behavior needed by old applications
        disable_component_resolution_regex_fallback => 1,
        enable_catalyst_header => 1, # Send X-Catalyst header
    );
</pre>

그리고 `Catalyst::View::TT` 를 사용해서 Default View 를 만든다면 또 `View/Default.pm` 파일에서는 또 아래와 같은 항목들이 있겠죠.

<pre class="brush: perl">
    package MyApp::View::Default;
    use Moose;
    use namespace::autoclean;

    extends 'Catalyst::View::TT';

    __PACKAGE__->config(
        TEMPLATE_EXTENSION => '.tt',
        render_die => 1,
    );
</pre>

또 DB 접속을 위해서 Model 에 무엇인가를 만들었다면 `__PACKAGE__->config()` 에 무엇인가 쓰여져있거나 쓰고 있지는 않을까 생각합니다.

위에서 .pl 형식을 사용한다고 했는데, 저의 경우는 일단 위의 모든 설정항목들을 기본 설정파일로 묶어냅니다.

<pre class="brush: perl">
    {
        name => 'MyApp',
        disable_component_resolution_regex_fallback => 1,
        enable_catalyst_header => 1,
        'View::Default' => {
            TEMPLATE_EXTENSION => '.tt',
            render_die => 1,
        },
        'Model::DB' => {
            connect_info => {
                dsn      => $ENV{DB_DSN}      || "dbi:mysql:db:127.0.0.1",
                user     => $ENV{DB_USER}     || "user",
                password => $ENV{DB_PASSWORD} || "whatthehell",
                RaiseError        => 1,
                AutoCommit        => 1,
                mysql_enable_utf8 => 1,
                on_connect_do     => ["SET NAMES utf8"],
                quote_char        => q{`},
            },
        },
    };
</pre>

이렇게 말이죠. 왜냐면 이게 편하잖아요 :-)

### psgi 파일을 바꾼다

Catalyst App 을 생성하면 psgi 파일 명명규칙은 기본적으로 해당 이름을 따라가기 마련입니다. `MyApp::Web` 이라면 `myapp_web.psgi` 이 되겠죠. 일단 이 psgi 파일을 `app.psgi` 로 바꿉니다.

그러면 `plackup` 으로 기동시에 psgi 파일을 지정하지 않아도 됩니다;;; 기본적으로 `app.psgi` 파일이 있는지 확인하고 그걸 참조하도록 되어 있습니다.

<pre class="brush: bash">
    $ plackup -Ilib
</pre>

만으로도 어플리케이션 서버가 뜨게 됩니다.

## Conclusion

특별한 고민없이 카탈리스트를 사용해서 만들기 시작한 게 어언 3년이 넘어갑니다. 그런 중에 쌓였을 다양한 경험들이 있을 터인데, 새로운 시도보다 기존에 해온 것을 답습해옴으로인해서 뭔가 정체가 온 것은 아닌가 하는 생각이 드네요. 좀 더 다양하고 새로운 도전으로 기존의 방법들을 좀 더 개선할 수 있는 방법들은 없는지 꾸준히 노력해봐야 할 것 같습니다.

 
