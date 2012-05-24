나 지금 바쁘니까 나중에 할게
=====================

글쓴이
----

[@JEEN_LEE](http://twitter.com/JEEN_LEE), 하니아빠, 킨들러리스트


## 시작하며

보통 우리는 컴퓨터를 사용하여 어떤 처리를 통해서 결과를 얻게 됩니다. 하지만 그 결과는 지금봐도 되고, 나중에 봐도 되죠. 지금 당장 모든 결과를 보겠다고 꾸역꾸역 이것저것 처리를 한꺼번에 하게 된다면 그만큼 대기 시간은 길어지게 됩니다.

그러니 당장 필요한 최소한의 결과만을 먼저 얻어내고, 그 이외의 것들은 따로 처리하여 별도로 결과를 축적/확인할 수 있다면 그 대기시간은 짧아지겠죠.

이런 방법들을 위해서 실제 동작하고 있는 서비스의 코드를 예로 설명하고자 합니다.

## 어떤 경우에?

아래는 특정항목에 대한 고객의 요구사항입니다.

1. 사진을 올리겠다.
2. 사진을 올리면 센터를 기준으로 특정부분을 잘라서 보관했으면 싶다.
3. 그리고 썸네일도 만들었으면 싶다.
4. 그리고 ...
5. 그리고 ...
6. 그리고 ...
7. 그리고 ...
8. 아, 또 스마트폰에 푸쉬도 되게 했음 싶다.

대충 밝힐 수 없는 부분까지 포함해서, 스마트폰에서 사진을 뿅하고 올렸을 때 해야할 작업내용들이 주루루루룩 나열되어 있습니다. 엄청 핫한 스펙이라면 어떻게든 되겠지 하며 대충대충 써버리겠지만 돈이 없어 장비빨도 안받쳐주니...

아무튼 모든 Request 의 처리기준은 절대로 1초를 넘기지 않는다라는 고집을 발휘해봅니다.

## 써볼까 Job Queue

Perl 에서 대표적인 Job Queue 관련 모듈은 [TheSchwartz](https://metacpan.org/module/TheSchwartz) 가 있습니다. 큐를 DB 에 때려박아서 특정시간을 반복해서 DB 에 쌓인 큐가 있으며 뽑아서 해치우는 그런 방식입니다. 뭐 그래서.. 네, 즉시성이라는 게 떨어집니다. (조정은 가능하지만 기본 3초였던 걸로)

하지만 TheSchwartz 를 사용했을 때의 문제는 Priority 를 지정했음에도 불구하고 해당 Priority 순서대로 Queue 나오지 않는다는 문제가 있었습니다. 그러니까 FIFO 이 아니라 그냥 랜덤하게 막 뽑혀나오는 사태가 벌어지는 것이죠(아, 물론 바로 랜덤으로 뽑히는 건 아닙니다)

TheSchwartz 의 코드를 확인하면 Priority 로 뽑아서 순서를 만들어서 결과를 보여주어야할 타이밍 이전에 한번 결과를 섞어버립니다.

- [## Got some jobs! Randomize them to avoid contention between workers.](https://github.com/saymedia/TheSchwartz/blob/master/lib/TheSchwartz.pm#L317)

그러는 이유가 워커들까리의 경합이 발생하는 것 때문이라고 하지만... 그럴려면 왜 Priority 가 필요한지 의문이기는 합니다.

일단 기존의 코드를 존중함과 동시에 원하는 바(Priority 순서대로 뽑혔으면..) 를 충족하기 위한 방법도 마찬가지로 소스안에서 소개되고 있습니다.

- [https://github.com/saymedia/TheSchwartz/blob/master/t/priority.t#L20](https://github.com/saymedia/TheSchwartz/blob/master/t/priority.t#L20)

`$FIND_JOB_BATCH_SIZE` 를 1로 조정함으로써 매번 Queue DB 에 질의를 날릴 시에 Priority 기준으로 하나의 결과만을 뽑는 것입니다. 그렇기에 결과를 섞든 뭘 하든 결국 하나만 나오니까 이렇게함으로 Priority 가 지켜지게 되는 것입니다. 하지만 `$FIND_JOB_BATCH_SIZE` 를 1로 조정한다는 건 기존의 기본값인 50에 비해서 잦은 횟수로 DB 에 질의를 날려야 된다는 것입니다. 하지만 그런 것을 걱정할 규모까지는 아닌 것 같아서 그냥 거기까지는 걱정을 접기로 합니다.

결론은 저는 TheSchwartz 를 쓰지 않게 되었다는 것이죠.

## 그럼 어떤 걸 쓸까?

- [Qudo - simple and extensible job queue manager](https://metacpan.org/module/Qudo)

저는 그래서 그냥 Qudo 라는 놈을 골랐습니다. TheSchwartz 처럼 Queue 저장소를 DB 를 사용합니다. 스키마를 보면 같다고 봐도 되구요. 거기에 위에서 언급한 Priority 문제가 Qudo 에는 없습니다. 또한 다양한 후크포인트를 지정할 수 있습니다. 큐DB에 넘기는 파라메터의 Serialize/Deserialize 때라든가, Enqueue 전후, 그리고 Worker 에서 처리하기 전후에 대해서 다양한 처리를 심어넣을 수가 있습니다. 예를들어 뭐 로깅이라든가 그런 거 말이죠.

아무튼 바로 코드로 들어가 보겠습니다. 우선 Catalyst 에서 Qudo 를 사용하기 위해서 Qudo 에 접근하기 위한 모델을 만듭니다.


    package MyApp::Web::Model::Queue;
    use Moose;
    use namespace::autoclean;

    extends 'Catalyst::Model::Adaptor';

    sub mangle_arguments {
        my ($self, $args) = @_; 

        return %$args;
    }
    __PACKAGE__->meta->make_immutable;
    1;

그리고 이 모델에 대한 설정정보는 아래와 같습니다.

    "Model::Queue" => {
        class => "Qudo",
        args => {
            databases => [
                { dsn => 'dbi:mysql:myapp_queue:xxxx',
                  username => 'xxxxx',
                  password => 'xxxxx',
                },
                ....,
            ],
            default_hooks => [
                'Qudo::Hook::Serialize::JSON'
            ],
        },
    },

`C::M::Adaptor` 를 통해서 Qudo 를 그대로 연결합니다. 이때 Qudo 는 HashRef 가 아닌 Hash 를 인수로 받기에 `mangle_arguments` 를 통해서 인수를 정형하도록 합니다.

설정항목에서는 큐DB 의 접속정보와, 파라메터를 Serialize 할때 JSON 으로 하기위해서 위처럼 `default_hooks` 값을 지정해줬습니다. 

그리고 Catalyst Controller 에서 특정 상황에서 enqueue 를 해줍니다.

    sub enqueue_notification {
        my ($self, $args, $uniqkey) = @_; 

        $args->{apns} = $self->config->{push}->{apns};
        $args->{GoogleLogin} = $self->config->{push}->{c2dm}->{GoogleLogin};

       $self->model('Queue')->enqueue('MyApp::Worker::Notify', {
            arg => $args,
            uniqkey => $uniqkey.':'.time(),
       }); 
    }

특정상황에서의 Push 를 할 시에 위의 코드를 사용해서 enqueue 를 하도록 합니다. `MyApp::Worker::Notify` 라는 Job에 `arg` 에 넘기는 파라메터는 JSON 으로 Serialize 되게 됩니다.

`MyApp::Worker::Notify`는 아래와 같습니다.

    package MyApp::Worker::Notify;
    use Moose;
    use namespace::autoclean;
    extends 'MyApp::Worker';
    use Net::APNS::Persistent;
    use HTTP::Request::Common;
 
    sub work {
        my ($self, $job) = @_; 

        my $args = $job->arg;

        # Net::APNS::Persistent 로 iOS Notification..
        # CD2M 에 Request 하며 Android 에 Notification..

        $job->completed;
    }
    __PACKAGE__->meta->make_immutable;

    1;

자 그럼 이렇게 할 일들을 만들어 뒀으면 일꾼을 움직여야 되겠죠.

    use strict;
    use warnings;
    use Config::ZOMG;
    use Qudo;
    use Qudo::Parallel::Manager;

    my $configloader = Config::ZOMG->new(
        name => 'MyApp::Web'
    );

    my $config = $configloader->load;
    my $qudo_args = $config->{'Model::Queue'}->{args};
    $qudo_args->{manager_abilities} = [qw/
        MyApp::Worker::Image::Thumbnail
        MyApp::Worker::Image::Crop
        MyApp::Worker::Notify
    /];

    my $manager = Qudo::Parallel::Manager->new(%{ $qudo_args },
        work_delay            => 3,
        max_workers           => 3,
        min_spare_workers     => 1,
        max_spare_workers     => 3,
        max_request_per_child => 30, 
        auto_load_worker      => 1,
        admin                 => 1,
        debug                 => 0,
    );

    $manager->run;

기존의 QueueDB 접속 정보등은 Catalyst App 의 설정항목을 그대로 가져와서 사용하며, `manager_abilities` 에 위에서 정의한 Worker 클래스들을 지정해줍니다.

또한 `Qudo::Parallel::Manager` 를 사용함으로 인해서 동시에 올라가는 워커의 갯수등을 설정할 수 있고, 혹시나 모를 메모리누수등의 문제에 대비해서 MaxRequestPerChild 를 통해서 30번정도 돌아갔으면 워커를 다시 띄울 수 있게 할 수 있습니다.

또한,

    use IO::Socket::INET;
    my $sock = IO::Socket::INET->new(
        PeerHost => '127.0.0.1',
        PeerPort => 90000,
        Proto    => 'tcp',
    ) or die 'can not connect admin port.';
 
    # get scoreborad
    # ex)   _ . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . .
    my $status = $sock->getline;
    $sock->close;

위의 코드를 통해서 Scoreboard 도 얻을 수 있습니다. 이를 통해서 Worker 들이 제대로 동작하는 지 등등을 가볍게 체크할 수도 있겠죠.

## 결론

웹서비스를 만들고 운영해나가는 데 있어서 조그만 고집같은 게 있습니다. 위에서 말한 모든 리퀘스트는 1초안에 처리되어야 된다든가 하는 그런 거 말이죠. 물론 뭐 고집도 중간중간에 꺾여야 몸도 마음도 편할 때도 있기는 합니다. 당장은요.

Job Queue Manager 의 선택지는 무궁무진합니다. 최신 트렌드에 맞고 속도도 무지빠르고 그런 것들도 많겠지만, 어디까지나 규모에 맞게 __적정기술__ 을 선택하는 방법이 좋지 않을까 하는 뭐 그런 생각을 해봅니다.
