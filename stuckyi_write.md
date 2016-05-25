<table id="meta">
	<thead><th>160525</th><th>Lab - Application</th><th>김태경</th></thead>
	<tbody>
	<tr><td></td><td></td><td></td></tr>
	<tr><td></td><td></td><td></td></tr>
	<tr><td></td><td></td><td></td></tr>
	</tbody>
</table>

# 데이터 기반 애플리케이션 제작
이 글에서는 유엔 인간 개발 데이터 API를 활용하여 데이터 기반 애플리케이션을 만들어 본다. 재사용 가능한 컴포넌트 작성을 위해 D3를 사용하고 애플리케이션 상태를 구조화하고 관리하기 위해 백본을 적용한다. 템플릿과 감단한 마크업 언어를 이요해서 웹 애플리케이션을 만들 수 있는 제킬을 사용하는 방법도 알아본다 우리가 만든 정적인 사이트를 아마존 S3와 깃허브 페이지를 이용해서 호스팅하는 방법도 함께 살펴본다.
 
 
 ## 웹 애플리케이션 생성
 
 이 절에서는 국가별 인간 개발지수 HDI - Human Development Index 의 변화를 탐색해 볼 수 있는 데이터 시각화를 만들어 보고 인간 개발 지수의 구성 요소인 기대 수명 교육 수준 소득 수준 등에 대한 정보를 보여줄 것이다
 
 
 앞서 언급한 것과 같이 차트를 만들고 애플리케이션의 구조를 만들기 위해 D3와 backbone을 사용한다 다음과 같이 의존 라이브러리를 설치한다.
 
 <pre><code>
 $ bower install --save-dev d3 backbone underscore
 </code></pre>
 이렇게하면 bower_component 디렉토리가 만들어지고, 의존하는 라이브러리 패키지가 설치된다. HDI 컴포넌트의 디자인과 아이콘을 위해 부트스트랩과 FontAwesome도 설치한다. 부트스트랩은 jQuery에 의존하고 있으므로 바우어가 jquery를 자동으로 설치해 줄 것이다.
 
 <pre><code>
 $ bower install --save-dev bootstrap font-awesome typehead.js
 </code></pre>
 
 검색어 자동입력 완성 기능을 위해 트위터의 Typehead 라이브러리를 이용할 것이다 `--save-dev`옵션을 주고 설치하면 `bower.json`파일의 내용 중 개발의존 관계를 나타내는 `devDependencies`의 내용이 함께 업데이트된다 `--save-dev`옵션을 준 라이브러리도 일반적인 의존 관계와 다를 게 없지만, 나중에 모든 의존 라이브러리르 하나의 파일에 담기 위해 `--save-dev`옵션을 사용해서 개발용 의존 관계에 필요한 라이브러리를 추가한다.
 
 바우어를 쓰면 안전하게 의존 관계를 관리할 수 있다 바우어는 시맨틱 버전 번호 부여를 필수 요건으로 하고 있으며, 구버전 호환성이 보장된 배포판만 업데이트한다.
 
 
 ## 재킬을 이용한 정적 사이트 생성
 
 이번 절에서는 제킬을 써서 웹사이트를 생성하는 방법과 제킬과 깃허브 페이지를 함께 사용해서 웹앱을 호스팅하는 방법을 살펴본다.
 
 > 제킬은 루비로 작성되었으며, 간단하게 블로그 같은 정적 사이트를 만들 수 있는 도구다. 제킬을 쓰면 웹 서버나 데이터베이스의 설정 없이도 콘텐츠를 담고 있는 웹사이트나 블로그를 만들 수 있다. 웹 페이지를 만들려면 간단한 마크업 언어를 써서 작성한 다음 HTML로 컴파일 하면 된다 제킬을 템플릿 파일을 사용할 수도 있으며 HTML코드도 부분적으로 지원한다.
 
 ### 제킬의 설치
 리눅스나 OS X에서는 터미널 명령창에서 제킬을 설치할 수 있다. 제킬은 루비로 작성되었으므로 제킬을 사용하려면 먼저 루비와 `RubyGems`가 설치되어야 한다. 다음의 명령어로 제킬을 설치할 수 있다.
 
 <pre><code>
 $ gem install Jekyll
 </code></pre>
 
  제킬은 프로젝트 기본 디렉토리 구조(bolierplate)와 예제 콘텐츠 및 템플릿 (Jekyll new --help참고)을 제공하지만, 여기서는 백지 상태에서 직접 템플릿, 콘텐츠를 만들고 설정해 볼 것이다. 먼저 제킬의 디렉토리 구조를 만들어 보자.
  
  
