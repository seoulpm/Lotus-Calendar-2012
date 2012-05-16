첫째날 : 펄을 이용한 그림 파일 긁어오기.
===========================================

글쓴이 : JellyPo

## 개요
이 글에서 다루는 기술적 수준은 eeyees 님의 2010년 12월 6일 기사 '[나만의 E-Book으로 따뜻한 크리스마스를!](http://advent.perl.kr/2010/2010-12-06.html)'(이하 나만의 이북) 에서 다룬 내용입니다. 개별 사이트에 적용하면서 제 필요에 맞게 개선했습니다.

이 기사에선 '나만의 이북'에서 그림 파일 긁어오기 기능을 좀 더 개선 및 다양한 사이트에 적용한 결과를 공개합니다.

그림 파일 긁어오기는 재미도 있을 뿐만 아니라, 여러 쓸모가 있습니다.
다만 그림파일이 있는 사이트 관리자는 싫어하겠죠. 그림을 긁어오는 행위 자체와 긁어온 그림을 공개하는 것이 법적으로(저작권, 청소년 보호법) 등에 위배되지 않는지는 꼭 미리 확인해보세요. 여기 공개 된 내용으로 뭔가 하더라도 이 기사의 글쓴이와 펄 커뮤니티에서 책임을 지지 않습니다.;

또한, 특정 사이트에 맞춰 작성된 코드이기에 범용성이 매우 떨어지고, 매번 사이트에 맞춰 코드 수정이 필요합니다.

글쓴이 JellyPo가 펄에 입문한지 5개월(시간 날 때 가끔 들여다보는 수준)임을 감안하고 봐 주세요. :D

## 개발 환경 및 전제조건
 1. 리눅스(우분투)에서 개발, 실행 하였습니다.
 1. 받아온 그림 파일명은 'img\_%04d.jpg'입니다. img\_0000.jpg 이런 식으로 저장이 되는거죠. 확장자는 받아온 다음 체크해서 변경합시다.
 1. 시행착오 과정도 같이 적었습니다. 바로 실행해보길 원하시는 분은 각 단락 마지막 부분 소스만 확인하시면 됩니다.
 1. 나만의 이북 내용을 한 번 읽고 보시길 권장합니다.
 1. 최종 소스의 타겟 사이트가 디씨인사이드가 아닌 알지롱닷넷입니다.

## 나만의 이북 만들기 개선
나만의 이북 만들기는 여러번 실행할 경우, 받아온 기존 그림 파일을 덮어 씌워버립니다. 파일을 저장할 때 사용하는 변수 $file\_num가 프로그램 실행할 때 0으로 고정 되어 있기 때문이죠.


그림 파일을 숫자 크기 순으로 정렬한 다음, 마지막 파일명을 가져오면 해결할 수 있을거 같네요. 그렇게 가져온 값에 +1 한 다음  $file\_num 변수에 넣겠습니다.

나만의 이북 만들기 중, get-images.pl 에서 $file\_num 선언한 부분을 아래와 같이 수정합니다.
그리고 get\_last\_num 함수를 추가하면 그림 파일 덮어 씌우는 문제는 해결할 수 있습니다.

### 리눅스 시스템 명령어를 사용한 숫자 정렬
<pre class="brush: perl">
my $file_num = 0;                 # 기존
my $file_num = get_last_num();    # 이렇게 수정해 주세요.
</pre>

아래는 리눅스 시스템 명령어를 사용한 get_last_num입니다.
<pre class="brush: perl">
# 이미 저장된 파일 중 숫자 끝자리에 +1 더한 값 반환함.
sub get_last_num {
    my @unsorted_imgs = `ls -1 ./ | sort -t _ -k 2 -n`;
    my $lastnum;

    $lastnum = $unsorted_imgs[-1];
    $lastnum =~ s|img_(.+)\....|$1|g;

    $lastnum =~ s|\n||g;
    return ++$lastnum;
}
</pre>

'ls -1 ./' 을 하면 아래와 같은 결과가 나옵니다.(이미 한 번 실행해서 이미지를 저장한 상태라고 했을 때 결과입니다.)

