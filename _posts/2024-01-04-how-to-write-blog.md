---
layout: post
title: How To Write Blog
subtitle: MarkDown Grammar
author: dabeann
categories: write
banner:
  # video: https://vjs.zencdn.net/v/oceans.mp4
  # loop: true
  # volume: 0.8
  # start_at: 8.5
  image: "/assets/images/banners/banner.jpeg"
  opacity: 0.618
  background: "#000"
  height: "400px"
  # min_height: "38px"
  heading_style: "font-size: 4.25em; font-weight: bold; text-decoration: underline"
  subheading_style: "color: gold" #이거해서 subtitle이 금색임
tags: write blog
top: 1
sidebar: []
---

<!-- --- 배너 없이 이렇게만 하면 글씨만 제목에 들어감. 사진X
layout: post
title: Test markdown
subtitle: Each post also has a subtitle
categories: markdown
tags: [test, blog]
--- -->

새로운 포스트를 추가하려면, _posts 디렉토리에 YYYY-MM-DD-name-of-post.ext 형식을 따르는 파일을 추가

## Code Chunk

{% highlight ruby %}
def print_hi(name)
puts "Hi, #{name}"
end
print_hi('Tom')
#=> prints 'Hi, Tom' to STDOUT.
{% endhighlight %}

```cpp
#include <iostream>
using namespace std;

int main() {
  cout << "Hello World!";
  return 0;
}
// prints 'Hi, Tom' to STDOUT.
```

```python
class Person:
  def __init__(self, name, age):
    self.name = name
    self.age = age

p1 = Person("John", 36)

print(p1.name)
print(p1.age)
```

~~~
var foo = function(x) {
  return(x + 5);
}
foo(3)
~~~

And here is the same code with syntax highlighting:

```javascript
var foo = function(x) {
  return(x + 5);
}
foo(3)
```

And here is the same code yet again but with line numbers:

{% highlight javascript linenos %}
var foo = function(x) {
  return(x + 5);
}
foo(3)
{% endhighlight %}

## 활용법

활용법은 다음 문서 확인: [Jekyll docs][jekyll-docs]
마크다운 배우는 곳: [markdown](https://markdowntutorial.com/)

[jekyll-docs]: https://jekyllrb.com/docs/home

## Bold
**bold text**

## Inline
\`escape inline code\`  
`inline code`

## quote
> 인용구

## Picture
How about a yummy crepe?
It can also be centered!

![Crepe](https://s3-media3.fl.yelpcdn.com/bphoto/cQ1Yoa75m2yUFFbY2xwuqw/348s.jpg){: .center-block :}

<!-- ### Emoji

This single quote code `inet:email:message:to` will not be parsed to emoji icon
:+1:. :bolivia: -->

<!-- ## Boxes 얘네 박스들 왜 안되는지 ㅁㄹ겠는데 표시가 안됨
You can add notification, warning and error boxes like this:

### Notification

{: .box-note}
**Note:** This is a notification box.

### Warning

{: .box-warning}
**Warning:** This is a warning box.

### Error

{: .box-error}
**Error:** This is an error box. -->

## Table

| Number | Next number | Previous number |
| :------ |:--- | :--- |
| Five | Six | Four |
| Ten | Eleven | Nine |
| Seven | Eight | Six |
| Two | Three | One |

9 \* 9

| 1 \* 1 = 1 |
| 1 \* 2 = 2 | 2 \* 2 = 4 |
| 1 \* 3 = 3 | 2 \* 3 = 6 | 3 \* 3 = 9  |
| 1 \* 3 = 3 | 2 \* 3 = 6 | 3 \* 4 = 12 | 4 \* 4 = 16 |

## Table example as below

### Headerless
Table header can be eliminated.

|--|--|--|--|--|--|--|--|
|♜ |  |♝ |♛ |♚ |♝ |♞ |♜ |
|  |♟ |♟ |♟ |  |♟ |♟ |♟ |
|♟ |  |♞ |  |  |  |  |  |
|  |♗ |  |  |♟ |  |  |  |
|  |  |  |  |♙ |  |  |  |
|  |  |  |  |  |♘ |  |  |
|♙ |♙ |♙ |♙ |  |♙ |♙ |♙ |
|♖ |♘ |♗ |♕ |♔ |  |  |♖ |

## Table Tips
열을 구분하기 위해 파이프 {% raw %}(|){% endraw %}를 사용하고, 헤더 행과 표의 나머지 부분을 구분하기 위해 대시(-)를 사용하자.
마크다운 프로세서에게 공백은 중요하지 않다. 모든 추가 공백은 제거되지만, 가독성을 높이는 데에는 도움이 될 수 있다.
아래의 두 마크다운 예시는 모두 같은 표를 생성한다.

열을 구분하기 위해 파이프 `{% raw %}(`|`){% endraw %}` 를 사용하고, 헤더 행과 표의 나머지 부분을 구분하기 위해 대시(-)를 사용하자.

## URLs

* A named link to [MarkItDown][3]. The easiest way to do these is to select what you want to make a link and hit `Ctrl+L`.
* Another named link to [MarkItDown](https://www.markitdown.net/)
* Sometimes you just want a URL like <https://www.markitdown.net/>.

## Horizontal rule

A horizontal rule is a line that goes across the middle of the page.

---

[3]: https://www.markitdown.net/