<pre class="highlight"><code>
//제킬에게 사이트를 만들도록 지시한다.
$ jekyll build
</code></pre>
  
  
<pre><code>
//제킬의 디렉터리 구조

hdi-explorer/
    _includes/
        navbar.html
    _layouts/
        main.html
    _data/
    _drafts/
    _posts/
    index.md
    _config.yml  
</code></pre>
  
  #### `_config.yml` 파일
   
  `_config.yml` 파일은 JSON과 비슷하지만 추가적인 타입과 주석을 지원하고 중괄호 대신 들여 쓰기를 사용하는 직렬화 표준인 [YAML](http://yaml.org)로 작성 된 설정 파일이다.  `_config.yml` 파일에는 제킬이 콘텐츠를 생성해 내는 방법과 사이트 전체 범위에서 사용되는 변수를 정의한다. 우리 프로젝트에서는 `_config.yml`파일에 몇 가지 제킬 옵션과 우리가 만들 사이트의 이름, 기준 URL, 저장소를 정의한다.
  
  <pre><code>
  # 제킬 설정
  safe: true
  markdown: rdiscount
  permalink:    pretty
  
  # Site 정보
  name:  Human Development Index Explorer
  baseurl: http://tadkim.github.io/hdi-explorer
  github:http://github.com/tadkim/hdi-explorer.git
  
</code></pre>

- `safe` : 모든 제킬 플러그인을 비활성화
- `markdown` : 우리가 사용할 마크다운 언어를 지정
- `permalink` : 생성할 URL의 타입을 지정한다.
- `timezone` : 시간대
- `excluded files` : 배제할 파일

####  `_layout` 디렉토리

`_layout` 디렉토리에는 콘텐츠로 치환 될 자리 채우미(place holder)를 가진 페이지 템플릿이 포함되어 있다. 어떤 레이아웃을 사용할 것인 지는 각 페이지의 YAML front matter라고 불리는 특정 영역에서 선언할 수 있다. 템플릿은 `{{ content }}`태그와 같은 변수를 포함하고 있고, `{% include navbar.html %}`와 같은 태그도 포함하고 있다. 변수의 내용은 `front matter`에 저장할 수 있고, `_config.yml`에 정의할 수도 있다. 

`include`태그는 `_include` 디렉토리에 있는 파일로 해당 영역을 대체한다. 예를 들어, `main.html` 템플릿은 다음과 같은 기본 페이지 구조를 가지고 있다.

<pre class="highlight"><code>
&lt;!DOCTYPE html&gt;
&lt;html lang="ko"&gt;
&lt;head&gt;
	&lt;meta charset="utf-8"&gt;
	&lt;title&gt; {{ page.title }} &lt;/title&gt;
	&lt;link href="{{ site.baseurl }}/hdi.css" rel="stylesheet"&gt;
&lt;/head&gt;
&lt;body&gt;
	&lt;!-- Navigation Bar --&gt;
	{% include navbar.html %}

	&lt;!-- Content --&gt;
	&lt;div class="container-fluid"&gt;
		{{ content }}
	&lt;/div&gt;
&lt;/body&gt;
&lt;/html&gt;
</code></pre>

`{{ site.baseurl }}` 변수의 값은 `_config.yml` 파일에 저장되어 있고, `{{ page.title }}`변수는 템플릿을 이용하는 실제 페이지의 `front matter`에 정의된 값으로 치환되며, `{{ content }}` 변수는 템플릿을 이용하는 실제 페이지의 내용으로 치환된다. 

`_config.yml` 파일에 `http://tadkim/github.io/hdi-explorer`를 가리키는 `baseurl`변수를 정의했다. 제킬 콘텐츠 생성을 위해 이 템플릿을 사용하면, `baseurl`변수의 값인 `http://tadkim.github.io/hdi-explorer`가 `{{ site.baseurl }}`을 대체하게 된다. 생성된 페이지에서는 CSS 파일에 대한 경로가 `http://tadkim.github.io/hdi-explorer/hdi.css`로 올바르게 표시된다. `Liquid` 템플릿 언어에 대한 자세한 내용은  [Liquid 공식 홈페이지](http://liquidmarkup.org/)를 참고하자.

#### `_include`디렉토리

`_include` 디렉토리는 다른 페이지에 포함될 HTML 프래그먼트(fragments)가 포함되어 있다. HTML 프래그먼트는 머리말(header), 꼬리말(footer)이나 내비게이션 바(navigation bar)등 페이지의 일부분을 모듈화 할 때 아주 유용하다. 여기서는 내비게이션 바 역할을 할 `navbar.html`파일을 작성한다.

##### navbar.html 파일 작성

`navbar.html`파일은 다음과 같이 작성한다.

<pre class="highlight"><code>
<!-- 네비게이션 바-->
&lt;nav class="navbar navbar-default" role="navigation"&gt;
	&lt;div class="container-fluid"&gt;
	<!-- ... 다른 요소들 ... -->
	&lt;a class="navbar-brand" href="#"&gt; {{ site.name }} &lt;/a&gt;
	<!-- ... ... -->
	&lt;/div&gt;
&lt;/nav&gt;
</code></pre>

`navbar.html` 파일의 내용은 템플릿에 `{ % include navbar.html %}`로 표시된 부분을 대체한다. `_includes` 디렉토리에 있는 파일은 `Liquid` 태그를 포함할 수 있다.


#### `_posts`디렉토리

`_posts` 디렉토리에는 블로그 포스트가 저장된다. 각 블로그 포스트는 "연도-월-일-제목.MARKDOWN"의 이름 형식으로 저장되어야 한다. 제킬은 각 포스트의 이름을 기준으로 작성일과 URL을 계산한다. 


#### `_data`디렉토리

`_data` 디렉토리는 사이트 전체 범위에 사용되는 추가적인 변수 정보를 포함한다.
 
 #### `_draft`디렉토리
 
 `_draft` 디렉토리는 나중에 발행할 포스트 초안을 보관하고 있다. 
 
 이번 프로젝트에서는 포스트나 초안은 만들지 않을 것이다.
 
 #### `index.md`파일
 
 `index.md` 파일은 프로젝트의 레이아웃을 이용해 렌더링될 콘텐츠를 포함하고 있다 파일의 시작 부분에 3개의 대시(-)로 표기되어 있는 YAML front matter가 있다 이 부분은 YAML코드로 인터프리트 되어 템플릿을 렌더링 하는 데 사용된다. 예를 들면, 메일 템플릿에는 `{{ page.title }}`라는 자리 채우미가 있는데, 페이지가 렌더링 되면 제킬은 `{{ page.title }}`을 front matter에 정의된 `title` 변수의 값으로 치환한다. font matter 이후의 내용은 템플릿에 `{{ content }}` 태그로 표시된 부분을 대체하게 된다.
 
 <pre class="highlight"><code>
 ---
 layout:main
 title:HDI Explorer
 ---
 &lt!-- Content --&gt;
 Hello world
 </code></pre>
 
 `layout`변수는 지킬이 페이지 렌더링을 위해 어떤 레이아웃을 적용해햐할지 알아야 하기 때문에  필수다
 
 <pre class="highlight"><code>
 $ jekyll build
 </code></pre>
 
 이 명령을 실행하면 `_layouts`이나 `_includes` 처럼 제킬과 직접적으로 관련되는 디렉토리 외에 모든 콘텐츠를 포함하는 `_site` 디렉토리가 생성된다 이번 생성에서는 `index.html`파일이 포함되어 있을 것이다 생성된 `index.html`은 `index.md`, `navbar.html`, `_config.yml`파일의 내용과 `main.html`의 레이아웃이 적용되어 있다. 생성된 파일의 내용은 다음과 같다.
  
  

