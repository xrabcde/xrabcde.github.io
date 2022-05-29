---
layout: post
title: Mac/M1 맥북에 Jekyll 깃헙블로그 로컬환경 세팅하기
date: 2022-05-29
excerpt: "M1 Mac에 Jekyll Github 블로그 로컬 테스트환경을 구축해보자"
tags: [github, jekyll, m1, mac]
comments: true
---

## 개요
이번에 M1 맥북에 글또 활동을 위해 로컬 jekyll 블로그 환경 세팅을 해보았다. 
이 과정에서 다양한 이슈들을 마주쳤지만 해결에 급급했기 때문에 바로바로 기록을 남겨두지 못했다.  
하지만 혹시나 누군가 (아마도 미래의 내가 되겠지..) 아래 내가 겪었던 문제들과 비슷한 문제를 만난다면 이 중에 하나는 정답이겠지... 하는 마음으로... 
Chrome 방문기록과 터미널 커맨드 히스토리를 더듬어가며 이슈들과 시도한 방법들을 정리해보려고 한다. 

## Ruby 설치

먼저 brew를 통해 rbenv와 ruby를 설치해주어야 한다.

```
$ brew update
$ brew install rbenv ruby-build
```

`rbenv versions`를 입력하면 설치되어있는 버전을 확인할 수 있다.
m1 맥북에는 기본적으로 rbenv가 설치되어있기 때문에 system 한 개가 뜨고 `*` 표시로 선택되어있는 것을 확인할 수 있다. 

```
$ rbenv versions
* system
```

`rbenv install -l` 명령어를 입력하면 설치가능한 rbenv 버전 목록을 확인할 수 있다. 
자신의 jekyll 블로그 테마의 호환성을 확인해 설치할 버전을 선택하면 된다.

```
$ rbenv install -l 
```

<div style="width:100% !important; margin:0 auto">
<img src="/assets/img/m1-jekyll1.png" alt="m1-jekyll1.png">
</div>

나는 2.6.10 버전을 설치했다. (이후에 3.1.2로 다시 설치하긴 했지만...)  
하지만, `rbenv install 2.6.10` 를 입력해 rbenv를 설치하려고 하면 자꾸 아래와 같은 에러가 나오며 설치가 되지 않았다.

```
$ rbenv install 2.6.10

...

ruby-build: using readline from homebrew

BUILD FAILED (macOS 12.0.1 using ruby-build 20211019)

Inspect or clean up the working tree at /var/folders/_r/73hpqvps01n344k0bld57x800000gn/T/ruby-build.20211028131523.17883.QC3g3a
Results logged to /var/folders/_r/73hpqvps01n344k0bld57x800000gn/T/ruby-build.20211028131523.17883.log

Last 10 log lines:
        imemo_type_ids[9] = rb_intern("imemo_parser_strterm");
                            ^~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
../.././include/ruby/ruby.h:1826:56: note: expanded from macro 'rb_intern'
        __extension__ (RUBY_CONST_ID_CACHE((ID), (str))) : \
                                                       ^
1340 warnings generated.
linking shared-object objspace.bundle
422 warnings generated.
linking shared-object date_core.bundle
make: *** [build-ext] Error 2
```
이 때부터 로그를 구글링해가며 나오는 해결방법들을 전부 시도해보았다.  
rvm으로도 시도했지만 잘 되지 않았고...

<div style="width:100% !important; margin:0 auto">
<img src="/assets/img/m1-jekyll2.png" alt="m1-jekyll2.png">
</div>


### (mach-o file, but is an incompatible architecture (have 'x86_64', need 'arm64e')