<pre>
    img_0000.jpg
    img_0001.jpg
    img_0002.jpg
    img_0003.jpg
    ...
    img_9999.jpg
    img_11111.jpg
</pre>

' | sort -t \_ -k 2 -n' 는 'ls -1'의 표준 출력을 넘겨 받아 img\_0000.jpg 중에 \_ (언더바)를 기준으로 나눈 문자열 img, 0000 중에 두번째인 0000을 기준으로 정렬하게 됩니다.

1. -t, --field-separator 필드 분리 구분을 무엇으로 하는지 정하는 옵션
1. -k 몇 번째 키를 기준으로 정렬할 것인지 지정
1. -n, --numeric-sort 자연수 정렬. 이 옵션이 없으면 img\_9999.jpg가 img\_11111.jpg 보다 나중에 정렬 됩니다. 아래처럼

<pre>
    img_0000.jpg
    img_0001.jpg
    ...
    img_1111.jpg
    img_11111.jpg
    img_1112.jpg
    ...
    img_9999.jpg
</pre>

앞에서 부터 비교해서 정렬하기 때문에 사람이 생각했을 때 숫자 정렬과 다른 결과가 나옵니다. 그래서 -n 옵션을 붙이죠.

### 순수 펄로 구현한 get\_last\_num
해당 소스의 구동에 대해 제게 묻지 말아주세요. a3ro 님 코드인데 왜 저렇게 되는지 이해를 못하고 있습니다. 그냥 되니까 갖다 쓰는 중. ^^;;
<pre class="brush: perl">
sub get_last_num {
my @a = glob("img_\*");
my $last = ( sort { $a <=> $b } map { m/(\d+)/ } @a )[-1];
return ++$last;
}
</pre>

성능은 놀랍게도 시스템 명령어 사용한 쪽이 빠릅니다.(시스템 명령어 사용시 100ms, 펄로 구현시 400ms)
시스템 명령어는 glob 과정 없이 모든 결과를 바로 정렬하기 때문인 것 같습니다. 다만 그림 파일 이외에 다른 파일이 섞여 있을 때 잘못된 결과값이 나올 수 있습니다.

저는 성능이 좀 더 나은 시스템 명령어를 사용한 코드를 사용했습니다.
다양한 시스템에서 실행될 수 있게 하려면 순수 펄로 구현된 코드를 사용하는게 좋겠죠?

### 순수 펄로 get_last_num 구현 방법 2 ...의 힌트
기사 마감 이후 제보 받은 내용에 따르면 다음과 같이 opendir을 할 경우 glob 을 안 쓰기 때문에 빠르게 파일을 얻을 수 있다고 합니다. 디렉토리 내 파일 목록을 얻을 수 있긴 있네요. opendir과 readdir을 이용한 get_last_num 함수의 실제 구현은 차후에 ...언젠가...
<pre class="brush: perl">
my $some_dir = "./";

opendir(my $dh, $some_dir) || die;
while(readdir $dh) {
        print "$some_dir$_\n";
}
closedir $dh;
</pre>


### 파일 확장자 변경
나만의 이북 만들기는 모든 그림파일을 .jpg로 저장합니다.

제대로 된 확장자를 찾아줍시다.

File::Type 모듈을 이용하면 파일 타입이 어떤 것인지 알려준다고 합니다. 이 모듈을 이용해  png, gif, bmp, jpg 순으로 확장자를 바꿔주는 함수를 만들었습니다.

    $ft->checktype_filename(파일명)

이렇게 파일명을 넘기면 해당 파일이 무엇인지 텍스트로 나옵니다(바로 출력하진 않아요.).
펄 파일을 넣어보니 application/x-perl 라고 나오네요.

jpg, bmp, png, gif 등은 image/jpeg 등으로 나옵니다. 결과값으로 확장자를 정해줄 수 있을거 같네요.

if unless 등으로 확장자를 하나씩 찾아줬습니다.

뭔가 더 좋은 방법이 없을까요..?


