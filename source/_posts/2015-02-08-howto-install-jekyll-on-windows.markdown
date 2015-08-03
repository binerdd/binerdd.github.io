---
layout: post
title: "Howto Install Jekyll On Windows"
date:   2015-02-08 03:00:25
categories: howto
---
#시작하기 전
친절한 가이드와 달리 설치과정에서 수많은 에러를 만나게 된다... 당황하지 말고 설치를 성공해보자. 

#파일 다운로드
[Ruby Installer](http://dl.bintray.com/oneclick/rubyinstaller/rubyinstaller-2.1.5.exe?direct) <br>
[Ruby DevTools](http://cdn.rubyinstaller.org/archives/devkits/DevKit-mingw64-32-4.7.2-20130224-1151-sfx.exe) <br>
[Python 2.7](https://www.python.org/ftp/python/2.7.9/python-2.7.9.msi) <br>
[pip](https://bootstrap.pypa.io/get-pip.py)

#Ruby 및 DevTools 설치
먼저 Ruby 2.1.5 32bit 버전을 설치한 후(설치 옵션에서 "Add Ruby Executables to your PATH" 선택), 
Ruby DevTools를 C:\RubyDevTools 폴더에 압축을 푼다.

(만약 빈폴더에 압축을 풀지 않으면, 해제되는 파일들이 기존파일들과 섞여버려서 구분이 안됨)

C:\RubyDevKit 폴더에 들어가면 dk.rb 라는 루비 파일이 있는데 아래와 같이 옵션을 주어 실행한다.<br>
<b>설치과정에서 사용하는 명령창은 RubyDevTools 폴더안에 있는 mingw를 사용하는 것이 편하다</b>
{% highlight bash %}
ruby dk.rb init
ruby dk.rb install 
{% endhighlight %}

#환경변수 설정
C:\RubyDevTools\bin 폴더를 시스템 환경변수 PATH에 추가하도록 하자.
(설정을 안하면, 설치과정에서 실행하는 파일들의 경로를 찾지못하는 에러 발생)

#Jekyll 설치
{% highlight bash %}
gem install jekyll 
{% endhighlight %}

위 명령어를 명령창에서 실행하면 아래와 유사하게 ssl 검증을 할 수 없다는 에러 메시지가 출력된다. 

{% highlight bash %}
> gem install jekyll
ERROR: Could not find a valid gem 'jekyll' (>= 0), here is why: 
Unable to download data from https://rubygems.org/ 
- SSL_connect returned=1 errno=0 state=SSLv3 read
server certificate B: certificate verify failed 
(https://rubygems.org/latest_specs.4.8.gz)
{% endhighlight %}

이럴 경우 gem 의 소스 url을 HTTP 주소로 아래와 같이 바꿔주면 된다. 

{% highlight bash %}
gem source -r https://rubygems.org
gem source -a http://rubygems.org
{% endhighlight %}

#Python 설치
Jekyll에서는 syntax highlighting을 제공하기 위해 Pygments 라는 것을 사용하는데, 윈도우에서 이를 사용하기 위해서는
Python 기반의 버전을 설치해야한다. 
<b>파이썬 2.7 버전</b>을 설치할때, 마찬가지로 파이썬 경로를 시스템 환경변수에 추가하는 옵션을 활성화하도록 한다.

설치가 완료되면, 다운받은 get-pip.py파일을 파이썬으로 실행시킨다.
{% highlight bash %}
python get-pip.py
{% endhighlight %}
그리고 Pygments 설치
{% highlight bash %}
python -m pip install Pygments
{% endhighlight %}

이후에 Pygments 를 syntax highlighter로 사용할때는, jekyll로 생성한 사이트 폴더에 있는 _config.yml 파일에 옵션으로 설정해주면 된다.
{% highlight yaml %}
highlighter: pygments
{% endhighlight %}

#wdm 설치
Jekyll 2.4버전 부터는 자동으로 사이트 폴더를 모니터링하면서 변화가 있을 경우 다시 빌드해주는 기능이 기본적으로 탑재되어있다. {% highlight bash %}jekyll serve{% endhighlight %}

위 명령어의 기능을 활용하기 위해서는 윈도우에서 wdm 이라는 gem을 설치해야한다. 
{% highlight bash %}gem install wdm{% endhighlight %}

#Jekyll 실행하기
사이트를 생성하고자 하는 경로로 이동한 다음, jekyll를 통해 새로운 사이트를 생성한다.
{% highlight bash %}
jekyll new <name of the site>
{% endhighlight %}

그 후 생성된 사이트 폴더로 이동하고 jekyll를 통해 로컬 서버를 실행시키면 웹브라우저를 통해 (127.0.0.1:4000) 사이트의 모습을 동적으로 확인할 수 있다. 
{% highlight bash %}
cd <site folder>
jekyll serve
{% endhighlight %}

Jekyll의 다양한 기능 및 옵션들에 대해서는 [공식사이트](http://jekyllrb.com)를 참고합시다.

#References
[http://jekyll-windows.juthilo.com/1-ruby-and-devkit/](http://jekyll-windows.juthilo.com/1-ruby-and-devkit/)<br>
[http://jekyllrb.com](http://jekyllrb.com)<br>
[http://codingtips.kanishkkunal.in/ssl-error-ruby-gems-windows/](http://codingtips.kanishkkunal.in/ssl-error-ruby-gems-windows/)