그러던 중 `arm64e` 관련 로그로 구글링을 하다가 아래와 같은 명령어를 [시도해보라는 글](https://github.com/rbenv/ruby-build/issues/1700#issuecomment-759893415)을 찾았다.  
이 글을 참고해 설치를 해보았고

```
$ arch -x86_64 /bin/bash -c 'rbenv install 2.6.0'
```

설치가 되고 난 후 터미널 설정을 바꾸어주어야 해서 .zshrc 파일을 수정해주었다.

```
$ vi ~/.zshrc

export PATH="$HOME/.rbenv/bin:$PATH"
eval "$(rbenv init -)"

$ source ~/.zshrc
```

이제 다시 아까 입력했던 rbenv versions 명령어를 입력해보면 새로 설치한 2.6.0 버전이 추가된 것을 확인할 수 있다. 하지만 여전히 system 에 * 선택이 되어있으므로 이를 global 명령어를 이용해 바꿔주어야 한다.

```
$ rbenv versions

$ rbenv global 2.6.0
```

하지만 이후에도 문제가 잘 해결되지 않아서 rbenv를 가장 최신 버전인 3.1.2 버전으로 재설치해주었다. 호환성에 크게 문제가 되지 않는다면 처음부터 그냥 최신 버전을 설치하시길...

```
$ arch -x86_64 /bin/bash -c 'rbenv install 3.1.2'

$ rbenv global 3.1.2
```

### Bus Error
이렇게 rbenv까지 잘 설치가 되어서 끝난 줄 알았더니 `bundle exec jekyll serve`를 입력하면 아래와 같이 Bus Error 가 났다.

<div style="width:100% !important; margin:0 auto">
<img src="/assets/img/m1-jekyll3.png" alt="m1-jekyll3.png">
</div>

이번엔 [이 글](https://synoti21.github.io/blog%20dev/There-are-no-gemspecs-at-~~-%ED%95%B4%EA%B2%B0-%EB%B0%A9%EB%B2%95-(%EA%B9%83%ED%97%99-%ED%8E%98%EC%9D%B4%EC%A7%80-%EA%B2%8C%EC%8B%9C%ED%95%A0-%EB%95%8C)/)을 참고해 gem 재설치를 시도해보았다.

```
$ sudo gem uninstall sassc
$ sudo gem install sassc -- --disable-march-tune-native
$ sudo gem uninstall ffi
$ sudo gem install ffi
$ bundle update --bundler
```

```
$ bundle install
```

그러면 문제가 해결되지는 않지만 뭔가가 주루룩 로그가 뜨면서 설치가 되는 듯한 모습을 확인할 수 있고, 다음 에러를 만날 수 있었다.

### Gem::FilePermissionError
```
ERROR: While executing gem ... (Gem::FilePermissionError)
You don't have write permissions for the /Library/Ruby/Gems/2.6.0 directory
```

이런 Permission 에러 같은 경우는 간단히 앞에 sudo를 붙여주는 방법으로 해결이 되었고

```
$ sudo gem install jekyll
```

<div style="width:100% !important; margin:0 auto">
<img src="/assets/img/m1-jekyll4.png" alt="m1-jekyll4.png">
</div>

### cannot load such file -- webrick (LoadError)
그렇게 이제 진짜 되겠지 하고 `bundle exec jekyll serve` 명령어를 입력했지만 이번엔 webrick 에러를 만날 수 있었다.

```
bundler: failed to load command: jekyll (/Users/kakao_ent/.rbenv/versions/3.1.2/bin/jekyll)
/Users/kakao_ent/.rbenv/versions/3.1.2/lib/ruby/gems/3.1.2/gems/jekyll-3.9.0/lib/jekyll/commands/serve/servlet.rb:3:in `require': cannot load such file -- webrick (LoadError)
```

이번에도 [위의 글](https://synoti21.github.io/blog%20dev/There-are-no-gemspecs-at-~~-%ED%95%B4%EA%B2%B0-%EB%B0%A9%EB%B2%95-(%EA%B9%83%ED%97%99-%ED%8E%98%EC%9D%B4%EC%A7%80-%EA%B2%8C%EC%8B%9C%ED%95%A0-%EB%95%8C)/)을 참고해  webrick을 수동으로 추가해주었다.

```
$ bundle add webrick
```

### missing gems : nokogiri

<div style="width:100% !important; margin:0 auto">
<img src="/assets/img/m1-jekyll5.png" alt="m1-jekyll5.png">
</div>

수동으로 webrick을 추가해주는 과정에서는 nokogiri 에러를 만날 수 있었는데 
이 에러는 [이 글](https://chobolife.github.io/blog/2019/08/12/jekyll-theme/)을 참고해 해결할 수 있었다. 
아래 명령어들을 순서대로 입력했다.

```
$ sudo gem install nokogiri
$ gem update
$ bundle update
$ bundle
```

### NoMethodError: undefined method `full_name' for nil:NilClass
마지막으로 위의 해결과정 중 bundle 명령어를 입력하는 과정에서 아래와 같은 에러를 만났다.

```
NoMethodError: undefined method `full_name' for nil:NilClass

  warning << "* #{unmet_spec_dependency}, depended upon #{spec.full_name}, unsatisfied by #{@specs.find {|s| s.name == unmet_spec_dependency.name && !unmet_spec_dependency.matches_spec?(s.spec) }.full_name}"
                                                                                                                                                                                                   ^^^^^^^^^^                 
```

<div style="width:100% !important; margin:0 auto">
<img src="/assets/img/m1-jekyll6.png" alt="m1-jekyll6.png">
</div>

가물가물하지만 이 에러는 [이 글](https://github.com/rubygems/rubygems/issues/5088)을 참고해 해결이 되었던 것 같다.

```
$ bundle install
```

### 성공!
이제 `bundle exec jekyll serve` 을 입력해서 아래와 같이 로컬서버가 구동이 되면 성공이다!!!!

<div style="width:100% !important; margin:0 auto">
<img src="/assets/img/m1-jekyll7.png" alt="m1-jekyll7.png">
</div>


## References
- [https://github.com/rbenv/ruby-build/issues/1700#issuecomment-759893415](https://github.com/rbenv/ruby-build/issues/1700#issuecomment-759893415)
- [https://synoti21.github.io/blog%20dev/There-are-no-gemspecs-at-~\~-%ED%95%B4%EA%B2%B0-%EB%B0%A9%EB%B2%95-(%EA%B9%83%ED%97%99-%ED%8E%98%EC%9D%B4%EC%A7%80-%EA%B2%8C%EC%8B%9C%ED%95%A0-%EB%95%8C)/](https://synoti21.github.io/blog%20dev/There-are-no-gemspecs-at-~~-%ED%95%B4%EA%B2%B0-%EB%B0%A9%EB%B2%95-(%EA%B9%83%ED%97%99-%ED%8E%98%EC%9D%B4%EC%A7%80-%EA%B2%8C%EC%8B%9C%ED%95%A0-%EB%95%8C)/)
- [https://chobolife.github.io/blog/2019/08/12/jekyll-theme/](https://chobolife.github.io/blog/2019/08/12/jekyll-theme/)
- [https://frhyme.github.io/blog/install_jekyll_again/](https://frhyme.github.io/blog/install_jekyll_again/)
- [https://github.com/rubygems/rubygems/issues/5088](https://github.com/rubygems/rubygems/issues/5088)