img\_0000.jpg 저장한 이후에 아래 코드를 추가한 후
<pre class="brush: perl">
my $final\_name = file\_type\_check($file);
rename $file, $final\_name;
</pre>

file\_type\_check 함수를 추가합니다.
<pre class="brush: perl">
#저장한 파일 타입(png, gif, bmp, jpg)를 확인해서 확장자 바꾼 파일명을 반환
sub file_type_check {
#   my $file = shift;
    my ($file) = @_;
    my $ext;
    my $ft = File::Type->new();

    $ext= $ft->checktype_filename($file);
    $ext=~ s|image/||g;

    if ( $ext =~ /gif/ ) {
        unless ( $file =~ /\.gif/ ) {
            $file =~ s|(img_\d+)\.jpg|$1\.gif|g;
        }
        return $file;
    } elsif ( $ext =~ /x-png/ ) {
        unless ( $file =~ /\.png/ ) {
            $file =~ s|(img_\d+)\.jpg|$1\.png|g;
        }
        return $file;
    } elsif ( $ext =~ /x-bmp/ ) {
        unless ( $file =~ /\.bmp/ ) {
            $file =~ s|(img_\d+)\.jpg|$1\.bmp|g;
        }
        return $file;
    } else {
        return $file;
    }
}
</pre>



### 이미지 링크 찾기 개선
기존 소스 동작은 다음과 같습니다.
 1. 게시판에 들어감
 1. 게시물 링크를 배열에 넣음
 1. 게시물을 하나씩 읽어들임
 1. 읽어들인 게시물에서 그림 태그 찾아서 저장
 1. 다음 게시물로.. 반복
 1. 게시물 배열이 끝나면 게시판 다음 페이지로

이와 같이 동작하는데 그림 파일이 없는 게시물에도 들어가기 때문에 처리속도가 늦습니다.
디씨인사이드는 그림 파일이 첨부된 게시물은 게시판 목록에서 아이콘으로 표시가 됩니다.

그림 파일 첨부된 게시물의 html tag
<pre>
    &lt;td align=left style="word-break:break-all;"&gt;&nbsp;  &lt;img src='http://wstatic.dcinside.com/gallery/skin/skin\_new/img\_icon.jpg' /&gt;      &lt;a href="/list.php?id=bicycle&no=771868&page=1&bbs="   &gt;[더질]이거 왜이러는지 아시는분 서군횽 및 지게미유저 소환!!&lt;/a&gt; &lt;/span&gt;&lt;font style=font-family:tahoma;font-size:7pt&gt;&lt;a href="/list.php?id=bicycle&no=771868&page=1&view\_comment=1" &gt;&lt;/a&gt; &lt;/font&gt; &lt;/td&gt;
</pre>

그림 파일 없는 게시물의 html tag
<pre>
    &lt;td align=left style="word-break:break-all;"&gt;&nbsp;  &lt;img src='http://wstatic.dcinside.com/gallery/skin/skin\_new/new\_head.gif' /&gt;      &lt;a href="/list.php?id=bicycle&no=771864&page=1&bbs="   &gt;횽들 타이어 세척은 주로 뭘로 하시나요~?&lt;/a&gt; &lt;/span&gt;&lt;font style=font-family:tahoma;font-size:7pt&gt;&lt;a href="/list.php?id=bicycle&no=771864&page=1&view\_comment=1" &gt;[1]&lt;/a&gt; &lt;/font&gt; &lt;/td&gt;
</pre>
그림 파일이 있는 게시물 링크만 얻을 수 있으면 빠른 그림 긁어오기가 될 것 같습니다. 정규표현식 한 번에 가능할 것 같네요. 아래 소스를 봐주세요.

#### 기존 소스
<pre class="brush: perl">
sub get_image_links {
    my $url = shift;
    my @links;
    my $response;
    eval { $response = $page_scrap->scrape( URI->new($url) ); };
    warn $@ if $@;

    for my $link ( @{ $response->{link} } ) {
        next unless $link =~ /bbs=$/;
        push @links, $link;
    }

    return @links;
}
</pre>



