---
layout: docs
title: Cloning Medium with Parchment
permalink: /guides/cloning-medium-with-parchment/
redirect_from:
  - /docs/parchment/
  - /guides/building-on-parchment/
---
<!-- head -->
<style>
.codepen {
  height: 408px;
}
</style>
<!-- head -->

일관된 편집 환경을 제공하려면 데이터와 예측 가능한 비헤이비어가 일관성이 있어야합니다. DOM은 유감스럽게도 양질의 데이터 저장소가되기 위해 이 두 가지가 부족합니다. 현대 편집자를위한 솔루션은 내용을 표현하기 위해 자체 문서 모델을 유지하는 것입니다. [Parchment](https://github.com/quilljs/parchment/)는 Quill을위한 솔루션입니다. 고유 한 API 레이어를 사용하여 자체 코드베이스로 구성됩니다. Parchment를 통해 Quill이 인식하는 컨텐트 및 형식을 사용자 정의하거나 완전히 새로운 형식을 추가 할 수 있습니다.

이 가이드에서는 Parchment와 Quill이 제공 한 빌딩 블록을 사용하여 Medium의 편집기를 복제합니다. 우리는 Quill의 뼈로 시작하여, 테마 나 관계없는 모듈 또는 형식없이 시작합니다. 이 기본 수준에서 Quill은 일반 텍스트 만 이해합니다. 그러나이 가이드의 끝 부분에서 링크, 비디오 및 Tweet을 이해하게 됩니다.

### Groundwork

Quill을 사용하지 않고 그냥 텍스트 영역과 버튼으로 시작하여 가짜 이벤트 리스너에 연결하자. 우리는이 가이드 전체에서 편의상 jQuery를 사용 하겠지만 Quill이나 Parchment는 이것에 의존하지 않습니다. 또한 [Google Fonts](https://fonts.google.com/)와 [Font Awesome](http://fontawesome.io/)의 도움으로 기본 스타일을 추가 할 것입니다. 이 중 아무 것도 Quill이나 Parchment와 관련이 없으므로 신속하게 처리하겠습니다.

<div data-height="400" data-theme-id="23269" data-slug-hash="oLVAKZ" data-default-tab="result" data-embed-version="2" class="codepen"></div>


### Adding Quill Core

다음으로 우리는 테마 영역이 없는 Quill 코어, 형식 및 관계없는 모듈로 텍스트 영역을 대체 할 것입니다. 에디터에 입력하는 동안 개발자 콘솔을 열어 데모를 확인하십시오. 동작하는 Parchment 문서의 기본 빌딩 블록을 볼 수 있습니다.

<div data-height="400" data-theme-id="23269" data-slug-hash="QEoZQb" data-default-tab="result" data-embed-version="2" class="codepen"></div>

DOM과 마찬가지로 Parchment 문서도 트리구조입니다. Blot라고 하는 노드는 DOM 노드에 대한 추상화입니다. 스크롤, 블록, 인라인, 텍스트 및 나누기 등 몇 가지 Blot이 이미 정의되어 있습니다. 입력 할 때 Text Blot은 해당 DOM Text 노드와 동기화됩니다. 입력은 새로운 Block blot을 생성하여 처리됩니다. Parchment에서는 Blot은 자식을 가질 수 있어야 하므로 빈 블럭은 브레이크 Blot으로 채워집니다. 따라서 Leaf를 간단하고 예측 가능하게 처리 할 수 있습니다. 이 모든 것은 루트 Scroll blot 아래에 구성됩니다.

의미있는 구조나 서식을 문서에 제공하지 않으므로이 시점에서 입력하면 인라인 Blot을 볼 수 없습니다. 유효한 Quill 문서는 정규적이고 컴팩트해야합니다. 주어진 문서를 나타낼 수있는 유효한 DOM 트리가 하나 뿐이며 DOM 트리에는 최소한의 노드가 포함됩니다.

`<p><span>Text</span></p>`와 `<p>Text</p>`는 동일한 내용을 표현하기 때문에 전자는 무효이며 `<span>`을 풀기위한 Quill의 최적화 과정의 일부입니다. 마찬가지로 형식을 추가하면 `<p><em>Te</em><em>st</em></p>`와 `<p><em><em>Test</em></em></p>`도 가장 컴팩트 한 표현이 아니므로 유효하지 않습니다.

이러한 제약으로 인해 **Quill은 임의의 DOM 트리 및 HTML 변경을 지원할 수 없습니다**.
그러나이 구조가 제공하는 일관성과 예측 가능성으로 인해 풍부한 편집 경험을 쉽게 만들 수 있습니다.

### Basic Formatting


앞에서 Inline은 서식을 지정하지 않는다고 언급했습니다. 이는 기본 인라인 클래스에 대해 작성된 규칙이 아니라 예외입니다. 기본 블록 Blot는 블록 레벨 요소에 대해 동일한 방식으로 작동합니다.

굵은 글씨체와 이탤릭체를 구현하려면 인라인에서 상속 받아 `blotName`과 `tagName`을 설정하고 Quill로 등록하면됩니다. 상속 된 정적 메서드 및 변수의 서명에 대한 전체 참조는 [Parchment](https://github.com/quilljs/parchment/)를 참조하십시오.

```js
let Inline = Quill.import('blots/inline');

class BoldBlot extends Inline { }
BoldBlot.blotName = 'bold';
BoldBlot.tagName = 'strong';

class ItalicBlot extends Inline { }
ItalicBlot.blotName = 'italic';
ItalicBlot.tagName = 'em';

Quill.register(BoldBlot);
Quill.register(ItalicBlot);
```

우리는 여기서`strong`와`em` 태그를 사용할 때 Medium의 예제를 따르지만`b`와`i` 태그를 사용할 수도 있습니다. Blot의 이름은 Quill에 의해 형식의 이름으로 사용됩니다. Blot을 등록하면 Quill의 전체 API를 새로운 형식으로 사용할 수 있습니다.

```js
Quill.register(BoldBlot);
Quill.register(ItalicBlot);

var quill = new Quill('#editor');

quill.insertText(0, 'Test', { bold: true });
quill.formatText(0, 4, 'italic', true);
// If we named our italic blot "myitalic", we would call
// quill.formatText(0, 4, 'myitalic', true);
```

더미 버튼 처리기를 없애고 굵게 및 기울임 체 버튼을 Quill의 [`format()`](/docs/api/#format)에 연결합시다. 우리는 단순화를 위해 항상 형식을 추가하기 위해 'true'를 하드 코딩합니다. 응용 프로그램에서는 [`getFormat()`](/docs/api/#getformat)을 사용하여 임의의 범위에서 현재 서식을 검색하여 서식을 추가하거나 제거할지 여부를 결정할 수 있습니다. [Toolbar](/docs/modules/toolbar/) 모듈은 Quill에 이것을 구현하며, 여기서 다시 구현하지는 않습니다.

개발자 콘솔을 열고 새로운 굵은 기울임 꼴 및 굵은 기울임 꼴로 Quill의 [APIs](/docs/api/)를 사용해보세요. 데모에서 'quill` 변수에 액세스하려면 컨텍스트를 올바른 CodePen iframe으로 설정하십시오.

<div data-height="400" data-theme-id="23269" data-slug-hash="VjRovy" data-default-tab="result" data-embed-version="2" class="codepen"></div>

어떤 텍스트에 굵은 글씨체와 이탤릭체를 모두 적용한다면, 어떤 순서로 할지라도 Quill은 `<em>` 태그 밖에서 `<strong>` 태그를 일관된 순서로 래핑합니다.

### Links

링크는 링크 URL을 저장하기 위해 부울 이상이 필요하기 때문에 약간 더 복잡합니다. 이것은 두 가지 방법으로 링크 Blot에 영향을줍니다: 생성 및 포맷 검색. url을 문자열 값으로 나타내지만 url 키가있는 객체와 같은 다른 방법으로 쉽게 할 수 있습니다. 다른 키/값 쌍을 설정하고 링크를 정의 할 수 있습니다. 나중에 [images](#images)로 이를 보여줍니다.

```js
class LinkBlot extends Inline {
  static create(value) {
    let node = super.create();
    // 원한다면 URL이 올바른지 확인하세요.
    node.setAttribute('href', value);
    // 다른 비 형식 관련 속성을 설정해도 괜찮습니다.
    // Parchment에서는 볼 수 없으므로 정적이어야합니다.
    // ** Parchment에서 볼 수 없다는 의미는 데이터로 관리되지 않는다는 말이다.
    // ** 즉, 저장하지 않아도 되는 값들만 추가 설정 할 수 있다.
    node.setAttribute('target', '_blank');
    return node;
  }

  static formats(node) {
    // Link Blot으로 이미 결정된 노드로만 호출 될 것이므로 자신을 확인할 필요가 없습니다.
    return node.getAttribute('href');
  }
}
LinkBlot.blotName = 'link';
LinkBlot.tagName = 'a';

Quill.register(LinkBlot);
```

이제 Quill의 `format ()`에 전달하기 전에, 링크 버튼을 멋진 `prompt`에 연결시킬 수 있습니다.

<div data-height="400" data-theme-id="23269" data-slug-hash="oLVKRa" data-default-tab="result" data-embed-version="2" class="codepen"></div>


### Blockquote and Headers

Blockquotes는 Bold blot과 같은 방식으로 구현됩니다. 단, Block에서 기본 블록 레벨 Blot을 상속받습니다. **인라인 blot은 중첩 될 수 있지만 Block blot은 중첩 될 수 없습니다. 줄 바꿈 대신에 블록 blot은 동일한 텍스트 범위에 적용될 때 서로를 대체**합니다.

```js
let Block = Quill.import('blots/block');

class BlockquoteBlot extends Block { }
BlockquoteBlot.blotName = 'blockquote';
BlockquoteBlot.tagName = 'blockquote';
```

헤더는 하나의 차이점을 제외하고 정확히 동일한 방식으로 구현됩니다. 두 개 이상의 DOM 요소로 표현할 수 있습니다. 형식의 값은 기본적으로 'true'대신에 tagName이됩니다. 우리는 `formats ()`을 확장하여 이를 사용자 정의 할 수 있으며, 이는 [links](#links)와 비슷한 방식으로 가능합니다.
```js
class HeaderBlot extends Block {
  static formats(node) {
    return HeaderBlot.tagName.indexOf(node.tagName) + 1;
  }
}
HeaderBlot.blotName = 'header';
// Medium only supports two header sizes, so we will only demonstrate two,
// but we could easily just add more tags into this array
// Medium이 단 두가지의 헤더 크기를 지원하기 때문에 단 두개의 예만 보입니다.
// 그러나 이 배열에 태그를 더 추가하는 것 만으로 쉽게 가능합니다.
HeaderBlot.tagName = ['H1', 'H2'];
```

이 새로운 Blot을 각각의 버튼에 연결하고 `<blockquote>` 태그를 위한 CSS를 추가합시다.

<div data-height="400" data-theme-id="23269" data-slug-hash="NAmKAR" data-default-tab="result" data-embed-version="2" class="codepen"></div>

텍스트를 H1로 설정하고 콘솔에서`quill.getContents ()`를 실행하십시오. 우리는 사용자 정의 정적 `formats()` 함수를 보게 될 것이다. 데모에서 'quill` 변수에 액세스하려면 컨텍스트를 올바른 CodePen iframe으로 설정하십시오.

### Dividers

이제 첫 번째 Blot을 구현해 보겠습니다. 이전의 Blot 예제가 포맷팅에 기여하고 `format ()`을 구현하는 동안, 잎 Blot은 내용을 제공하고 `value ()`를 구현합니다. 리프 Blot은 Text 또는 Embed Blots 일 수 있으므로 섹션 구분선은 Embed입니다. 일단 생성되면 Embed Blots의 값은 변경되지 않으며 해당 위치의 내용을 변경하기 위해 삭제 및 재 삽입이 필요합니다.

우리의 방법론은 BlockEmbed에서 상속받는 것을 제외하고는 이전과 유사합니다. Embed는 `blots/embed`에도 존재하지만 인라인 레벨 Blot을 위한 것입니다. 우리는 디바이더 대신 블럭 레벨 구현을 원합니다.

```js
let BlockEmbed = Quill.import('blots/block/embed');

class DividerBlot extends BlockEmbed { }
DividerBlot.blotName = 'divider';
DividerBlot.tagName = 'hr';
```

클릭 핸들러는 [`insertEmbed()`](/docs/api/#insertembed)를 호출하는데, 이는 [`format()`](/docs/api/#format)처럼 우리를 위한 사용자 선택을 편리하게 결정, 저장 및 복원하지 않으므로 스스로 선택을 유지하기 위해 좀 더 많은 작업을 해야합니다. 또한 블록 중간에 BlockEmbed를 삽입하려고하면 Quill이 블록을 분할합니다. 이 동작을보다 명확하게하기 위해 분할자를 삽입하기 전에 개행을 삽입하여 블록을 명시 적으로 분할합니다. 자세한 내용은 CodePen의 Babel 탭을 살펴보십시오.

<div data-height="400" data-theme-id="23269" data-slug-hash="QEPLrv" data-default-tab="result" data-embed-version="2" class="codepen"></div>


### Images

이미지는 우리가 [Link](#links)와 [Divider](#divider) Blot을 만드는 것을 배운 것과 함께 추가 될 수 있습니다. 값에 대해 객체를 사용하여 이것이 어떻게 지원되는지 보여줍니다. 이미지를 삽입하는 버튼 핸들러는 정적 값을 사용할 것이므로이 가이드의 초점 인 [Parchment](https://github.com/quilljs/parchment/)와는 무관하게 툴팁 UI 코드에 주의를 기울이지 않아도됩니다.

```js
let BlockEmbed = Quill.import('blots/block/embed');

class ImageBlot extends BlockEmbed {
  static create(value) {
    let node = super.create();
    node.setAttribute('alt', value.alt);
    node.setAttribute('src', value.url);
    return node;
  }

  static value(node) {
    return {
      alt: node.getAttribute('alt'),
      url: node.getAttribute('src')
    };
  }
}
ImageBlot.blotName = 'image';
ImageBlot.tagName = 'img';
```

<div data-height="400" data-theme-id="23269" data-slug-hash="Pzggmy" data-default-tab="result" data-embed-version="2" class="codepen"></div>


### Videos

우리는 [images](#images)와 비슷한 방식으로 비디오를 구현할 것입니다. HTML5 `<video>` 태그를 사용할 수는 있지만 YouTube 동영상을 이런 식으로 재생할 수는 없으며 일반적으로 사용되는 경우가 많으므로 이를 지원하기 위해 `<iframe>`을 사용합니다. 우리는 여기에 있을 필요는 없지만, 여러개의 Blot이 같은 태그를 사용하기를 원한다면, 다음 [Tweet](#tweet) 예제에서 보여지는 `tagName`에 `className`을 사용할 수 있습니다.

또한 등록되지 않은 형식으로 너비와 높이에 대한 지원을 추가 할 것입니다. 등록 된 형식과의 네임 스페이스 충돌이없는 한, Embed에 특정한 형식은 별도로 등록 할 필요가 없습니다. Blots는 알 수없는 형식을 자식에게 전달하여 결국 Leaves에 도달하므로이 방법이 효과적입니다. 이것은 또한 다양한 Embed이 등록되지 않은 형식을 다르게 처리 할 수있게합니다. 예를 들어 이전 버전의 [image](#images) 임베드는 Google 비디오가 제공하는 것과 다른 방식으로 'width' 형식을 인식하고 처리 할 수 있었습니다.

```js
class VideoBlot extends BlockEmbed {
  static create(url) {
    let node = super.create();

    // Set non-format related attributes with static values
    node.setAttribute('frameborder', '0');
    node.setAttribute('allowfullscreen', true);

    return node;
  }

  static formats(node) {
    // We still need to report unregistered embed formats
    let format = {};
    if (node.hasAttribute('height')) {
      format.height = node.getAttribute('height');
    }
    if (node.hasAttribute('width')) {
      format.width = node.getAttribute('width');
    }
    return format;
  }

  static value(node) {
    return node.getAttribute('src');
  }

  format(name, value) {
    // Handle unregistered embed formats
    if (name === 'height' || name === 'width') {
      if (value) {
        this.domNode.setAttribute(name, value);
      } else {
        this.domNode.removeAttribute(name, value);
      }
    } else {
      super.format(name, value);
    }
  }
}
VideoBlot.blotName = 'video';
VideoBlot.tagName = 'iframe';
```

콘솔을 열고 [`getContents`](/docs/api/#getcontents)를 호출하면, Quill은 비디오를 다음과 같이보고합니다.

```js
{
  ops: [{
    insert: {
      video: {
        src: 'https://www.youtube.com/embed/QHH3iSeDBLo?showinfo=0'
      }
    },
    attributes: {
      height: '170',
      width: '400'
    }
  }]
}
```

<div data-height="400" data-theme-id="23269" data-slug-hash="qNwWzW" data-default-tab="result" data-embed-version="2" class="codepen"></div>


### Tweets

Medium는 다양한 유형의 삽입을 지원하지만 이 가이드에서는 Tweet에만 초점을 맞춥니다. Tweet Blot은 거의 [images](#images)와 거의 동일하게 구현됩니다. Embed Blots가 무효 노드에 해당 할 필요가 없다는 사실을 이용합니다. 그것은 임의의 노드가 될 수 있고 Quill은 그것을 무효 노드처럼 취급 할 것이고 자식이나 자손을 방해하지 않을 것입니다. 이렇게하면 `<div>`와 네이티브 Twitter Javascript 라이브러리를 사용하여 우리가 지정하는 `<div>` 컨테이너 내에서 만족스러운 것을 할 수 있습니다.

루트 Scroll Blot 또한 `<div>`을 사용하기 때문에 명확성을 위해 `className`도 지정합니다. 참고적으로 인라인 Blot은 `<span>`을 사용하고 블럭 Blot은 기본적으로 `<p>`를 사용합니다. 따라서 사용자 정의 Blot에 이 태그를 사용하려면 `tagName`에 추가적인 `className`을 지정해야합니다.

우리는 Blot을 정의하는 값으로 Tweet ID를 사용합니다. 다시 말해서 클릭 핸들러는 정적인 값을 사용하여 관련성없는 UI 코드에서 주의를 산만하게하지 않습니다.

```js
class TweetBlot extends BlockEmbed {
  static create(id) {
    let node = super.create();
    node.dataset.id = id;
    // 트위터 라이브러리가 컨텐츠를 수정하도록 허용
    twttr.widgets.createTweet(id, node);
    return node;
  }

  static value(domNode) {
    return domNode.dataset.id;
  }
}
TweetBlot.blotName = 'tweet';
TweetBlot.tagName = 'div';
TweetBlot.className = 'tweet';
```

<div data-height="400" data-theme-id="23269" data-slug-hash="vKrBjE" data-default-tab="result" data-embed-version="2" class="codepen"></div>


### Final Polish

우리는 그냥 평범한 텍스트를 이해하는 버튼과 Quill core로 시작했습니다. Parchment를 사용하여 굵게, 기울임 꼴, 링크, 인용구, 헤더, 섹션 구분선, 이미지, 비디오 및 트윗을 추가 할 수 있었습니다. 이 모든 것은 예측 가능하고 일관성있는 문서를 유지하면서 우리가 이러한 새로운 형식과 내용으로 Quill의 강력한 [APIs](/docs/api/)를 사용할 수 있게 합니다.

데모를 끝내기 위해 최종 마무리를 추가해 보겠습니다. Medium의 UI와 비교하지는 않지만, 유사합니다.

<div data-height="400" data-theme-id="23269" data-slug-hash="qNJrYB" data-default-tab="result" data-embed-version="2" class="codepen"></div>


<!-- script -->
<script src="//codepen.io/assets/embed/ei.js" type="text/javascript"></script>
<!-- script -->
