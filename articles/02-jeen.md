둘째날 : Template Toolkit -> Xslate
===================================

글쓴이
======

@JEEN_LEE, 하니(?:님|놈)아빠, 킨들러리스트

## 개요

습관적으로 Catalyst 를 쓰고, 습관적으로 View 는 Template Toolkit 을 사용해왔습니다. [Text::Xslate](http://metacpan.org/module/Text::Xslate) 은 잔잔한 일에서 사용해왔습니다만, 이번 기회에 Catalyst View 단을 Xslate 로 사용해보고 TT2 에서 부딪혀왔던 부분들을 어떻게 Xslate 에서 사용하고 있는지, 또 Xslate 를 사용하면서 어떻게 바뀌었는지에 대해서 이야기를 해보고자 합니다.

#### 158x Faster than TT2

[Xslate 홈페이지](http://xslate.org/) 를 보면 벤치마크 결과 TT2 보다 158배가 빠르다는 얘기가 있습니다. 여타 템플릿 엔진과 비교했을 때에도 Xslate 는 독보적인 성능을 자랑한다고 합니다.

기본적으로 Catalyst App 의 보틀넥은 TT2 라는 것이 일반적인 관념이라고 할까요.

#### Cache?

TT2 에서 템플릿 컴파일을 해놓은 파일을 이용해서 Cache 를 하고 있습니다.

<pre class="brush: perl">
    'COMPILE_DIR' => ....,
    'COMPILE_EXT' => '.ttc',
</pre>

뭐 대략 이 두 항목을 추가해줌으로 Cache 를 활성화할 수 있습니다.

Xslate 에서 Cache 를 활성화하기 위해서는

<pre class="brush: perl">
    'View::Default' => {
        ...,
        'cache' => 1,
        'cache_dir' => ...,
    },
</pre>

로 지정합니다. 이걸로 더 빠른 템플릿이 가능하다는 것이지요.

#### TTerse

<pre class="brush: perl">
    'View::Default' => {
        'syntax' => 'TTerse',
        'module' => [ 'Text::Xslate::Bridge::TT2Like' ]
    },
</pre>

Xslate 를 사용할 때 어떤 Syntax 를 사용할 것인가 설정을 해줘야 합니다. 일단 TT2 를 사용하던 습관이 있으면 `TTerse` 를 사용하는 것이 가장 확실합니다. 거의 모든(100% 는 아닙니다) 사용법이 TT2 와 동일하니까요.

그외 `Kolon`, `Metakolon` 등의 Syntax 도 있습니다만 저는 학습비용에 대한 부담때문에 `TTerse` 만 주로 사용하고 있습니다. 

Syntax 에서 `TTerse` 를 지정한 것만으로 끝난 것은 아닙니다.
위처럼 module 에서 지정한 `Text::Xslate::Bridge::TT2Like` 이 있어야 TT2 의 Virtual Methods 를 사용할 수 있습니다.

<pre class="brush: perl">
    [% array.jon(", ") %]
    [% hash.keys() %]
</pre>

이와 같은 것들 말이죠.

#### XSS?

TT2 에서는 기본적으로 외부에서 받는 데이터(일례로 Query Parameter) 에 대해서는

<pre class="brush: perl">
    [% val | html %]
</pre>

위처럼 매번 필터를 붙여주곤 했습니다만, Xslate 에서는

<pre class="brush: perl">
    [% val %]
</pre>

자체로 `html_escape` 된 상태로 값이 찍히게 됩니다. 그렇기에 거꾸로 `html_escape` 되지 않은 상태의 값을 찍으려면 

<pre class="brush: perl">
    [% val | mark_raw %]
</pre>

라고 해줘야할 필요성은 있습니다.

자, 일부러 `mark_raw` 필터를 주지 않는 이상은 우리는 이제 XSS의 위협에서 자유로울 수 있습니다.

#### 덤프

템플릿 안에서 복잡한 형식을 갖춘 값이 있다고 했을 때, 템플릿안에서 그 값을 열어보고 싶을 때 주로 덤프를 하기 마련입니다. TT2 의 경우에는 그럴 경우에는

<pre class="brush: perl">
    [% USE dumper %]
    [% dumper.dump(data) %]
</pre>

로 값을 덤프해볼 수 있었습니다. Xslate 에서는 이보다 더 간결합니다.

<pre class="brush: perl">
    [% data | dump %]
</pre>

#### 명시적으로 () 를 붙인다?

TT2 에서 여지껏 큰 고난없이 사용해왔던 대표적인 구문입니다만

<pre class="brush: perl">
    [% FOREACH k IN c.req.parameters.keys.sort %]
    [% END %]
</pre>

Xslate 에서는 이게 안먹히더라는 거죠.

<pre class="brush: perl">
    [% FOREACH k IN c.req.parameters.keys().sort() %]
    [% END %]
</pre>

이렇게 명시적으로 모든 메소드끝에 () 를 붙여주었습니다.

#### MACRO

Catalyst 에서 MACRO 는 `PRE_PROCESS` 에 지정된 파일에서 정의하고 있었습니다. Xslate 에서는 `PRE_PROCESS` 비슷한 설정항목이 보이지 않아서 망설이다가 그냥 이렇게 처리했습니다.

<pre class="brush: perl">
    'View::Default' => {
        ....,
        header => [ 'common.tx', 'header.tx' ],
        ....,
    },
</pre>

위에 지정한 대로 MACRO 따위는 전부 `common.tx` 에 집어넣고 사용하고 있습니다.

#### expose_methods

처음에 PHP 의 Smarty 를 쓰다가 TT2 를 사용할 때 느꼈던 불편함 중 하나는 Smarty 처럼 __간단하게__ 필터함수를 만들 수 없었죠. `Template::Plugin::*` 류의 모듈을 만들어서 사용함으로써 비로소 원하던 바를 이룰 수 있었는데, Xslate 에서는 그게 훨씬 쉬워졌습니다.

<pre class="brush: perl">
    package MyApp::View::Default;
    ....
    extends 'Catalyst::View::Xslate';

    sub do_something {
        my ( $self, $c , @args ) = @_;

        return ...;
    }
</pre>

위처럼 Xslate 를 사용하는 View 에서 `do_something` 이라는 메소드를 만듭니다. do_something 에서는 Catalyst Context 마저도 받을 수 있어서 여타 값들과 함수들을 쉽게 참조할 수 있습니다.

자, 이렇게 `do_something` 을 정의했으면 View 의 설정에서

<pre class="brush: perl">
    'View::Default' => {
        'expose_methods' => {
            'something' => 'do_something',
        },
    },
</pre>

으로 설정하고, 템플릿 파일에서는

<pre class="brush: perl">
    [% val | something %]
</pre>

으로 이용할 수 있습니다.

#### Comma ?

의미있는 숫자값들을 표시할 때는 천단위마다 콤마를 넣는 게 일반적인 상식이죠. TT2 에서는

<pre class="brush: perl">
    [% USE Comma %]
    [% number_val | comma %]
</pre>

기본적으로 이렇게 사용할 수 있었습니다. 템플릿 안에서 `Template::Plugin::Comma` 를 불러와서 써줘야 했었죠.

그럼 Xslate 에서는... 위의 `expose_methods` 를 통해서 콤마 필터 역할을 하는 메소드를 만들어줘도 되겠습니다만,

<pre class="brush: perl">
    'View::None' => {
        ....,
        module  => [ 'Text::Xslate::Bridge::TT2Like',
                     'Number::Format' => [':subs'] ], 
    },
</pre>

이렇게 해서 [Number::Format](http://metacpan.org/module/Number::Format)  모듈을 이용합니다. Number::Format 에서 제공하는 메소드들을 이용할 수 있습니다.

<pre class="brush: perl">
    [% number_val | format_number %]
</pre>

이렇게 말이죠.


## Conclusion

사실 위에서 열거한 내용들은 현재 진행중인 프로젝트에서 사용하고 있는 방법들을 엮은 것입니다. 제가 고심한 부분도 있고, 제가 고민한 것을 미리 고민해서 해결해놓은 다양한 웹상의 문서들도 참고를 했습니다. 시간이 된다면 현재 진행중인 프로젝트에 같은 옵션을 주고  TT2 vs Xslate 벤치마크를 해보고 결과를 공개해보고 싶었는데, 아쉽습니다.