#### 개선한 소스, 첨부파일 있는 게시물만 들어감
디씨인사이트의 경우, 첨부파일이 있음을 알리는 img\_icon.jpg 이후에 게시물에 대한 링크가 있었습니다.
img\_icon.jpg가 있는 줄만 매치 시키면서, 게시물 링크인 .\*bbs= 를 캡쳐하는 정규 표현식입니다.
정규표현식을 처음 접하시는 분은 어려운 내용일 수 있습니다.

<pre class="brush: perl">
sub get\_image\_links {
    my ($url) = @\_;
    my @links;
    $mech->get($url);
    my $response = $mech->content;
    my @urls = $response =~ m/img\_icon\.jpg.\*&lt;a href=\"(.\*bbs=)\"/g;

    foreach my $link ( @urls ) {
        $link = 'http://gall.dcinside.com' . "$link";
        next unless $link =~ /bbs=$/;
        push @links, $link;
    }

    return \@links;
}
</pre>

    img_icon\.jpg.*<a href="(.*bbs=)"
핵심이 되는 부분입니다.

 1. img\_icon\.jpg : img\_icon.jpg 파일을 매치합니다. '.'은 아무 글자 하나를 표현하기 때문에 '\.'으로 적어야 글자 dot 를 매치합니다. ...사실 없어도 지금 같은 경우는 매치 됩니다.
 1. .\* : '.' 아무 글자가 '\*' 없거나 무한정 존재 '.\*' 아무 글자가 있던 없던 $\가 나오기 전까지 모두 매치. $\는 보통 줄의 끝, \n 입니다. 단, 뒤에 뭔가 더 적혀 있으면 해당 글자 매치 될 때까지 될겁니다.
 1. &lt;a href=\"(.\*bbs=)\" : &lt;a href=" 까지 매치한 다음, '.\*bbs=' bbs=가 나올 때까지 아무글자나 매치. 괄호로 둘러 싸인 부분은 $1 변수에 들어갑니다. 캡쳐 라고 하는거 같습니다. 표현식에서 괄호로 여러개 캡쳐하면 $1 $2 $3..에 들어갑니다. 위 소스와 같이 변수에 들어가도록 지정하면 해당 변수에도 들어가는 모양입니다. 배열에 쏙 들어갔네요.

img\_icon.jpg 들어간 줄을 찾아서 그 뒤에 있는 a href 주소만 배열에 넣는 코드입니다. 이러면 그림 파일 있는 게시물 링크를 얻을 수 있네요.

