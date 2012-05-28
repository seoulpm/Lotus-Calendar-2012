펄로 알아보는 성인들의 자비지수
=======================

글쓴이
----

[@JEEN_LEE](http://twitter.com/JEEN_LEE) , 하니아빠

개요
---

각 문화권별로 대표적인 성인이 있습니다. 한국에서는 대표적으로 부처님, 예수님이 있겠죠. 마침 부처님 오신날의 위치선정이 뛰어나 중생들을 자비로 다스려주시니... 월요일은 쉬는날입니다.

그래서 오늘은 펄로 부처님과 예수님의 위치선정능력을 확인해보고자 합니다.

자비는 개뿔
--------

자비로움을 확인하기 이전에, 성인들께서 얼마나 잔혹해질 수 있는 지 부터 확인합니다. 대상은 2000년에서 2050년을 기준으로 주말에 걸치느냐입니다.

    use strict;
    use warnings;
    use Date::Holidays::KR;
    use Date::Korean;
    use DateTime;
    
    my $oops_year;
    for my $year (2000 .. 2050) {
        push @{ $oops_year->{B} }, $year if is_buddha_cruel($year);
        push @{ $oops_year->{J} }, $year if is_jesus_cruel($year);
    }

    while(my ($who, $years) = each %{ $oops_year }) {
        print $who. "\t". scalar(@{ $years }) ."\t". join(",", @{ $years })."\n";
    }

    sub is_buddha_cruel {
        my ($year) = @_;
        my ($sy, $sm, $sd) = Date::Korean::lun2sol($year, 4, 8, 0);
        my $dt_b           = DateTime->new( year => $sy, month => $sm, day => $sd );
        my $day_of_week_b  = $dt_b->day_of_week;

        return 1 if $day_of_week_b =~ /^[67]$/;
        return 1 if Date::Holidays::KR::is_solar_holiday($sy, $sm, $sd);
        return 0;
    }

    sub is_jesus_cruel {
        my ($year) = @_;

        my $dt_j   = DateTime->new( year => $year, month => 12, day => 25 );
        my $day_of_week_j = $dt_j->day_of_week;
        return 1 if $day_of_week_j =~ /^[67]$/;
        return 0;
    }

결과는 참담합니다.

    J	15	2004,2005,2010,2011,2016,2021,2022,2027,2032,2033,2038,2039,2044,2049,2050
    B	19	2002,2005,2006,2009,2016,2019,2022,2023,2025,2026,2029,2032,2036,2039,2043,2044,2046,2049,2050

대자대비의 상징인 부처님께서 50여년동안 무려 19년이나 잔혹해지시지요. 부처님이 아니라 Butcher 님입니다. 하지만 부처님도 사정이 있으시니 그 이유는 바로 음력을 먹고 들어가신다는 겁니다. 즉 5월 5일이 어린이날 + 석가탄신일 콤보를 먹고 들어갈 수 있다는 것이죠.

내가 너희 중생들을 구원할까?
---------------------

자.. 그럼 중생들을 어떻게 구원할 수 있을까요? 바로 금요일이나 월요일에 위치선정을 해주시는 것으로 중생들을 구원할 수 있습니다.

    use strict;
    use warnings;
    use Date::Holidays::KR;
    use Date::Korean;
    use DateTime;

    my $happy_year;
    for my $year (2000 .. 2050) {
        push @{ $happy_year->{B} }, $year if is_buddha_merciful($year);
        push @{ $happy_year->{J} }, $year if is_jesus_merciful($year);
    }

    while(my ($who, $years) = each %{ $happy_year }) {
        print $who. "\t". scalar(@{ $years }) ."\t". join(",", @{ $years })."\n";
    }

    sub is_buddha_merciful {
        my ($year) = @_;
        my ($sy, $sm, $sd) = Date::Korean::lun2sol($year, 4, 8, 0);
        my $dt_b           = DateTime->new( year => $sy, month => $sm, day => $sd );
        my $day_of_week_b  = $dt_b->day_of_week;

        return 0 if Date::Holidays::KR::is_solar_holiday($sy, $sm, $sd);
        return 0 if $day_of_week_b =~ /^[67]$/;
        return 1 if $day_of_week_b =~ /^[15]$/;
        if ($dt_b->month eq '5' && $dt_b->day =~ /^[46]$/) {
            return 1;
    }
        return 0;
    }

    sub is_jesus_merciful {
        my ($year) = @_;

        my $dt_j   = DateTime->new( year => $year, month => 12, day => 25 );
        my $day_of_week_j = $dt_j->day_of_week;
        return 1 if $day_of_week_j =~ /^[15]$/;
        return 0;
    }

 이 또한 참담한 결과가 나왔습니다.

    J	14	2000,2006,2009,2015,2017,2020,2023,2026,2028,2034,2037,2043,2045,2048
    B	10	2008,2010,2012,2013,2014,2015,2033,2037,2040,2042
 
 무려 부처님의 경우에는 어린이날 옆에 위치할 수 있는 어드벤티지가 있음에도 불구하고 말이죠.

이 2000년에서 2050년이라는 기준은 현재 우리세대가 직장인으로 활약할 것이라고 보는 뭐 대충 그런 기간입니다. ... 라는 것도 있고 사실 위에서 음력 판별을 위해 사용한 [Date::Korean](https://metacpan.org/module/Date::Korean) 이 2050년까지 지원하는 것이 더 큰 이유에서 입니다.

결론
---

원하던 결론은 아니었습니다. 적어도 부처님오신날 기념 달력인지라 부처님의 자비가 더 위일 줄 알았는데 말이죠. 

일체유심조라는 말이 있습니다. 모든 것은 마음먹기에 달려있다라는 것이죠. 로또되면 휴일이고 평일이고 뭐가 상관있겠습니까? 해마다 일희일비하는 것도 한계가 있지... 그냥 맘 편하게 좋은 위치선정으로 연차쓰는 것이 확실합니다.

사실 이 글은 부처님의 자비로움.jpg 를 보고서 쓰게 되었음을 알려드립니다.

![img](https://p.twimg.com/AtoRYLsCIAAfGD0.jpg)

적어도 부처님은 2015년까지는 압도적으로 자비로우십니다.