## 중복 파일 찾기
이렇게 파일을 모으다보니 중복 파일이 눈에 띄었습니다. DC 등을 보면 계속 같은 파일을 올리는 사람들이 있잖아요.
마침 비슷한 내용이 펄 크리스마스 달력에 있네요! yuni\_kim 님의 [MP3 파일 찾아서 정리하기](http://advent.perl.kr/2010/2010-12-23.html) 를 참조했습니다.
File::Find::Duplicates 를 사용해 중복 파일은 dup 디렉토리로 이동합니다.

나만의 이북 만들기랑 완전 별개 프로그램입니다.

File::Find::Duplicates 모듈은 디렉토리를 넘겨주면 해당 디렉토리의 파일 목록을 가져와서 중복 파일 목록을 만듭니다(배열의 배열 ; a랑 같은 파일 배열, b랑 같은 파일 배열..)

이렇게 만들어진 목록에서 첫 번째 파일만 남기고 나머지 파일은 중복이므로 dup 디렉토리로 옮기는 프로그램입니다.

### 구형 중복 파일 찾기
<pre class="brush: perl">
use strict;
use warnings;
use File::Find::Duplicates;
use File::Copy;
use Cwd;

my $wd = getcwd;
my @dupes = find_duplicate_files( $wd );
my $dupeset;
my @a;
my $tn;

if ( -e "$wd" . "/dup" ) {
        print "Found dup.\n";
} else {
        print "Not found dup. mkdir dup dir. \n";
        mkdir "dup", oct(0755);
}

foreach my $dupeset (@dupes) {
    @a= @{$dupeset->files}; # @a는 배열들을 담은 배열이다.
    shift(@a);  # @a에 담겨진 배열들의 첫번째 파일이름을 배열에서 밀어냄. 원본 하나는 남겨야 하기 때문.
    foreach (@a) {
            $tn = $_;
            $tn =~ s|(/.*/)(img_\d+....)|$1dup/$2|g;
            move ($_, $tn);
    }
}
</pre>

### 중복파일 검사 성능 개선
중복 파일까지 정리가 되고 나니 모든게 다 된 것 같았으나, 그림 모을 때마다 중복 검사를 실행해야 한다는 것과, 전체 그림 파일 수가 늘어날 수록 중복 검사 실행 시간은 점점 길어진 다는 점을 발견해, 원본 그림 파일과 새로 받은 파일명의 md5 비교를 하도록 했습니다. 원본 파일들 이름 자체를 MD5로 바꾸고요.

파일명을 그대로 두고 "파일명 - MD5"를 매치한 텍스트 파일이나 데이터 베이스가 있어도 되겠습니다만 ...10만개 파일이상이 되었을 때, 파일 목록을 배열에 넣고 검색하거나, 파일 grep 하는 것보다 if ( -e 파일명)이 더 낫지 않겠나 싶어서 일단 이렇게 만들었습니다.

프로그램 흐름을 다음과 같습니다.

 1. 현재 위치에 있는 img\_0000.jpg 의 md5값 구함
 1. 디렉토리 complete/ 아래에 md5로 된 파일명이 있는지 확인
  1. 있으면 중복 파일이니 dup/ 디렉토리 아래로 이동,
  1. 없으면 새로운 파일이니 complete/md5name 으로 저장

새로운 dup.pl
<pre class="brush: perl">
#!/usr/bin/perl
use strict;
use warnings;
use File::Find::Duplicates;
use File::Copy;
use Cwd;
use Digest::MD5;
use autodie;

my $pwd = getcwd;
my @img_file = `ls -1 img_*`;
my $count = 0;


open my $hashdb, ">>", 'hashdb.txt';
open my $dup_hash, ">>", 'dup_hash.txt';

foreach (@img_file) {
    my $work_file = $img_file[$count];
    chomp $work_file;

    # md5 파일명 구해오는 함수
    #    my $md5_name = &get_md5($work_file);  # 이 방식은 옛날 방식입니다. 아래처럼 하는 것이 낫다고 합니다.
    my $md5_name = get_md5($work_file);

    # md5 이름으로 complete 디렉토리에 해당 파일이 있는지 체크.
    # 있으면 dup 디렉토리로 이동하고
    # 없으면 이름 바꿔서 complete 디렉토리로 이동
    # 필요는 없으나 hashdb.txt에 내용 추가
    if ( -e "complete/$md5_name" ){
        print "$md5_name $work_file is exist.\n";
        move($work_file, "dup/$work_file");
        print $dup_hash "\n$work_file";

    } else {
        print "$md5_name $work_file is no exist, save file\n";
        move($work_file, "complete/$md5_name");
        print $hashdb "\n$md5_name";
    }
    $count++;
}
close $hashdb;
close $dup_hash;

sub get_md5 {
    # 인자로 받은 파일의 MD5 구해서 complete/md5 에 파일이 있는지 확인

    my $wf = shift; # 인자로 넘어온 $work_file 저장
    my $ext = $wf;
    $ext =~ s/img_\d+(\....)/$1/g;


    my $md5 = Digest::MD5->new;
    my $hashdb = 'hashdb.txt';
    open my $md5_fh, "<", $wf;
    $md5->addfile($md5_fh);
    my $digest = $md5->hexdigest;
    close $md5_fh;

    $digest .= $ext;
    return $digest;
}
</pre>
추가로, dup_hash 텍스트 파일을 만들게 되어 파일 중복 횟수를 구할 수 있게 되었습니다.

## 호게헌터
지금까지 나온 내용을 종합해 완성된 알지롱닷넷 호기심 게시판 짤방 모으기 프로그램입니다.

1. 시스템 명령어를 썼다는 점에 주의하세요!
1. Web::Scraper 안 쓰게 변경할 수 있습니다. ...만 적용은 못했네요.
1. 게시판 정렬 방식을 asc로 바꾸었기 때문에 1페이지가 최근 첫 페이지가 아닌, 가장 옛날 페이지임에 주의하세요.

<pre class="brush: perl">
#!/usr/bin/perl
use strict;
use warnings;
use URI;
use LWP::UserAgent;
use Web::Scraper;
use WWW::Mechanize;
use File::Type;

my $mech = WWW::Mechanize->new();
my $file_num = get_last_num();
my ( $start_page_num, $last_page_num ) = @ARGV;
$start_page_num ||= 1;
$last_page_num  ||= 3;

# bbs scraper
my $bbs_scrap = scraper {
    process 'img', 'imglink[]' => '@src';
};

for my $current_page_num ( $start_page_num .. $last_page_num ) {
    print "current page : $current_page_num" . "\n";
    my $g_name = sprintf(
        'http://rgr.kr/bbs/zboard.php?id=rgr201204&select_arrange=no&desc=asc&page=%s',
        $current_page_num,
    );
    my $links = get_image_links($g_name);

    # delete Notice :D
    if ( $current_page_num == 1 ) {
        shift( @{$links} ) for ( 1 .. 3 );
    }
    download($links);
}

sub get_image_links {
#   my $url = shift;
    my ($url) = @_;
    my @links;
    my $response;
    $mech->get($url);
    $response = $mech->content;
    my @urls = $response =~ m/a href=\"(.*)\".*image_up\.gif/g;


    foreach my $link ( @urls ) {
        $link = 'http://rgr.kr/bbs/' . "$link";
        next unless $link =~ /view\.php/;
        push @links, $link;
    }

    return \@links;
}

sub download {
#   my $links = shift;
    my ($links) = @_;

    for my $article_link ( @{$links} ) {
        my $response;
        eval { $response = $bbs_scrap->scrape( URI->new($article_link) ); };
        if ($@) {
            warn $@;
            next;
        }
        for my $img_link ( @{ $response->{imglink} } ) {
            if ( $img_link =~ m|rgr.kr/bbs/data/rgr201204/| ) {
                print $img_link . "\n";

                my $ua = LWP::UserAgent->new( agent =>
                'Mozilla/5.0'
                .' (Windows; U; Windows NT 6.1; en-US; rv:1.9.2b1)'
                .' Gecko/20091014 Firefox/3.6b1 GTB5'
        );
                my $res;
                eval { $res = $ua->get($img_link); };
                if ($@) {
                    warn $@;
                    next;
                }

                my $file = sprintf 'img_%04d.jpg', ++$file_num;
                open my $fh, ">", $file;

                binmode $fh;
                print $fh $res->content;
                close $fh;

        my $final_name = file_type_check($file);
        rename $file, $final_name;
           }
        }
    }
}

#이미 저장된 파일 중 숫자 끝자리에 +1 더한 값 반환함.
sub get_last_num {
    my @unsorted_imgs = `ls -1 dup/ | sort -t _ -k 2 -n`;
    my $lastnum;

    $lastnum = $unsorted_imgs[-1];
    $lastnum =~ s|img_(.+)\....|$1|g;

    $lastnum =~ s|\n||g;
    return ++$lastnum;
}


#저장한 파일 타입(png, gif, bmp, jpg)를 확인해서 확장자 바꾼 파일명을 반환
sub file_type_check {
#   my $file = shift;
    my ($file) = @_;
    my $ext;
    my $ft = File::Type->new();

    $ext= $ft->checktype_filename($file);
    $ext=~ s|image/||g;

    if ( $ext =~ /gif/ ) {
        unless ( $file =~ /\.gif/ ) {
            $file =~ s|(img_\d+)\.jpg|$1\.gif|g;
        }
        return $file;
    } elsif ( $ext =~ /x-png/ ) {
        unless ( $file =~ /\.png/ ) {
            $file =~ s|(img_\d+)\.jpg|$1\.png|g;
        }
        return $file;
    } elsif ( $ext =~ /x-bmp/ ) {
        unless ( $file =~ /\.bmp/ ) {
            $file =~ s|(img_\d+)\.jpg|$1\.bmp|g;
        }
        return $file;
    } else {
        return $file;
    }
}
</pre>

## 부록
### 김세영 갬블 다운로드
그냥 실행하기만 하면 최근 이미지를 다운로드 받습니다.
최근 회부터 첫회까지 받으려면 머니투데이 사이트 들어가셔서 '갬블' 마지막 페이지 숫자를 확인 후, 마지막 페이지 숫자가 40이라면

./gamble.pl 1 40

이렇게 실행하시면 됩니다.

중복 이미지가 있다면 중단 됩니다.

<pre class="brush: perl">
#!/usr/bin/perl
use strict;
use warnings;
use URI;
use LWP::UserAgent;
use Web::Scraper;
use WWW::Mechanize;
use File::Type;

my $mech = WWW::Mechanize->new();
my ( $start_page_num, $last_page_num ) = @ARGV;
$start_page_num ||= 1;
$last_page_num  ||= 3;

\# bbs scraper
my $bbs_scrap = scraper {
    process 'img', 'imglink[]' => '@src';
};

for my $current_page_num ( $start_page_num .. $last_page_num ) {
    print "current page : $current_page_num" . "\n";
    my $g_name = sprintf(
        'http://comic.mt.co.kr/comicDetail.htm?nComicSeq=35&nPage=%s',
        $current_page_num,
    );
    print "$g_name\n";
    my $links = &get_image_links($g_name);
    &download($links);
}

sub get_image_links {
    my ($url) = @_;
    my $response;
    my @links;
    $mech->get($url);
    $response = $mech->content;
    my @urls = $response =~ m|(/comicView\.htm\?cid=35&cno=\d+&nPage=\d+).*list_today|g;

    foreach my $link ( @urls ) {
        $link = "http://comic.mt.co.kr$link";
        push @links, $link;
        $link = "$link&cpg=2";
        push @links, $link;
     }

    return @links;
}

sub download {
    my ($links) = @_;
    my @img_urls;
    my $img_name;

    for my $article_link ( @{$links} ) {
        print "$article_link\n";
        $mech->get($article_link);
        my $response = $mech->content;
        @img_urls = $response =~ m|\"(http://comicmenu\.mt\.co\.kr/list/.*list_original.*)\" onMouseDown|g;

        foreach my $img_url ( @img_urls ) {
            print "$img_url\n";
            $img_name = $img_url;
            $img_name =~ s|.*/list/(.*\.jpg)|$1|g;
            unless ( -e $img_name ) {
                print "$img_name\n";
                $mech->get($img_url);
                open my $fh, ">", $img_name;
                binmode $fh;
                print $fh $mech->content;
                close $fh;
            } else {
                print "pass\n";
                exit;
            }
        }
    }
}
</pre>

### 김세영의 갬블시티 다운로드
갬블과 같은 방식입니다. 다운로드 페이지 수를 확인해서 인자로 넘기면 됩니다.
참고로 완결된 작품입니다. 코드 자체는 갬블이랑 똑같고 주소만 다르게 넣었습니다.

<pre class="brush: perl">
\#!/usr/bin/perl
use strict;
use warnings;

use URI;
use LWP::UserAgent;
use Web::Scraper;
use WWW::Mechanize;
use File::Type;

my $mech = WWW::Mechanize->new();
my ( $start_page_num, $last_page_num ) = @ARGV;
$start_page_num ||= 1;
$last_page_num  ||= 3;

\# bbs scraper
my $bbs_scrap = scraper {
    process 'img', 'imglink[]' => '@src';
};

for my $current_page_num ( $start_page_num .. $last_page_num ) {
    print "current page : $current_page_num" . "\n";
    my $g_name = sprintf(
        'http://comic.mt.co.kr/comicDetail.htm?nComicSeq=17&nPage=%s',
        $current_page_num,
    );
    print "$g_name\n";
    my $links = &get_image_links($g_name);
    &download($links);
}

sub get_image_links {
    my ($url) = @_;
    my $response;
    my @links;
    $mech->get($url);
    $response = $mech->content;
    my @urls = $response =~ m|(/comicView\.htm\?cid=17&cno=\d+&nPage=\d+).*list_today|g;

    foreach my $link ( @urls ) {
        $link = "http://comic.mt.co.kr$link";
        push @links, $link;
        $link = "$link&cpg=2";
        push @links, $link;
     }

    return @links;
}

sub download {
    my ($links) = @_;
    my @img_urls;
    my $img_name;

    for my $article_link ( @{$links} ) {
        print "$article_link\n";
        $mech->get($article_link);
        my $response = $mech->content;
        @img_urls = $response =~ m|\"(http://comicmenu\.mt\.co\.kr/list/.*list_original.*)\" onMouseDown|g;

        foreach my $img_url ( @img_urls ) {
            print "$img_url\n";
            $img_name = $img_url;
            $img_name =~ s|.*/list/(.*\.jpg)|$1|g;
            unless ( -e $img_name ) {
                print "$img_name\n";
                $mech->get($img_url);
                open my $fh, ">", $img_name;
                binmode $fh;
                print $fh $mech->content;
                close $fh;
            } else {
                print "pass\n";
                exit;
            }
        }
    }
}
</pre>

### 김세영의 갬블파티 다운로드
    foreach my $page_num ( 583 .. 585 ) { ... } 
이게 몇 회를 다운로드 받을지 정하는 부분입니다. 인자로 바꾼다는걸 깜빡 해서...
마감이 촉박하니 그냥 공개합니다.
하드코딩으로 숫자 바꿔서 실행하시면 되겠습니다.

    foreach my $page\_num ( 1 .. 585 ) { ... }
이렇게 바꾸시면 1화부터 585화까지 받게 됩니다.
도중에 링크 주소가 없으면 중단됩니다.

<pre class="brush: perl">
#!/usr/bin/perl
use strict;
use warnings;

use URI;
use WWW::Mechanize;
use File::Type;

my $mech = WWW::Mechanize->new();
my $g_name;
my $page;
my $num;

foreach my $page_num ( 583 .. 585 ) {
    $page = sprintf ("%03d", $page_num);
    $g_name = sprintf(
        'http://img.sportsseoul.com/comic/comic/party/%s/',
        $page,
    );

    foreach $num ( 1 .. 9 ) {
        my $img_url = sprintf(
            "$g_name%s",
            "$num.gif",
        );
        print "$img_url\n";
        download($img_url, $page, $num);
    }
}

sub download {
    my ($get_url, $page, $num) = @_;
    my $res;

    my $img_name = $page . '_' . $num . '.gif';
    if ( -e $img_name ) {
        print "$img_name exsist\n";
    } else {
        print "get $img_name\n";
        $mech->get($get_url);
        open my $fh, ">", $img_name;
        binmode $fh;
        print $fh $mech->content;
        close $fh;
#       sleep 1;
    }
}
</pre>

## 맺음말
이런 그림 파일 받아오는 프로그램은 그림 파일을 지정한 URL을 가져와 가공하는 문자열 편집 하면 되기에 펄 프로그래밍 실력이 많이 필요한 것은 아닙니다.

perl 코딩보단 사이트 분석하는데 시간이 더 많이 들었습니다. 자바 스크립트를 사용한 사이트의 경우 WWW::Mechanize에서 자바 스크립트를 지원 못하기 때문에 어려움이 있습니다. 코드 조금 작성하고 출력해서 결과 보고 하는 과정의 연속이었습니다.

갬블파티가 그런 예인데, 다행히 그림 파일 이름 명명 방식에 규칙이 있어서(하루에 1.gif ~ 9.gif) 그냥 그림 파일 주소를 요청해 다운 받았습니다.

이렇게 모은 파일을 perl 프레임워크를 이용해 실시간으로 공개하면 재미있는 짤방 사이트를 만들 수 있을거 같네요.

관세음보살